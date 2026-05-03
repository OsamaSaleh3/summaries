

## title: Hibernate & ORM Fundamentals tags: [java, hibernate, orm, jdbc, postgresql, backend] type: study-note

# Hibernate & ORM Fundamentals

> [!info] Overview
> 
> This note covers the transition from raw JDBC to Hibernate. It explains the core concepts of Object-Relational Mapping (ORM), how to set up a Hibernate project using Maven, configure the `hibernate.cfg.xml` file, and persist Java objects into a PostgreSQL database using JPA annotations.


## Lesson 1 & 2: Concepts - The "Why" Before the "How"

### Why do we need an ORM like Hibernate?

When building Java applications, you are working in an **Object-Oriented** paradigm. Everything is an object. However, relational databases (like PostgreSQL or MySQL) store data in **tables and relations**.

When using plain JDBC, bridging this gap is a tedious, manual process:

1. You have to write raw SQL queries (`INSERT INTO...`).
    
2. You have to manually fetch variables from your Java objects and map them into the SQL query parameters.
    
3. You have to handle connections and map the returned tabular data back into Java objects.
    

**Object-Relational Mapping (ORM)** solves this. An ORM framework takes your Java Object and automatically maps it to a database table.

- **Class** $\rightarrow$ **Table** (e.g., `Student` class becomes a `student` table)
    
- **Variables** $\rightarrow$ **Columns** (e.g., `sName` becomes an `sname` column)
    
- **Object Instance** $\rightarrow$ **Row** (e.g., `new Student(1, "Naveen")` becomes a row in the table)
    

> [!tip] Benefits of Hibernate
> 
> - **Productivity**: No need to write repetitive CRUD SQL queries.
>     
> - **Maintainability**: Easy to swap out underlying databases (e.g., moving from MySQL to PostgreSQL) without rewriting queries.
>     
> - **Performance**: Out-of-the-box caching and optimization.
>     

---

## Lesson 3: Project Setup & Dependencies

To get started with Hibernate, you need a build tool. We use **Maven** to pull in the required libraries.

### Why do we need these dependencies?

1. **PostgreSQL Driver**: Hibernate doesn't speak to the database directly; it still uses JDBC under the hood. You must provide the specific database driver.
    
2. **Hibernate ORM Core**: The actual framework that will intercept your Java objects and generate the SQL.
    



```XML
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.7.2</version> </dependency>

<dependency>
    <groupId>org.hibernate.orm</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>6.6.3.Final</version> </dependency>
```

---

## Lesson 4: The Hibernate Architecture (Session & SessionFactory)

### Why do we need a Session and a SessionFactory?

Connecting to a database and persisting data is a resource-heavy operation. Hibernate manages this using two main interfaces:

- **`SessionFactory`**: A heavy-weight, thread-safe object. You should only create **one** `SessionFactory` per database during your application's lifecycle. It reads the configuration and establishes the foundation.
    
- **`Session`**: A light-weight, non-thread-safe object created by the `SessionFactory`. You open a new `Session` for every unit of work (like saving a user), and then close it.
    

### The Implementation Steps:

1. Create your POJO (Plain Old Java Object), e.g., `Student`.
    
2. Build the `Configuration`.
    
3. Build the `SessionFactory`.
    
4. Open a `Session`.
    
5. Save the object.
    



```Java
// 1. Create the Object
Student s1 = new Student();
s1.setRollNumber(101);
s1.setName("Naveen");
s1.setAge(30);

// 2. Setup Configuration and SessionFactory
Configuration cfg = new Configuration();
cfg.addAnnotatedClass(Student.class); // Tell Hibernate about our Entity
cfg.configure(); // Loads hibernate.cfg.xml

SessionFactory sf = cfg.buildSessionFactory();

// 3. Open Session and Save
Session session = sf.openSession();
session.save(s1); // Deprecated in newer versions, use persist(s1)
```

> [!warning] Configuration Error
> 
> If you run the code above as-is, Hibernate will throw an error because it doesn't know _where_ your database is, what the credentials are, or that the `Student` class is meant to be a database table.

---

## Lesson 5: Configuration & Transactions

