# Database Internals: Relational Model, Algebra, and Optimization

## Executive Summary

This document outlines the foundational mechanics of **Relational Database Management System (RDBMS)** architectures, bridging the gap between theoretical data models and practical query execution. It establishes the principles of the **Relational Model** and details how **Relational Algebra** serves as the procedural engine underlying declarative languages like SQL. Furthermore, it explores how database engines leverage abstract syntax trees to represent queries, allowing the **Query Optimizer** to perform mathematical transformations that drastically reduce computational overhead during query execution.

---

## Deep Dive: The Relational Model & Algebra

### The Relational Model

The **Relational Model** is an abstraction used to conceptualize data organization and interaction, independent of physical storage implementation.

- **Relation (Table):** A representation of a specific entity type, containing multiple attributes.
    
- **Tuple (Row/Record):** A single object instance within a relation.
    
- **Attribute (Column/Field):** Properties of the entity. In the strict theoretical model, values must be atomic/scalar (e.g., strings, integers), forbidding nested arrays or objects.
    
- **Primary Key:** One or more attributes that uniquely identify a tuple within a relation. Denoted theoretically by an underline.
    
- **Set Semantics:** The pure relational model dictates that relations are mathematical sets. Therefore, **duplicate tuples are strictly prohibited**, and the order of tuples is semantically insignificant.
    

> [!INFO] Theory vs. Implementation
> 
> While the theoretical relational model strictly enforces set semantics (no duplicates), modern SQL implementations rely on _bag semantics_ (multisets), allowing duplicate rows unless explicitly restricted (e.g., via `DISTINCT`).

### Relational Algebra

**Relational Algebra** is a procedural language consisting of operations that take one or more relations as input and output a new relation. Because the output is always a relation, operations can be infinitely chained or composed.

#### Unary Operations

1. **Selection ($\sigma$):** Filters a relation, outputting only tuples that satisfy specific boolean conditions (supporting conjunctions and disjunctions).
    
2. **Projection ($\pi$):** Outputs a relation containing only a specified subset of attributes. It can also evaluate arithmetic/string expressions (e.g., `year + 1`).
    
    - _Note:_ Because relational algebra enforces set semantics, projecting a subset of columns may naturally eliminate tuples that become duplicates in the narrower scope.
        
3. **Rename ($\rho$):** Modifies the attribute names of a relation without altering the underlying data, crucial for resolving naming collisions during binary operations.
    

#### Binary and Set Operations

For set operations (Union, Intersection, Difference), the input relations must be **schema-compatible** (having the identical number of attributes with matching names and domains).

1. **Union ($\cup$):** Outputs tuples present in either input relation, eliminating duplicates.
    
2. **Intersection ($\cap$):** Outputs only tuples present in _both_ input relations.
    
3. **Difference ($-$):** Outputs tuples present in the first relation but absent in the second.
    
4. **Cartesian Product ($\times$):** Combines every tuple in the first relation with every tuple in the second relation. If relation A has $N$ rows and relation B has $M$ rows, the product yields $N \times M$ rows.
    
5. **Inner Join ($\bowtie$):** Combines tuples from two relations that have matching values for specified join keys (or conditions).
    
    - _Mechanics:_ A Join is mathematically equivalent to a Cartesian Product followed immediately by a Selection condition ($\sigma_{condition}(R \times S)$).
        

> [!TIP] Resolving Schema Conflicts
> 
> If you need to perform a Union ($\cup$) on two tables with different column names but identical semantic data, chain a Rename ($\rho$) operation first to align the schemas before executing the Union.

---

## Optimization Tactics: Query Trees and Transformations

While SQL is a **declarative** language (the user defines _what_ data they want), the database engine must translate this into a **procedural** execution path. This is handled by the **Query Optimizer**.

### Tree Representation

Relational algebra expressions are mapped into Abstract Syntax Trees (ASTs):

- **Leaves:** Base relations (tables).
    
- **Internal Nodes:** Relational operations (e.g., $\bowtie$, $\sigma$, $\pi$).
    
- **Execution Flow:** The tree is evaluated bottom-up. Child nodes must materialize their results before passing them as inputs to parent nodes.
    

### Transformation and Cost Evaluation

The **Query Optimizer** generates an initial logical tree from the SQL query and applies algebraic transformation rules to generate equivalent trees.

**Optimization Example: Selection Pushdown**

Consider the task: _"Find the name and job title of all personnel who are NOT managers."_

- **Sub-optimal Tree:** Join the `Person` and `Job` tables entirely ($\bowtie$), then apply a filter ($\sigma$) for non-managers, and finally project ($\pi$) the names. This requires holding a massive, unoptimized Cartesian/Join output in memory.
    
- **Optimized Tree:** Apply the selection filter ($\sigma_{title \neq 'manager'}$) directly to the `Job` relation _before_ executing the Join ($\bowtie$) with the `Person` relation.
    

> [!CAUTION] Optimizer Limitations
> 
> Optimizers use heuristics and statistics to choose the lowest-cost **Execution Plan** (minimizing CPU/memory). However, exploring the infinite set of all possible tree transformations is impossible. If a query is poorly structured, the optimizer might miss the most efficient route, necessitating manual query refactoring at the application layer.

---

## Summary Comparisons

### Relational Algebra vs. SQL

| **Feature**        | **Relational Algebra (Theory)**                 | **SQL (Implementation)**                           |
| ------------------ | ----------------------------------------------- | -------------------------------------------------- |
| **Paradigm**       | Procedural (Explicit order of operations)       | Declarative (Engine determines execution order)    |
| **Duplicates**     | Strict Set Semantics (Automatically eliminated) | Bag Semantics (Duplicates allowed by default)      |
| **Tuple Ordering** | Insignificant                                   | Can be enforced via `ORDER BY`                     |
| **Data Types**     | Strictly atomic/scalar                          | Often supports nested/complex types (JSON, Arrays) |

### Join Operation Equivalency

Plaintext

```
// SQL Representation
SELECT * FROM R INNER JOIN S ON R.id = S.id;

// Relational Algebra (Direct Join)
R ⨝_{R.id = S.id} S

// Relational Algebra (Product + Selection Equivalent)
σ_{R.id = S.id} (R × S)
```