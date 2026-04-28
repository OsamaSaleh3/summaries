
## Executive Summary

This document explores the granular physical storage architecture of row-based relational databases. It details how data is structured within individual disk pages, transitioning from rudimentary storage models to sophisticated implementations like **Slotted Pages** and **Log-Structured Pages**. Furthermore, it breaks down the internal anatomy of a **Tuple** (row), explaining how database engines serialize data into bytes and leverage hardware-level optimizations, such as word alignment, to maximize CPU processing efficiency.

---

## Deep Dive: Page Layouts and Tuple Management

### 1. The Page Header

Every data page begins with a **Page Header** containing crucial metadata for the database management system (DBMS) to interpret its contents. This metadata typically includes:

- Page size and available free space.
    
- Compression and encoding schemes used.
    
- Transaction visibility information.
    

### 2. The Naive Storage Approach

A rudimentary approach stores the total number of tuples in the header and appends uniform-sized tuples sequentially.

- **Locating Data:** Offset is calculated via $Offset = Tuple\_Number \times Tuple\_Size$.
    
- **Critical Flaws:** This approach immediately fails if tuples have variable sizes (e.g., `VARCHAR` columns) or if a tuple is deleted, as shifting data would invalidate existing offsets and break sequential math.
    

### 3. Slotted Pages Architecture

To handle variable-length data and deletions gracefully, most traditional row-based engines (like PostgreSQL and SQL Server) utilize **Slotted Pages**.

- **Architecture:** The page is divided into three sections: the header, an array of **Slots**, and the actual tuple data.
    
- **Growth Direction:** Slots grow forward from the top of the page (after the header), while tuple data grows backward from the bottom of the page. They eventually meet in the middle.
    
- **Mechanism:** Each slot is a fixed-size pointer containing the physical memory offset of its corresponding tuple.
    
- **Deletions:** When a tuple is deleted, its data is removed, and its corresponding slot pointer is nullified. The slot itself remains to preserve the indexing order.
    

> [!INFO] Decoding the Record ID
> 
> A tuple is globally locatable via a unique **Record ID** (or Tuple ID). The DBMS splits this identifier into three routing components:
> 
> 1. **File ID:** Which physical file holds the data.
>     
> 2. **Page ID:** Passed to the Page Directory to find the disk offset.
>     
> 3. **Slot Number:** Used inside the page to follow the exact pointer to the tuple's bytes.
>     

### 4. Log-Structured Pages (LSM Trees)

To mitigate the heavy random I/O and fragmentation issues of slotted pages, systems like RocksDB and Cassandra use **Log-Structured Storage**.

- **Architecture:** The page operates as an append-only log. Every database action (`SET` for inserts/updates, `DELETE` for removals) is appended sequentially with a timestamp and Tuple ID.
    
- **Updates/Deletes:** Existing data is never overwritten in place. An update simply appends a new `SET` operation. A deletion appends a `DELETE` tombstone.
    
- **Reading Data:** To find the current state of a tuple, the engine reads the log backward from the newest entry to the oldest, stopping at the first match. Because reverse scanning is slow, engines maintain an in-memory index mapping Tuple IDs to their most recent log entry.
    

### 5. Internal Tuple Structure and Hardware Alignment

A tuple is fundamentally a raw sequence of bytes. The operating system cannot read it; the DBMS must parse it using the table's defined schema.

- **Word Alignment:** CPUs read data in fixed-size chunks called **Words** (e.g., 64 bits on a 64-bit architecture). If a tuple attribute (like a 64-bit `TIMESTAMP`) crosses the boundary between two CPU words, the CPU must execute multiple read cycles and perform complex bit-shifting to stitch the data back together.
    
- **Padding:** To prevent performance degradation, the DBMS automatically injects empty space (padding) between attributes so that every value aligns perfectly with the CPU's word boundaries.
    

> [!TIP] Oversized Attributes
> 
> If an attribute (like a massive `VARCHAR` or `BLOB`) is larger than a CPU word, padding fails. The DBMS handles this by storing the massive data in a separate overflow page or external file, placing only a fixed-size memory pointer inside the primary tuple.

---

## Optimization Tactics

- **Compaction (Log-Structured Layouts):** Because append-only systems store multiple historical states for a single tuple, space is quickly wasted. Systems run background **Compaction** jobs. These jobs scan old pages, discard redundant historical entries and `DELETE` tombstones, and write only the newest state of each tuple to a fresh, sequentially ordered page.
    
    - _Trade-off:_ While compaction reclaims space and speeds up reads, it consumes heavy background CPU and disk I/O, temporarily bottlenecking ongoing write operations.
        
- **Slotted Page Defragmentation:** Periodically, slotted pages shift all active tuples to the very bottom of the page to consolidate fragmented free space in the middle, making room for larger incoming tuples.
    
- **Schema Attribute Ordering:** Because the DBMS pads tuples to fit word boundaries, the physical order of columns in a `CREATE TABLE` statement can impact disk footprint. Grouping similarly sized data types (e.g., placing all 32-bit `INT` columns together) can minimize wasted padding space.
    

---

## Summary Table: Slotted vs. Log-Structured Pages

|**Feature**|**Slotted Pages**|**Log-Structured Pages**|
|---|---|---|
|**Primary Use Case**|Traditional OLTP (PostgreSQL, SQL Server)|High-throughput write systems (Cassandra, RocksDB)|
|**Write Performance**|Slower (Requires random I/O and finding free space)|Extremely Fast (Sequential append-only writes)|
|**Read Performance**|Fast (Direct access via $O(1)$ slot pointer lookup)|Can be slower (Requires scanning logs or relying on secondary indexes)|
|**In-Place Updates**|Yes (If new data size $\le$ old data size)|No (Always appends a new entry)|
|**Space Efficiency**|Prone to internal fragmentation upon deletion|Wastes space with redundant entries until **Compaction** occurs|