To fix the missing configuration, we must create a `hibernate.cfg.xml` file in the `src/main/resources` folder and use JPA annotations on our class.

### Why do we need Transactions?

Relational databases follow ACID properties. Saving data modifies the database state, which means it must be treated as a transaction. Hibernate requires you to explicitly begin and commit a transaction when modifying data. If you don't commit, the data won't be saved to the database.

### 1. `hibernate.cfg.xml` Setup



```XML
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">

<hibernate-configuration>
    <session-factory>
        <property name="hibernate.connection.driver_class">org.postgresql.Driver</property>
        <property name="hibernate.connection.url">jdbc:postgresql://localhost:5432/telusko</property>
        <property name="hibernate.connection.username">postgres</property>
        <property name="hibernate.connection.password">0000</property>
        
        <property name="hibernate.hbm2ddl.auto">update</property>
    </session-factory>
</hibernate-configuration>
```

_Note: `hbm2ddl.auto` set to `update` allows Hibernate to automatically create the table if it doesn't exist, or update the schema if you add new columns. Use this for development, but avoid in production!_

### 2. Updating the Java Code for Transactions



```Java
Session session = sf.openSession();

// Begin Transaction
Transaction tx = session.beginTransaction();

session.save(s1); 

// Commit Transaction to push changes to the DB
tx.commit(); 
```

---

## Lesson 6: Advanced Configuration (Logging & Dialects)

When debugging, it feels like magic when Hibernate saves an object. But under the hood, it's generating SQL. You can expose this SQL for debugging purposes.

### Why do we need a Dialect?

While SQL is a standard, every database (PostgreSQL, MySQL, Oracle) has its own slight variations (dialects). Telling Hibernate exactly which dialect to use ensures it generates highly optimized and perfectly accurate SQL for your specific database engine.

Add these properties to your `hibernate.cfg.xml`:



```XML
<property name="hibernate.show_sql">true</property>

<property name="hibernate.format_sql">true</property>

<property name="hibernate.dialect">org.hibernate.dialect.PostgreSQLDialect</property>
```

---

## Spring & JPA Annotations

Hibernate implements the Jakarta Persistence API (JPA) standard. This means we use standard `@jakarta.persistence.*` annotations so our code isn't tightly coupled to Hibernate-specific libraries.

- **`@Entity`**:
    
    - **What it does:** Marks a standard Java class as a persistent entity.
        
    - **Why we use it:** It tells Hibernate, "This class should be mapped to a table in the database."
        
- **`@Id`**:
    
    - **What it does:** Marks a specific field as the Primary Key of the entity.
        
    - **Why we use it:** Relational databases require a primary key to uniquely identify records. Hibernate will refuse to map an `@Entity` unless at least one field is annotated with `@Id`.
        



```Java
import jakarta.persistence.Entity;
import jakarta.persistence.Id;

@Entity
public class Student {
    
    @Id
    private int rollNumber;
    private String sName;
    private int sAge;
    
    // Getters, Setters, and toString() omitted for brevity
}
```

---

## Summary (TL;DR)

- **ORM (Object-Relational Mapping)**: Bridges the gap between Object-Oriented Java and Relational Databases, eliminating the need to write manual JDBC SQL queries.
    
- **Hibernate**: A powerful ORM framework that implements the JPA standard.
    
- **Setup**: Requires the DB Driver and Hibernate Core dependencies in Maven.
    
- **Architecture**: You create a `Configuration`, build a heavyweight `SessionFactory` (once), and open lightweight `Session` objects for individual tasks.
    
- **Annotations**: Use `@Entity` to map a class to a table and `@Id` to define the primary key.
    
- **Transactions**: You must explicitly `beginTransaction()` and `commit()` when saving data to maintain database integrity.
    
- **Configuration**: The `hibernate.cfg.xml` file holds your DB credentials, connection URLs, and helpful development properties like `hbm2ddl.auto` (auto-table creation) and `show_sql` (debugging).
  
  
  ___


## title: Hibernate CRUD Operations & Advanced Mapping tags: [java, hibernate, jpa, crud, annotations, database] type: study-note

# Hibernate CRUD Operations & Advanced Mapping

