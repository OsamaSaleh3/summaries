

## Executive Summary

This document explores advanced database indexing structures, specifically focusing on Hash Indexes and Composite (Multi-Key) Indexes. It details the underlying data structures, algorithmic complexities, and operational constraints of each. The core objective is to understand how these indexes drastically reduce disk I/O for specific query patterns and how the query optimizer selects the appropriate index based on the predicate types (point vs. range) and attribute prefixes.

---

## Deep Dive: Advanced Indexing Structures

### Hash Indexes

A Hash Index is an alternative to tree-based indexing, optimized exclusively for strict equality lookups.

- **Architecture:** It consists of a **Hash Function** and a **Hash Table** made up of numbered buckets.
    
- **Mechanism:** When a key (e.g., a first name) is inserted or searched, the hash function computes a numerical value from the key. This value corresponds to a specific bucket ID. The database navigates directly to that bucket to retrieve the key and its associated **Row ID** (a data pointer containing the page ID and slot number).
    
- **Performance:** Because the hash function directly computes the bucket's location without traversing intermediate levels, the search complexity is **$O(1)$** (constant time). This makes it significantly faster than the $O(\log N)$ traversal required by a B-Tree.
    
- **Limitations:** The hashing process does not preserve the relative order of the original keys. Because values are pseudo-randomly distributed across buckets, Hash Indexes cannot process range queries (e.g., `>`, `<`).
    

> [!INFO] PostgreSQL Implementation
> 
> In PostgreSQL, you can explicitly create a Hash Index instead of the default B-Tree. PostgreSQL also utilizes a hidden system column called `ctid` which acts as the Row ID, exposing the physical physical page and slot tuple location.

SQL

```
-- Creating a Hash Index in PostgreSQL
CREATE INDEX name_hash_idx ON table_name USING hash (first_name);

-- Viewing the hidden Row ID (CTID)
SELECT ctid, * FROM table_name;
```

### Composite (Multi-Key) Indexes

A Composite Index is built on two or more attributes (e.g., `(X, Y)`). They can theoretically be implemented as Hash or B-Tree indexes, though many systems (like PostgreSQL) only support composite B-Trees.

- **B-Tree Composite Structure:** Each node entry contains multiple components (e.g., an $X$ value and a $Y$ value).
    
- **Sorting Mechanism:** The tree is sorted lexicographically based on the order the columns were defined. It is sorted primarily by the first attribute ($X$). In the event of a tie on $X$, it sorts secondarily by $Y$.
    
- **Point vs. Range:** * **Point Queries (`X = 2 AND Y = 'A'`):** The engine traverses the tree to find the exact match rapidly.
    
    - **Range Queries (`X = 1 AND Y >= 'B'`):** The engine finds the starting boundary (`1, 'B'`) and sequentially scans the linked leaf nodes until the condition breaks.
        

> [!CAUTION] The Non-Contiguous Range Penalty
> 
> If a composite index query applies a range condition on the primary attribute (e.g., `X > 1 AND Y = 'A'`), the matching entries for `Y` are no longer contiguous. The engine must scan through all leaves where `X > 1` and explicitly evaluate `Y = 'A'`, skipping non-matching tuples. This introduces wasted computational work.

---

## Optimization Tactics

### 1. Index Selection Strategy

When designing a schema, tailor the index to the anticipated query workload:

- Use **Hash Indexes** strictly for heavy read workloads dominated by exact-match lookups (Point Queries).
    
- Use **B-Tree Indexes** if the application requires sorting, inequalities, or range boundaries. If the query optimizer detects an inequality (`<`), it will automatically bypass a Hash Index and utilize a B-Tree if available.
    

### 2. The Prefix Search Rule for Composite Indexes

The order of columns in a composite index is critical. A composite index on `(A, B, C)` can only efficiently answer queries that filter on a contiguous **prefix** of those attributes:

- **Valid index usage:** Filtering on `(A)`, `(A, B)`, or `(A, B, C)`.
    
- **Invalid index usage:** Filtering strictly on `(B)` or `(C)`. Because the data is primarily sorted by `A`, searching for `B` without defining `A` requires scanning the entire index, which is often more expensive than a Full Table Scan.
    

> [!TIP] Optimizing Column Order
> 
> Always order composite index columns starting with the attributes most frequently used in `WHERE` clauses, particularly those used with equality (`=`) predicates.

---

## Summary Table: Index Capability Comparison

|**Index Type / Scenario**|**Point Query (=)**|**Range Query (<, >)**|**Algorithmic Complexity**|**Primary Drawback**|
|---|---|---|---|---|
|**Hash Index**|**Excellent**|**Incapable**|$O(1)$|Useless for ranges or sorting; not universally supported for composite keys.|
|**B-Tree Index**|Good|**Excellent**|$O(\log N)$|Slightly slower point lookups than Hash due to tree depth traversal.|
|**Composite B-Tree `(X, Y)`**|Excellent (if prefix provided)|Good (if prefix provided)|$O(\log N)$|Useless if querying only on secondary attributes (e.g., only `Y`); non-contiguous range scanning overhead.|