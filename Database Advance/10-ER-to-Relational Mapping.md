
## Executive Summary

This document provides a technical guide on translating a conceptual Entity-Relationship (ER) diagram into a physical relational database schema. It covers the explicit rules for mapping entities and handling varying relationship cardinalities (1:1, 1:N, M:N) and participation constraints. The summary outlines structural transformations, primary key selection, foreign key placement, and normalization techniques to ensure referential integrity and query efficiency.

---

## Deep Dive: Mapping Mechanisms

### Step 1: Mapping Entities

Every conceptual entity is mapped directly to a relational table.

- **Columns:** Entity attributes directly translate to table columns.
    
- **Primary Key (PK) Selection:** A PK must be identified to uniquely represent each row. It must be **unique**, **not null**, and **minimal**. It can be a **simple key** (one attribute) or a **composite key** (multiple attributes). Surrogate keys (e.g., `user_id`, `post_id`) are often favored for simplicity and stability over natural keys.
    

### Step 2: Mapping One-to-One (1:1) Relationships

The mapping strategy for 1:1 relationships depends entirely on participation constraints (whether an entity's involvement is optional or mandatory):

- **Total / Total Participation:** Merge both entities and any relationship attributes into a single table.
    
- **Total / Partial Participation:** Place a **Foreign Key (FK)** in the table with total participation (mandatory side), referencing the primary key of the table with partial participation (optional side).
    
    - _Example:_ A `Post` may have an `Image` (partial), but an `Image` must belong to a `Post` (total). The `Image` table receives a `post_id` FK. Because it is a 1:1 relationship, `post_id` can simultaneously serve as the PK for the `Image` table.
        
- **Partial / Partial Participation:** Add an FK to either table, referencing the other. Any relationship attributes are assigned to the table holding the FK.
    

### Step 3: Mapping One-to-Many (1:N) Relationships

For 1:N relationships, the **Foreign Key** is always placed on the "Many" side, pointing to the primary key of the "One" side.

- **Mechanism:** If one `User` can write multiple `Posts`, the `Post` table (the "Many" side) receives a `user_id` FK (e.g., `author_id`) referencing the `User` table.
    
- **Relationship Attributes:** Any attributes belonging to the relationship itself (e.g., a timestamp) are also placed in the table on the "Many" side.
    

### Step 4: Mapping Many-to-Many (M:N) Relationships

M:N relationships cannot be resolved with a simple foreign key column. They strictly require a new **junction table** (or associative entity).

- **Table Creation:** Create a new table dedicated entirely to representing the relationship.
    
- **Foreign Keys:** The junction table contains FKs pointing to the primary keys of both parent tables.
    
- **Primary Key Assignment:** * Typically, the combination of both FKs serves as a **composite primary key** (e.g., a `Likes` table with the PK `(user_id, post_id)`).
    
    - _Exception:_ If the relationship can occur multiple times between the exact same entities (e.g., a user commenting on the same post multiple times), the FK combination is not unique. A new surrogate key (e.g., `comment_id`) must be introduced to form the PK, often alongside the FKs.
        

> [!INFO] Unary Relationships
> 
> A unary relationship (e.g., `User` follows `User`) obeys the standard cardinality rules. For an M:N unary relationship, a junction table is created with two FKs, both pointing back to the same parent table (e.g., a `Follows` table containing `follower_id` and `followed_id`).

---

## Optimization Tactics

- **Denormalization vs. Separation in 1:1:** If an entity (like an `Image`) only holds a single attribute (like a URL) and maintains a 1:1 relationship with a parent (`Post`), storing the URL directly as a nullable column in the `Post` table avoids costly `JOIN` operations at the application layer. However, if the `Image` entity requires multiple attributes (size, format, EXIF metadata), creating a separate table is optimal for schema cleanliness.
    
- **Cascading Deletes for Total Participation:** For entities with total participation (e.g., an `Image` that cannot exist without a `Post`), configure the database's foreign key constraint to `ON DELETE CASCADE`. Deleting the parent record will automatically purge dependent records, maintaining referential integrity without requiring manual, multi-step transaction logic from the application.
    
- **Surrogate Keys for High-Frequency M:N:** In junction tables where events repeat frequently between the same two entities (like comments on a post), using a dedicated surrogate key (`comment_id`) avoids the overhead of managing wide, multi-column composite keys and vastly improves indexing efficiency for read/write workloads.
    

---

## Summary Table: Relationship Mapping Strategies

|**Relationship Type**|**Participation Constraints**|**Mapping Strategy**|**Primary Key Selection**|
|---|---|---|---|
|**1:1**|Total / Total|Merge into one unified table.|Single PK for the combined table.|
|**1:1**|Total / Partial|Add FK to the "Total" side, referencing the "Partial" side.|The FK itself can often serve as the PK.|
|**1:N**|Any|Add FK to the "Many" side, referencing the "One" side.|The existing PK of the "Many" side table.|
|**M:N**|Any|Create a new junction table with FKs to both parent entities.|Composite key of both FKs (or a newly generated surrogate key).|