> [!info] Overview
> 
> This note transitions from basic setup to actual database manipulation using Hibernate. It covers standardizing operations to JPA specifications (replacing deprecated methods), performing CRUD (Create, Read, Update, Delete) operations, handling transactions, and using advanced JPA annotations to customize database schemas.

---

## 1. Concepts: The "Why" Before the "How"

### Why are methods like `save()`, `update()`, and `delete()` deprecated?

If you use Hibernate 6.0+, you will notice that classic methods like `session.save()` are crossed out (deprecated). This is because Hibernate is actively aligning its native API with the **Jakarta Persistence API (JPA)** standard. To maintain compatibility and future-proof your code, you should use the JPA-equivalent methods:

- `save()` $\rightarrow$ `persist()`
    
- `update()` or `saveOrUpdate()` $\rightarrow$ `merge()`
    
- `delete()` $\rightarrow$ `remove()`
    

### Why do we need to close the `Session` and `SessionFactory`?

A `SessionFactory` is a heavy-weight object that consumes significant system resources. A `Session` holds an active connection to the database. If you do not explicitly close them (or use a `try-with-resources` block), you risk memory leaks and exhausting your database connection pool.

> [!tip] Chaining Configuration
> 
> Instead of writing multiple lines to configure Hibernate, you can chain the methods into a single, cleaner statement:
> 
> Java
> 
> ```
> SessionFactory sf = new Configuration()
>     .addAnnotatedClass(Student.class)
>     .configure()
>     .buildSessionFactory();
> ```

---

## 2. CRUD Operations in Hibernate

### Read (GET)

To fetch data, you use the `get()` method.

**Why no transaction?** You only need to explicitly begin and commit a transaction when you are _modifying_ database state (Insert, Update, Delete). Since reading data does not alter the database, a transaction is unnecessary.



```Java
// Open session (assuming SessionFactory 'sf' is already created)
Session session = sf.openSession();

// Fetch the student with Primary Key 102
Student s2 = session.get(Student.class, 102);

// Always check for null before using the object to avoid NullPointerExceptions
if (s2 != null) {
    System.out.println(s2);
} else {
    System.out.println("Student not found!");
}

session.close();
```

### Update (MERGE)

To update an existing record, you use `merge()`.

**Why use `merge()`?** The `merge()` method is smart. It acts like an "upsert" (Update or Insert). First, it fires a `SELECT` query to check if the record exists. If it does, it updates it. If the record does not exist, it inserts it as a new row.



```Java
Session session = sf.openSession();
Transaction tx = session.beginTransaction(); // Transaction REQUIRED for updates

// 1. Define the object with the updated data
Student s1 = new Student();
s1.setRollNumber(103); // Existing ID
s1.setName("Harsh");
s1.setAge(23); // Updated Age

// 2. Merge the changes
session.merge(s1);

tx.commit();
session.close();
```

### Delete (REMOVE)

To delete a record, use the `remove()` method. The safest way to delete a record is to fetch it first to ensure it exists, and then pass that object to the `remove()` method.



```Java
Session session = sf.openSession();
Transaction tx = session.beginTransaction(); // Transaction REQUIRED for deletions

// 1. Fetch the object first
Student s1 = session.get(Student.class, 109);

// 2. Remove it if it exists
if(s1 != null) {
    session.remove(s1);
}

tx.commit();
session.close();
```

---

## 3. Spring / JPA Annotations

By default, Hibernate maps your Java class name to the database table name, and your variable names to the column names. However, Java conventions (camelCase) often clash with Database conventions (snake_case). We use annotations to override these default mappings.

- **`@Table(name="...")`**
    
    - **What it does:** Changes the name of the database table that this entity maps to.
        
    - **Why we use it:** To decouple the database schema from our Java class naming conventions. (e.g., mapping class `Alien` to table `alien_table`). _Note: You can also use `@Entity(name="...")`, but that changes the entity name in HQL/JPQL queries as well as the table name. `@Table` only alters the database table._
        
- **`@Column(name="...")`**
    
    - **What it does:** Overrides the default column name for a specific variable.
        
    - **Why we use it:** To map a Java variable like `aName` to a database column like `alien_name`.
        
- **`@Transient`**
    
    - **What it does:** Instructs Hibernate to completely ignore the field.
        
    - **Why we use it:** Sometimes you need a variable in your Java object for temporary business logic or calculations, but you _do not_ want a corresponding column created or saved in the database schema.
        

### Annotation Example:



```Java
import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import jakarta.persistence.Table;
import jakarta.persistence.Column;
import jakarta.persistence.Transient;

@Entity
@Table(name = "alien_table") // Table name in DB will be 'alien_table'
public class Alien {

    @Id
    private int aid;

    @Column(name = "alien_name") // Column name in DB will be 'alien_name'
    private String aName;

    @Transient // This field will NOT be stored in the database
    private String technology; 

    // Getters, Setters, and toString() omitted
}
```

> [!warning] `hibernate.hbm2ddl.auto` Behaviors
> 
> If you set `<property name="hibernate.hbm2ddl.auto">create</property>` in your config, Hibernate will **drop** the table if it exists and create a brand new, empty table every time you run the application. Switch it back to `update` to preserve existing data.

---

## Summary (TL;DR)

- **JPA Alignment:** Stop using `save()`, `update()`, and `delete()`. Use the JPA-standard methods: `persist()` (Create), `merge()` (Update/Insert), and `remove()` (Delete).
    
- **Reading Data:** Use `session.get(Entity.class, PrimaryKey)`. Transactions are **not** required for reading. Always handle potential `null` returns.
    
- **Modifying Data:** `merge()`, `persist()`, and `remove()` **require** an active transaction (`beginTransaction()` and `commit()`) to alter the database.
    
- **Schema Customization:** * `@Table(name="name")` maps the class to a specific table.
    
    - `@Column(name="name")` maps a field to a specific column.
        
    - `@Transient` tells Hibernate to ignore the variable completely (no DB mapping).
        
- **Resource Management:** Always `.close()` your `Session` and `SessionFactory` objects to prevent memory leaks.
  
  ---

## title: Hibernate Embeddables & Entity Relationships (One-to-One) tags: [java, hibernate, jpa, database, orm, mapping] type: study-note

# Hibernate Embeddables & Entity Relationships

> [!info] Overview
> 
> This note transitions from storing primitive data types to mapping complex Java objects in Hibernate. It covers how to use `@Embeddable` to flatten complex objects into a single table, and introduces Database Relationship mappings (specifically `@OneToOne`) to connect independent entities via Foreign Keys.

---

## 1. Concepts: The "Why" Before the "How"

### Why do we need `@Embeddable` objects?

In Java, it is best practice to group related fields into cohesive objects. For example, instead of an `Alien` class having `laptopBrand`, `laptopModel`, and `laptopRam` as separate string/int variables, you create a dedicated `Laptop` class to hold them.

However, by default, Hibernate only knows how to map basic JDBC types (like `int` to `INTEGER` or `String` to `VARCHAR`). If you give Hibernate a complex `Laptop` object, it throws a "Cannot determine JDBC type" error. If you don't want to create a separate database table for laptops, you make the class **Embeddable**. This tells Hibernate to take the fields inside the `Laptop` object and "embed" or flatten them as distinct columns inside the `alien` table.

### Why switch from Embeddables to Relationships (Entities)?

While embedding works for simple compositions, it breaks down in real-world scenarios:

1. **Identity:** A laptop is a physical, independent object. It should have its own identity (Primary Key) in the database.
    
2. **Reassignment:** If a company takes a laptop from Alien A and gives it to Alien B, an embedded object makes this difficult to track.
    
3. **Multiplicity:** What if an Alien has _zero_ laptops? What if they have _three_? You cannot dynamically add columns to a single table to accommodate multiple laptops.
    

Therefore, `Laptop` must become its own `@Entity` with its own table, and we must link the `Alien` table to the `Laptop` table using a **Foreign Key** mapping (e.g., One-to-One).

---

## 2. Implementation: From Embeddable to @OneToOne

### Phase 1: Using `@Embeddable` (Single Table)

In this scenario, all laptop fields become columns in the `Alien` table. No separate table is created.



```Java
import jakarta.persistence.Embeddable;

@Embeddable // Tells Hibernate: Do not create a new table, embed these fields in the parent table.
public class Laptop {
    private String brand;
    private String model;
    private int ram;
    
    // Getters, Setters, toString() omitted
}
```



```Java
import jakarta.persistence.Entity;
import jakarta.persistence.Id;

@Entity
public class Alien {
    @Id
    private int aid;
    private String aName;
    
    // The Laptop fields will be stored as columns in the Alien table
    private Laptop laptop; 
    
    // Getters, Setters, toString() omitted
}
```

### Phase 2: Using `@OneToOne` Mapping (Two Tables)

Now, we elevate `Laptop` to a full Entity. It will have its own table, and `Alien` will store a Foreign Key pointing to the `Laptop`.

**Step 1: Upgrade Laptop to an Entity**



```Java
import jakarta.persistence.Entity;
import jakarta.persistence.Id;

@Entity // Now a standalone table
public class Laptop {
    @Id // Requires a primary key
    private int lid;
    private String brand;
    private String model;
    
    // Getters, Setters, toString() omitted
}
```

**Step 2: Map the Relationship in the Parent**



```Java
import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import jakarta.persistence.OneToOne;

@Entity
public class Alien {
    @Id
    private int aid;
    private String aName;
    
    @OneToOne // Tells Hibernate to create a Foreign Key linking to the Laptop table
    private Laptop laptop;
    
    // Getters, Setters, toString() omitted
}
```

> [!warning] Critical Configuration Steps
> 
> 1. **Add Both Classes:** You must tell Hibernate about the new entity in your configuration setup: `cfg.addAnnotatedClass(Laptop.class);`
>     
> 2. **Persist Order Matters:** Because `Alien` depends on `Laptop` (via the foreign key), you must persist the `Laptop` object into the database _before_ you persist the `Alien` object.
>     



```Java
// Execution Example
Laptop l1 = new Laptop();
l1.setLid(1);
l1.setBrand("Asus");

Alien a1 = new Alien();
a1.setAid(101);
a1.setAName("Navin");
a1.setLaptop(l1); // Link the objects

// Persist the independent entity FIRST
session.persist(l1); 
// Then persist the dependent entity
session.persist(a1); 
```

---

## 3. Spring / JPA Annotations Mentioned

- **`@Embeddable`**
    
    - **What it does:** Marks a class to be embedded into the table of the entity that contains it.
        
    - **Brief Explanation:** Prevents the creation of a separate table while allowing you to use complex Object-Oriented structures in your Java code.
        
- **`@Entity`**
    
    - **What it does:** Marks a class as a standalone database table requiring its own `@Id`.
        
- **`@OneToOne`**
    
    - **What it does:** Defines a 1:1 relational mapping between two entities.
        
    - **Brief Explanation:** Hibernate will automatically generate a Foreign Key column in the table of the entity holding the annotation, pointing to the Primary Key of the target entity.
        
- **`@OneToMany` / `@ManyToOne` / `@ManyToMany`**
    
    - **Brief Explanation:** Mentioned conceptually. Used when an entity can hold a collection (List/Set) of other entities, or when multiple entities map to a single entity. `@ManyToMany` strictly requires the creation of a third "mapping" table in the database.
        

---

## Summary (TL;DR)

- **Complex Types:** By default, Hibernate doesn't know how to map custom Java objects to a database.
    
- **`@Embeddable`:** Use this when you want to group variables into a Java object (like `Laptop`), but want those variables stored as standard columns inside the parent's table (like `Alien`). No primary key is needed for the embedded class.
    
- **Entity Relationships:** When an object represents a standalone real-world concept that can be shared or reassigned, it must be an `@Entity` with its own `@Id`.
    
- **`@OneToOne`:** Maps a single entity to exactly one other entity. It automatically creates a Foreign Key in the database to link the two tables.
    
- **Rule of Thumb:** When using relationships, you must explicitly add _all_ entity classes to your Hibernate configuration and persist the parent (referenced) object before the child (referencing) object to avoid constraint errors.

---

## title: Hibernate Entity Relationships (One-to-Many, Many-to-Many) & Fetch Types tags: [java, hibernate, jpa, relationships, mapping, caching] type: study-note

# Hibernate Entity Relationships & Fetch Types

> [!info] Overview
> 
> This note explores complex database relationships in Hibernate, specifically transitioning from One-to-One to **One-to-Many** and **Many-to-Many** mappings. It also introduces the `mappedBy` attribute to prevent redundant tables and explains the critical performance concepts of **Lazy vs. Eager Fetching** and **Level 1 Caching**.

---

## 1. Concepts: The "Why" Before the "How"

### Why does Hibernate create unexpected third tables?

When you define a `@OneToMany` relationship (e.g., One Alien has Many Laptops), Hibernate defaults to creating a **Join Table** (a third table like `alien_laptop`) to map the relationship.

Furthermore, in a `@ManyToMany` relationship, if both entities manage the mapping, Hibernate will create _two_ redundant mapping tables (e.g., `alien_laptop` and `laptop_alien`). This happens because both entities assume it is their responsibility to manage the database schema.

### Why do we need the `mappedBy` attribute?

To avoid these redundant third tables, we must explicitly tell Hibernate which entity "owns" the relationship. Using `mappedBy` on one side of a bidirectional relationship delegates the foreign key management to the other entity.

- _Example:_ "Hey Alien, don't create a mapping table. The relationship is already mapped by the `alien` variable in the `Laptop` class."
    

### Why does Hibernate use Lazy Fetching by default?

When you fetch an entity (like an `Alien`) from the database, it might have a large collection of related entities (like a list of 100 `Laptops`). If Hibernate fetched all 100 laptops every time you simply needed the Alien's name, it would severely degrade database performance and waste memory.

**Lazy Fetching** solves this by only fetching the primary entity's basic fields. It delays fetching the collection from the database until you explicitly access it in your Java code.

### Why doesn't `session.get()` fire a SQL Query sometimes?

Hibernate has a **Level 1 Cache** tied to the current `Session`. If you persist an object and then immediately call `session.get()` within the _same_ session, Hibernate fetches the object directly from memory (the cache) instead of hitting the database. To see the actual SQL query, you must fetch the data using a fresh `Session`.

---

## 2. Implementation: Managing Relationships

### One-to-Many & Many-to-One Mapping

To map an Alien with multiple Laptops without generating a third table, we place `@OneToMany(mappedBy = "...")` on the parent and `@ManyToOne` on the child.



```Java
// Parent Entity (Alien)
@Entity
public class Alien {
    @Id
    private int aid;
    private String aName;

    // Tells Hibernate: "Do not create an alien_laptop table. 
    // The 'alien' variable in the Laptop class handles the Foreign Key."
    @OneToMany(mappedBy = "alien")
    private List<Laptop> laptops = new ArrayList<>();
    
    // Getters and Setters
}
```



```Java
// Child Entity (Laptop)
@Entity
public class Laptop {
    @Id
    private int lid;
    private String brand;

    @ManyToOne // Generates an 'alien_aid' Foreign Key column in the Laptop table
    private Alien alien;
    
    // Getters and Setters
}
```

### Many-to-Many Mapping

For Many-to-Many, a third mapping table is **mandatory**, but we must use `mappedBy` on one side to prevent Hibernate from creating duplicate mapping tables.



```Java
// In Alien.java
@ManyToMany(mappedBy = "aliens") // Delegates mapping to Laptop
private List<Laptop> laptops = new ArrayList<>();

// In Laptop.java
@ManyToMany // This side owns the relationship and creates the single 'laptop_alien' table
private List<Alien> aliens = new ArrayList<>();
```

---

## 3. Implementation: Lazy vs. Eager Fetching

By default, collections (`@OneToMany`, `@ManyToMany`) use `FetchType.LAZY`. If you strictly require all data to be loaded via a SQL `JOIN` immediately, you can switch it to `FetchType.EAGER`.

> [!warning] Best Practice
> 
> Always prefer `FetchType.LAZY` (the default) in production environments. `FetchType.EAGER` can lead to massive performance bottlenecks when dealing with deep or large object graphs.



```Java
@Entity
public class Alien {
    @Id
    private int aid;
    private String aName;

    // Forces Hibernate to fetch the laptops immediately using a JOIN query
    @OneToMany(mappedBy = "alien", fetch = FetchType.EAGER)
    private List<Laptop> laptops = new ArrayList<>();
}
```

---

## 4. Spring / JPA Annotations Context

- **`@OneToMany`**
    
    - **What it does:** Indicates that one instance of the current entity can be associated with multiple instances of another entity.
        
- **`@ManyToOne`**
    
    - **What it does:** Indicates that multiple instances of the current entity can be associated with exactly one instance of another entity.
        
- **`@ManyToMany`**
    
    - **What it does:** Defines a many-to-many relationship, strictly requiring a third mapping table in the database to resolve.
        
- **`mappedBy = "..."`**
    
    - **What it does:** An attribute used inside relationship annotations. It specifies the name of the variable in the target entity that owns the mapping, suppressing redundant table generation.
        

---

## Summary (TL;DR)

- **Join Tables:** By default, Hibernate creates intermediate tables for collections. Use `mappedBy` on the inverse/parent side of the relationship to force Hibernate to use standard Foreign Keys instead.
    
- **`@OneToMany` & `@ManyToOne`:** Used together. The "Many" side usually owns the relationship (holds the foreign key).
    
- **`@ManyToMany`:** Requires a third table, but using `mappedBy` on one side prevents the creation of a _second_ redundant mapping table.
    
- **Level 1 Cache:** Hibernate caches objects within a single `Session`. Calling `persist()` then `get()` in the same session won't trigger a `SELECT` database query.
    
- **Lazy Fetching:** The default behavior for collections. Hibernate only fetches the related objects from the DB when the list is actually accessed in code.
    
- **Eager Fetching:** Configured via `fetch = FetchType.EAGER`. It fetches all associated data immediately via a `JOIN`, which is usually discouraged for performance reasons.


---

## title: Hibernate Query Language (HQL) & Caching tags: [java, hibernate, jpa, hql, database, caching] type: study-note

# Hibernate Query Language (HQL) & Caching

> [!info] Overview
> 
> This note covers advanced data retrieval using Hibernate Query Language (HQL), managing query parameters, optimizing application performance with Lazy fetching (proxies), and implementing Level 2 Caching across multiple sessions.

---

## 1. Concepts: The "Why" Before the "How"

### Why do we need HQL (Hibernate Query Language)?

While methods like `session.get()` are great for fetching an entity by its Primary Key, real-world applications require complex filtering (e.g., finding all laptops with 32GB RAM).

Instead of writing native SQL—which tightly couples your Java application to the database table structures—you use HQL.

- **SQL** uses Database concepts: `SELECT * FROM laptop_table WHERE ram_col = 32`
    
- **HQL** uses Java Object concepts: `FROM Laptop WHERE ram = 32` (It uses the Class Name and Variable Names).
    

### Why use parameters (`?1`) instead of string concatenation?

When passing user input into a query, concatenating strings (e.g., `"... WHERE brand = '" + brand + "'"` ) is dangerous and inefficient. It exposes your application to SQL Injection attacks and prevents the database engine from caching the query execution plan. Parameterized queries fix both issues.

### Why does `load()` or `getReference()` exist alongside `get()`?

`session.get()` is an **Eager** operation; the moment the line executes, a SQL `SELECT` query is fired. However, sometimes you just need an object reference to fulfill a mapping (like assigning a laptop to an alien) without actually needing to print or process the laptop's internal data.

To save database hits, Hibernate provides **Lazy** fetching via proxies. It gives you a "dummy" object and delays firing the SQL query until you actually call a getter method on that object.

### Why do we need Level 2 Caching?

By default, Hibernate provides a **Level 1 Cache** that is strictly tied to a specific `Session`. If you query the same primary key twice within the _same_ session, Hibernate only hits the database once.

However, if two _different_ sessions query the same primary key, Hibernate fires two SQL queries. To share cached data globally across the entire `SessionFactory` (all sessions), you must configure a **Level 2 Cache**.

---

## 2. Implementation: Hibernate Query Language (HQL)

### Basic HQL Query

To fetch all records or filter by specific properties, use `session.createQuery()` with HQL syntax. Note how we don't need `SELECT *`—we just write `FROM <ClassName>`.



```Java
// Fetching all laptops where the RAM property is 32
Query<Laptop> query = session.createQuery("FROM Laptop WHERE ram = 32", Laptop.class);
List<Laptop> laptops = query.getResultList();

for(Laptop lap : laptops) {
    System.out.println(lap);
}
```

### Parameterized Queries

Use positional parameters starting with `?` and an index number.



```Java
String brandName = "Asus";
int ramSize = 16;

// Using ?1 and ?2 as placeholders
Query<Laptop> query = session.createQuery("FROM Laptop WHERE brand = ?1 AND ram = ?2", Laptop.class);
query.setParameter(1, brandName);
query.setParameter(2, ramSize);

List<Laptop> laptops = query.getResultList();
```

### Fetching Partial Data (Specific Columns)

If you only select a single column, HQL returns a List of that specific data type. If you select multiple columns, it returns a `List<Object[]>`.



```Java
// Fetching only two specific columns
Query<Object[]> query = session.createQuery("SELECT brand, model FROM Laptop", Object[].class);
List<Object[]> laptops = query.getResultList();

for(Object[] data : laptops) {
    // Typecast from Object to the underlying type
    String brand = (String) data[0];
    String model = (String) data[1];
    System.out.println(brand + " : " + model);
}
```

---

## 3. Implementation: Lazy vs. Eager Fetching (`get` vs `getReference`)

The deprecated `session.load()` method has been replaced in modern JPA/Hibernate. Use `byId().getReference()` to achieve the lazy-loading proxy behavior.



```Java
Session session = sf.openSession();

// EAGER FETCHING: Database is queried immediately on this line.
Laptop eagerLap = session.get(Laptop.class, 2); 

// LAZY FETCHING (Proxy): Database is NOT queried yet.
Laptop lazyLap = session.byId(Laptop.class).getReference(2);

// The SQL SELECT query is only fired when we actually attempt to use the data
System.out.println(lazyLap.getBrand()); 
```

---

## 4. Implementation: Level 2 Cache

To enable Level 2 cache so that data is shared across multiple sessions, you need to use an external provider like **Ehcache**.

**Step 1:** Add Maven Dependencies



```XML
<dependency>
    <groupId>org.hibernate.orm</groupId>
    <artifactId>hibernate-jcache</artifactId>
    <version>6.6.3.Final</version> 
</dependency>
<dependency>
    <groupId>org.ehcache</groupId>
    <artifactId>ehcache</artifactId>
    <version>3.10.8</version> 
</dependency>
```

**Step 2:** Update `hibernate.cfg.xml`



```XML
<property name="hibernate.cache.use_second_level_cache">true</property>

<property name="hibernate.cache.region.factory_class">org.hibernate.cache.jcache.JCacheRegionFactory</property>
```

**Step 3:** Use the Annotation (See section below).

---

## 5. Spring / JPA Annotations Context

- **`@Cacheable`**
    
    - **What it does:** Instructs Hibernate to store instances of this entity in the Level 2 Cache.
        
    - **Why we use it:** Without this annotation, Hibernate will still bypass the L2 cache for the entity, even if global caching properties are enabled in the `hibernate.cfg.xml`. You must selectively opt-in the entities you want to cache.
        



```Java
import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import jakarta.persistence.Cacheable;

@Entity
@Cacheable // Enables Level 2 Caching specifically for Laptop entities
public class Laptop {
    @Id
    private int lid;
    private String brand;
    private int ram;
    // ...
}
```

---

## Summary (TL;DR)

- **HQL vs SQL:** HQL uses Java class names and variables instead of DB table names and columns (e.g., `FROM Laptop`).
    
- **Parameters:** Use numbered placeholders (`?1`, `?2`) combined with `query.setParameter()` to safely dynamically pass variables into HQL queries.
    
- **`get()` vs `getReference()`:** `get()` eagerly fetches data and executes SQL immediately. `getReference()` (the modern alternative to `load()`) returns a proxy and lazily delays the SQL execution until a field is explicitly accessed.
    
- **Level 1 Cache:** Enabled by default. Caches database queries within the _same_ `Session`.
    
- **Level 2 Cache:** Requires external providers (like Ehcache). Needs config properties and the `@Cacheable` annotation on the entity to cache data globally across _multiple_ `Sessions`.