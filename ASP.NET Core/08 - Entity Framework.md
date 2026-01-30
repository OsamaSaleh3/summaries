

---

### **Video 2 — Add DbContext and ConnectionString**

✅ Add `DbContext` class  
✅ Configure connection string in `appsettings.json`

```csharp
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

    public DbSet<Product> Products { get; set; }
}
```

**appsettings.json**

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=.;Database=MyDb;Trusted_Connection=True;"
  }
}
```

**Program.cs / Startup.cs**

```csharp
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
```

---

### **Video 3 — Add First Migration**

✅ Install EF tools  
✅ Create first migration

**Command**

```
Add-Migration InitialCreate
```

✅ Update database

```
Update-Database
```

---

### **Video 4 — Save new record (.Add())**

✅ Add a new entity and save to DB

```csharp
var product = new Product { Name = "Laptop", Price = 1000 };
_context.Products.Add(product);
_context.SaveChanges();
```

---

### **Video 5 — Migration Rollback**

✅ Roll back last migration

**Command**

```
Remove-Migration
```

✅ OR revert DB to a previous migration

```
Update-Database MigrationName
```

---

### **Video 6 — Add Your Own Migration**

✅ Create a custom-named migration

**Command**

```
Add-Migration AddCategoryTable
```

✅ Apply to database

```
Update-Database
```

---

### **Video 7 — DataAnnotations vs Fluent API**

✅ Use **DataAnnotations** inside models

```csharp
public class Product
{
    public int Id { get; set; }

    [Required]
    [MaxLength(100)]
    public string Name { get; set; }
}
```

✅ Use **Fluent API** inside `OnModelCreating`

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Product>()
        .Property(p => p.Name)
        .IsRequired()
        .HasMaxLength(100);
}
```

---

### **Video 8 — Mark Column as Required**

✅ Make sure a property is not null

```csharp
[Required]
public string Description { get; set; }
```

✅ OR with Fluent API

```csharp
modelBuilder.Entity<Product>()
    .Property(p => p.Description)
    .IsRequired();
```

---

✅ Got it! Here’s a clean note for **Video 10 — Exclude Entity From Model Or From Migrations**:

---

### **Video 10 Summary**

In this video, you learned **how to exclude an entity from the EF Core model or exclude it only from migrations.**

---

### **Key Code**

✅ **1️⃣ Exclude entity completely from the model**

```csharp
modelBuilder.Ignore<Post>();
```

👉 This tells EF Core **do not track or map the `Post` entity at all** — it will not generate any table or column for it.

---

✅ **2️⃣ Exclude entity only from migrations**

```csharp
modelBuilder.Entity<Blog>().ToTable("Blogs", b => b.ExcludeFromMigrations());
```

👉 This keeps the `Blog` entity in the model, but **migrations will ignore creating or modifying its table**.  
⚠️ Useful if the table already exists in the database and you don’t want EF to touch it.

---

**Summary in one line:**  
☝ You can **ignore an entity fully** or **keep it only for runtime, not migrations** using these patterns.

---

# Video 11 Summary

In this video, you learned how to change the table name in EF Core using two approaches:

### Key Code

✅ **1 Using DataAnnotation**

```csharp
[Table("Posts")]
public class AuditEntry
{
    public int Id { get; set; }
    public string UserName { get; set; }
    public string Action { get; set; }
}
```

👉 Maps the `AuditEntry` class to the `Posts` table.

---

✅ **2 Using Fluent API**

```csharp
modelBuilder.Entity<AuditEntry>().ToTable("Posts");
```

👉 Configured in `OnModelCreating`.

---

### Summary in one line:

☝️ You can rename the table with `[Table()]` or `.ToTable()`.

---


Here’s the full text from your second image exactly as it is:

---

# Video 12 Summary

You learned how to change the database schema and map a model to a database view.

### Key Code

✅ **1 Change schema using DataAnnotation**

```csharp
[Table("Audits", Schema = "AuditSchema")]
public class AuditEntry
{
    public int Id { get; set; }
    public string UserName { get; set; }
    public string Action { get; set; }
}
```

---

✅ **2 Change schema using Fluent API**

```csharp
modelBuilder.Entity<AuditEntry>().ToTable("Audits", schema: "Sch");
```

---

✅ **3 Map model to a database view**

```csharp
modelBuilder.Entity<AuditEntry>().ToView("AuditView", schema: "AuditSchema");
```

👉 EF Core treats views as **read-only**.

---

### Summary in one line:

☝️ Use `[Table(..., Schema=...)]`, `.ToTable(...)`, or `.ToView(...)` to customize schema/view mapping.

---

### Set Default Schema

```csharp
modelBuilder.HasDefaultSchema("AuditSch");
```

👉 All unmapped entities will default to this schema.

---

### Example

```csharp
public class Product { ... }
modelBuilder.Entity<Product>().ToTable("Products");
// ↪ Maps to: AuditSch.Products
```

---

Here’s the exact text content from your **Video 13 Summary** image:

---

# Video 13 Summary

You learned how to exclude properties from being mapped.

### Key Code

✅ **1 Using DataAnnotation**

```csharp
[NotMapped]
public DateTime AddedOn { get; set; }
```

---

✅ **2 Using Fluent API**

```csharp
modelBuilder.Entity<Blog>().Ignore(b => b.AddedOn);
```

---

### Summary in one line:

☝️ Use `[NotMapped]` or `.Ignore()` to skip properties.

---

Here’s the text content from your **Video 14 Summary** image:

---

# Video 14 Summary

You learned how to change column names in the database.

### Key Code

✅ **1 Using DataAnnotation**

```csharp
[Column("BlogUrl")]
public string Url { get; set; }
```

---

✅ **2 Using Fluent API**

```csharp
modelBuilder.Entity<Blog>()
    .Property(b => b.Url)
    .HasColumnName("BlogUrl");
```

---

### Summary in one line:

☝️ Use `[Column()]` or `.HasColumnName()` to rename columns.

---


# Video 15 Summary

You learned how to set specific SQL data types.

### Key Code

✅ **1 Using DataAnnotation**

```csharp
[Column(TypeName = "varchar(200)")]
public string Url { get; set; }
```

✅ **2 Using Fluent API**

```csharp
modelBuilder.Entity<Blog>(eb =>
{
    eb.Property(b => b.Url).HasColumnType("varchar(200)");
    eb.Property(b => b.AddedOn).HasColumnType("DateTime");
});
```

👉 **Summary in one line:**  
☝️ Use `[Column(TypeName = "...")]` or `.HasColumnType()` to control SQL types.

---

# Video 16 Summary

You learned how to set max length for strings.

### Key Code

✅ **1 Using DataAnnotation**

```csharp
[MaxLength(200)]
public string Url { get; set; }
```

✅ **2 Using Fluent API**

```csharp
modelBuilder.Entity<Blog>()
    .Property(b => b.Url)
    .HasMaxLength(200);
```

⚡ **Note:** Without max length, EF Core may use `nvarchar(max)` by default.

👉 **Summary in one line:**  
☝️ Use `[MaxLength()]` or `.HasMaxLength()` to limit string length.

---

# Video 17 Summary — Column Comments

You learned how to add comments to database columns.

### Key Code

✅ **1 Using DataAnnotation**

```csharp
[Comment("The date and time of add")]
public DateTime AddedOn { get; set; }
```

✅ **2 Using Fluent API**

```csharp
modelBuilder.Entity<Blog>()
    .Property(b => b.AddedOn)
    .HasComment("The date and time of add");
```

⚡ **Note:** Not all DBs support comments (e.g., SQL Server 2022+, PostgreSQL do).

👉 **Summary in one line:**  
☝️ Use `[Comment()]` or `.HasComment()` to document your schema.

---

# Video 18 Summary

You learned how to set a primary key.

### Key Code

✅ **1 Using DataAnnotation**

```csharp
[Key]
public int BookKey { get; set; }
```

✅ **2 Using Fluent API**

```csharp
modelBuilder.Entity<Book>()
    .HasKey(b => b.BookKey);
```

✅ **Composite Key**

```csharp
modelBuilder.Entity<Book>()
    .HasKey(b => new { b.BookKey, b.AnotherKey });
```

👉 **Summary in one line:**  
☝️ Use `[Key]` or `.HasKey()` to define primary (or composite) keys.

---

# Video 19 Summary

You learned how to rename the primary key constraint.

### Key Code

✅ **Using Fluent API**

```csharp
modelBuilder.Entity<Book>()
    .HasKey(b => b.BookKey)
    .HasName("PK_BookKey");
```

⚡ **Note:** Only Fluent API allows renaming.

👉 **Summary in one line:**  
☝️ Use `.HasName()` after `.HasKey()` to rename the PK constraint.

---

# Video 20 Summary

You learned how to define a **composite** (multi-column) primary key.

### Key Code

✅ **Using Fluent API**

```csharp
modelBuilder.Entity<Book>()
    .HasKey(b => new { b.BookKey, b.Author });
```

✅ **Rename the composite key**

```csharp
modelBuilder.Entity<Book>()
    .HasKey(b => new { b.BookKey, b.Author })
    .HasName("PK_Book_Author");
```

⚡ **Note:** Composite keys require Fluent API.

👉 **Summary in one line:**  
☝️ Use `.HasKey()` with multiple properties to define a composite PK (Fluent API only).

---

# Video 21 Summary — Default Values

You learned how to set default values for columns: fixed values and SQL expressions.

### Key Code

✅ **1 Set fixed default value**

```csharp
modelBuilder.Entity<Blog>()
    .Property(b => b.Rating)
    .HasDefaultValue(1);
```

👉 Inserts without `Rating` will default to `1`.

✅ **2 Set SQL expression as default**

```csharp
modelBuilder.Entity<Blog>()
    .Property(b => b.AddedOn)
    .HasDefaultValueSql("GETDATE()");
```

👉 Uses a SQL function like `GETDATE()` for default values.

⚡ **Notes:**

- `.HasDefaultValue()` → static values
    
- `.HasDefaultValueSql()` → dynamic expressions (`GETDATE`, `NEWID`)
    
- Works only with Fluent API
    

👉 **Summary in one line:**  
☝️ Use `.HasDefaultValue()` or `.HasDefaultValueSql()` depending on your need.

---

# Video 22 Summary — Computed Columns

You learned how to create computed columns using Fluent API.

### Key Code

```csharp
modelBuilder.Entity<Author>()
    .Property(p => p.DisplayName)
    .HasComputedColumnSql("[LastName] + ' ' + [FirstName]");
```

👉 Database calculates `DisplayName` instead of storing it manually.

✅ **Make it stored (if supported):**

```csharp
.HasComputedColumnSql("[LastName] + ' ' + [FirstName]", stored: true);
```

👉 **Summary in one line:**  
☝️ Use `.HasComputedColumnSql()` to create computed columns like full names or totals.

---


# Video 23 Summary — Auto-Increment (Identity)

Configure non-`int` primary keys (like `byte`) to auto-increment.

### Key Code

✅ **1 Using DataAnnotation**

```csharp
[DatabaseGenerated(DatabaseGeneratedOption.Identity)]
public byte Id { get; set; }
```

✅ **2 Using Fluent API**

```csharp
modelBuilder.Entity<Category>()
    .Property(b => b.Id)
    .ValueGeneratedOnAdd();
```

⚠️ **Migration Fix:** You may need to drop and re-add the column.

```csharp
migrationBuilder.DropColumn(...);
migrationBuilder.AddColumn<byte>(...)
    .Annotation("SqlServer:ValueGenerationStrategy", SqlServerValueGenerationStrategy.IdentityColumn);
```

👉 **Summary in one line:**  
☝️ Use `[DatabaseGenerated(...)]` or `.ValueGeneratedOnAdd()` and recreate the column if needed.

---

# Video 24 Summary — One-to-One Relationship

You learned how to configure 1:1 relationships in EF Core.

### Entities:

**Blog**

```csharp
public BlogImage BlogImage { get; set; }
```

**BlogImage**

```csharp
public int BlogKey { get; set; }
[ForeignKey("BlogKey")]
public Blog blog { get; set; }
```

✅ **Fluent API**

```csharp
modelBuilder.Entity<Blog>()
    .HasOne(b => b.BlogImage)
    .WithOne(c => c.blog)
    .HasForeignKey<BlogImage>(b => b.BlogKey);
```

👉 **Summary in one line:**  
☝️ Use `.HasOne().WithOne().HasForeignKey()` to define one-to-one relationships.

---

# Video 25 Summary — One-to-Many Relationship

One **Blog** → many **Posts**.

### Entities:

**Blog**

```csharp
public List<Post> posts { get; set; }
```

**Post**

```csharp
public int BlogId { get; set; }
public Blog blog { get; set; }
```

✅ **Fluent API (basic)**

```csharp
modelBuilder.Entity<Blog>()
    .HasMany(b => b.posts)
    .WithOne();
```

✅ **Fluent API (explicit with FK)**

```csharp
modelBuilder.Entity<Post>()
    .HasOne(b => b.Blog)
    .WithMany()
    .HasForeignKey(b => b.BlogId)
    .HasConstraintName("FK-test");
```

👉 **Summary in one line:**  
☝️ Use `.HasOne().WithMany().HasForeignKey()` to configure one-to-many.

---

# Video 26 Summary — Alternate & Composite Keys

✅ **Alternate Key**

```csharp
modelBuilder.Entity<RecordOfSale>()
    .HasOne(s => s.Car)
    .WithMany(b => b.SaleHistory)
    .HasForeignKey(s => s.CarLicensePlate)
    .HasPrincipalKey(o => o.LicensePlate);
```

✅ **Composite Key**

```csharp
modelBuilder.Entity<RecordOfSale>()
    .HasOne(s => s.Car)
    .WithMany(b => b.SaleHistory)
    .HasForeignKey(c => new { c.CarLicensePlate, c.carState })
    .HasPrincipalKey(o => new { o.LicensePlate, o.State });
```

👉 **Summary in one line:**  
☝️ Use `.HasPrincipalKey()` for alternate/composite keys in relationships.

---

# Video 27 Summary — Many-to-Many (Simple & Custom)

✅ **Option 1: Simple Many-to-Many**

```csharp
modelBuilder.Entity<Post>()
    .HasMany(p => p.Tags)
    .WithMany(t => t.Posts)
    .UsingEntity(j => j.ToTable("PostsTagsTest"));
```

✅ **Option 2: Custom Join Entity**

```csharp
modelBuilder.Entity<Post>()
    .HasMany(p => p.Tags)
    .WithMany(t => t.Posts)
    .UsingEntity<PostTag>(
        j => j.HasOne(pt => pt.tag).WithMany(t => t.postTags).HasForeignKey(pt => pt.TagID),
        j => j.HasOne(pt => pt.post).WithMany(p => p.postTags).HasForeignKey(pt => pt.PostID),
        j =>
        {
            j.Property(pt => pt.AddedOn).HasDefaultValueSql("GETDATE()");
            j.HasKey(t => new { t.PostID, t.TagID });
        });
```

👉 **Summary in one line:**  
☝️ Use `.UsingEntity()` for many-to-many with or without custom join data.

---

# Video 28 Summary — Indirect Many-to-Many

Manually model the join table as a full entity.

✅ **Define Composite Key**

```csharp
modelBuilder.Entity<PostTag>()
    .HasKey(t => new { t.PostID, t.TagID });
```

✅ **Set Relationships**

```csharp
modelBuilder.Entity<PostTag>()
    .HasOne(pt => pt.post)
    .WithMany(p => p.postTags)
    .HasForeignKey(pt => pt.PostID);

modelBuilder.Entity<PostTag>()
    .HasOne(pt => pt.tag)
    .WithMany(t => t.postTags)
    .HasForeignKey(pt => pt.TagID);
```

👉 **Summary in one line:**  
☝️ Model the join explicitly with `.HasKey()` and relationships for extra control.

---

# Video 29 Summary — Indexes

✅ **Using DataAnnotation**

```csharp
[Index(nameof(Title))]
public class Post { ... }
```

✅ **Using Fluent API**

```csharp
modelBuilder.Entity<Post>().HasIndex(b => b.Title);
```

✅ **Composite & Named Index**

```csharp
modelBuilder.Entity<Post>()
    .HasIndex(b => new { b.Title, b.Content })
    .HasDatabaseName("IX_Post_Title_Content");
```

✅ **Unique Index**

```csharp
.HasIndex(b => b.Title).IsUnique();
```

👉 **Summary in one line:**  
☝️ Use `[Index]` or `.HasIndex()` to optimize query performance.

---

# Video 30 Summary — Composite Indexes

✅ **DataAnnotation**

```csharp
[Index(nameof(Title), nameof(Content))]
public class Post { ... }
```

✅ **Fluent API**

```csharp
modelBuilder.Entity<Post>()
    .HasIndex(b => new { b.Title, b.Content })
    .HasDatabaseName("IX_Post_Title_Content")
    .IsUnique();
```

👉 **Summary in one line:**  
☝️ Composite indexes improve performance for multi-column filtering/sorting.

---

# Video 31 Summary

You learned how to create unique indexes to ensure that a combination of values is unique across the table.

### Key Code

✅ **1 Using DataAnnotation**

```csharp
[Index(nameof(Title), nameof(Content), IsUnique = true)]
public class Post
{
    public int PostId { get; set; }
    public string Title { get; set; }
    public string Content { get; set; }
    public List<PostTag> postTags { get; set; }
}
```

👉 Creates a unique composite index on Title + Content.  
👉 No two rows can have the same Title + Content combination.

✅ **2 Using Fluent API**

```csharp
modelBuilder.Entity<Post>()
    .HasIndex(b => new { b.Title, b.Content })
    .IsUnique();
```

👉 Same effect, configured via Fluent API.

⚡ **Extra notes:**

- Unique indexes enforce data integrity at the DB level.
    
- Help prevent duplicates in fields or combinations.
    
- You can name the unique index:
    

```csharp
modelBuilder.Entity<Post>()
    .HasIndex(b => new { b.Title, b.Content })
    .IsUnique()
    .HasDatabaseName("UX_Post_Title_Content");
```

👉 **Summary in one line:**  
☝️ Use `[Index(..., IsUnique = true)]` or `.IsUnique()` to create unique constraints on single or composite indexes.

---


# Video 32 Summary

In this video, you learned how to explicitly set a custom name for an index in EF Core.

### Key Code

✅ **1 Using DataAnnotation**

```csharp
[Index(nameof(Title), nameof(Content), Name = "Index_URL")]
public class Post
{
    public int PostId { get; set; }
    public string Title { get; set; }
    public string Content { get; set; }
    public List<PostTag> postTags { get; set; }
}
```

👉 This creates an index on Title + Content with the name `Index_URL` in the database.

---

✅ **2 Using Fluent API**

```csharp
modelBuilder.Entity<Post>()
    .HasIndex(b => new { b.Title, b.Content })
    .HasDatabaseName("Index_URL");
```

👉 Same as above, but configured using Fluent API.

⚡ **Extra notes:**

- By default, EF Core generates index names like `IX_TableName_ColumnNames`.
    
- Custom names are useful when:  
    ✅ You want clearer or more meaningful names.  
    ✅ You need to match an existing database schema.
    

---

### Summary in one line:

☝️ Use `Name` (DataAnnotation) or `.HasDatabaseName()` (Fluent API) to set a custom name for your index.

---

# Video 33 Summary

In this video, you learned how to add filters (WHERE clauses) to indexes in EF Core.  
Filtered indexes are partial indexes — they only apply to rows meeting a condition.

### Key Code

✅ **1 Create filtered index (ignore NULL titles)**

```csharp
modelBuilder.Entity<Post>()
    .HasIndex(b => new { b.Title, b.Content })
    .HasFilter("[Title] IS NOT NULL");
```

👉 This creates an index only on rows where Title IS NOT NULL.

---

✅ **2 Remove filter from unique index**

```csharp
modelBuilder.Entity<Post>()
    .HasIndex(b => new { b.Title, b.Content })
    .IsUnique()
    .HasFilter(null);
```

👉 This ensures the unique index applies to all rows, including NULL values.

⚡ **Extra notes:**  
Filtered indexes are useful for:  
✅ Excluding soft-deleted rows (IsDeleted = 0)  
✅ Ignoring NULL or default values  
✅ Optimizing for partial query patterns

---

**Example:**

```csharp
.HasFilter("[IsDeleted] = 0")
```

👉 This would exclude logically deleted rows.

⚡ Not all database providers support `.HasFilter()` (works in SQL Server).

---

### Summary in one line:

☝️ Use `.HasFilter()` to create partial (filtered) indexes that apply only to rows matching a condition.

---

# Video 34 Summary

In this video, you learned how to use database sequences in EF Core to generate custom numeric values.

### Key Concepts

✅ **What is a sequence?**  
A sequence is a database object that generates sequential numeric values, independent of any specific table.

Useful for:

- Custom order numbers
    
- Shared counters across multiple tables
    
- Non-identity, customizable numbering
    

---

### Key Code

✅ **1 Define a simple sequence**

```csharp
modelBuilder.HasSequence<int>("OrderNumber");
```

👉 Creates a sequence named `OrderNumber` (default: starts at 1, increments by 1).

---

✅ **2 Use sequence as default value for a column**

```csharp
modelBuilder.Entity<Post>()
    .Property(b => b.PostId)
    .HasDefaultValueSql("NEXT VALUE FOR OrderNumber");

modelBuilder.Entity<Tag>()
    .Property(b => b.TagId)
    .HasDefaultValueSql("NEXT VALUE FOR OrderNumber");
```

👉 Sets the column’s default value to get the next value from the sequence.

---

✅ **3 Define sequence with schema, start, and increment**

```csharp
modelBuilder.HasSequence<int>("OrderNumber", schema: "shared")
    .StartsAt(1000)
    .IncrementsBy(5);
```

👉 This:

- Creates sequence under `shared` schema
    
- Starts at 1000
    
- Increments by 5 each time
    

⚡ **Extra notes:**

- Sequences are shared — multiple tables can pull values from the same sequence.
    
- They work independently from identity columns.
    
- Supported mainly in SQL Server and PostgreSQL.
    

---

### Summary in one line:

☝️ Use `.HasSequence()` and `NEXT VALUE FOR` to generate custom sequential numbers across entities.

---

# Video 36 Summary: Manage Migrations and Generate SQL Scripts

This video explains the key EF Core migration commands, both in Visual Studio (Package Manager Console) and .NET Core CLI, with their purpose and when to use them.

---

🛠 **1 Add New Migration**

**Visual Studio:**

```sql
Add-Migration AddNewTables
```

**.NET Core CLI:**

```csharp
dotnet ef migrations add AddNewTables
```

✅ **Purpose:**  
Create a new migration file that tracks schema changes (e.g., new tables, columns) compared to the last migration.

💡 **When to use:**  
Whenever you make model changes you want to apply to the database.

---

🛠 **2 Update Database to Latest Migration**

**Visual Studio:**

```pgsql
Update-Database
```

**.NET Core CLI:**

```pgsql
dotnet ef database update
```

✅ **Purpose:**  
Apply all pending migrations to the database.

💡 **When to use:**  
To sync your database with the latest model after adding migrations.

---

🛠 **3 Update Database to Specific Migration**

**Visual Studio:**

```pgsql
Update-Database AddNewTables
```

**.NET Core CLI:**

```pgsql
dotnet ef database update AddNewTables
```

✅ **Purpose:**  
Roll back or advance the database to a specific migration.

💡 **When to use:**  
For controlled migration steps, or to undo later migrations temporarily.

---

🛠 **4 Remove Last Migration**

**Visual Studio:**

```mathematica
Remove-Migration
```

**.NET Core CLI:**

```arduino
dotnet ef migrations remove
```

✅ **Purpose:**  
Delete the most recent migration file.

💡 **When to use:**  
If you added a migration by mistake or before applying it.

---

🛠 **5 Generate SQL Script (from blank to latest)**

**Visual Studio:**

```nginx
Script-Migration
```

**.NET Core CLI:**

```nginx
dotnet ef migrations script
```

✅ **Purpose:**  
Generate a SQL script to create the full database schema from scratch.

💡 **When to use:**  
To apply migrations on a production database using raw SQL.

---

🛠 **6 Generate SQL Script (from specific migration to latest)**

**Visual Studio:**

```nginx
Script-Migration AddNewTables
```

**.NET Core CLI:**

```nginx
dotnet ef migrations script AddNewTables
```

✅ **Purpose:**  
Create a SQL script for applying changes from a specific migration to the latest.

💡 **When to use:**  
When you need to update production databases from a known state.

---

🛠 **7 Generate SQL Script (from one migration to another)**

**Visual Studio:**

```nginx
Script-Migration AddNewTables AddAuditTable
```

**.NET Core CLI:**

```nginx
dotnet ef migrations script AddNewTables AddAuditTable
```

✅ **Purpose:**  
Create a SQL script to move between two specific migrations.

💡 **When to use:**  
For partial updates or controlled rollouts.

---

🛠 **8 List Migrations**

**Visual Studio:**

```sql
Get-Migration
```

**.NET Core CLI:**

```nginx
dotnet ef migrations list
```

✅ **Purpose:**  
List all migrations in the project.

💡 **When to use:**  
To check which migrations exist and track migration history.

⚡ **Summary in one line:**  
☝️ These commands help you create, apply, remove, script, and inspect migrations, letting you control how your database schema evolves alongside your code.


---

## ✅ Video 37 Summary

In this video, you learned how to reverse-engineer an existing database into EF Core models and DbContext using scaffolding commands.

This is called Database-First approach (as opposed to Code-First).

💡 **What is scaffolding?**  
`scaffold-dbcontext` is a command that:  
✅ Reads an existing database schema  
✅ Generates:

- Entity classes (models)
    
- DbContext class
    

This lets you start using EF Core with an existing database.

---

## ✅ Common scaffold-dbcontext command examples

### 1️⃣ Basic scaffolding — full database

```bash
scaffold-dbcontext "Server=.;Database=DigitalLibraryDB;User Id=sa;Password=123456;TrustServerCerti...
```

👉 Generates all tables + context into the current folder.

---

### 2️⃣ Scaffold only specific table (e.g., Book)

```bash
scaffold-dbcontext "...connection..." Microsoft.EntityFrameworkCore.SqlServer -tables book
```

👉 Generates only the Book table entity + context.

---

### 3️⃣ Scaffold into specific folder (models)

```bash
scaffold-dbcontext "...connection..." Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models
```

👉 Puts the entity classes into a Models folder.

---

### 4️⃣ Scaffold with context in a separate folder

```bash
scaffold-dbcontext "...connection..." Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models -c...
```

👉 Puts:  
Entities → Models folder  
DbContext → Data folder

---

### 5️⃣ Scaffold with only context folder

```bash
scaffold-dbcontext "...connection..." Microsoft.EntityFrameworkCore.SqlServer -contextDir Data
```

👉 DbContext goes to Data; entities stay in root or default.

---

### 6️⃣ Scaffold with custom context name

```bash
scaffold-dbcontext "...connection..." Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models -c...
```

👉 Renames DbContext to ApplicationDbContext.

---

### 7️⃣ Scaffold using DataAnnotations instead of Fluent API

```bash
scaffold-dbcontext "...connection..." Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models -d...
```

👉 Generates entity classes using `[Key]`, `[Required]`, etc., instead of Fluent API.

---

⚡ Extra notes:

- Add `-f` (force) to overwrite existing files.
    
- Add `-Schemas` if you want to scaffold specific schemas.
    
- Useful for quick starts with legacy or third-party databases.
    

✅ **Summary in one line:**  
👉 Use `scaffold-dbcontext` to generate EF Core models and context from an existing database, with options for folders, context name, tables, and annotations.

---

## ✅ Video 38 — Select All Data, Select One Item Using `.Find()`

### ✅ Select all data

```csharp
var stocks = _Context.Stock.ToList();
```

👉 Retrieves all rows from the Stock table.

---

### ✅ Select one item by primary key using `.Find()`

```csharp
var stock = _Context.Stock.Find(1);
```

👉 Looks up the entity with primary key = 1.

- Faster because it checks the EF change tracker before hitting the database.
    
- Works only with primary keys.
    

---

## ✅ Video 39 — Select One Item Using `.Single()` / `.SingleOrDefault()`

### ✅ Single

```csharp
var stock = _Context.Stock.Single(s => s.Symbol == "AAPL");
```

👉 Retrieves exactly one record matching the condition.  
⚠️ Throws an exception if:

- No match
    
- More than one match
    

---

### ✅ SingleOrDefault

```csharp
var stock = _Context.Stock.SingleOrDefault(s => s.Symbol == "AAPL");
```

👉 Same as `.Single()`, but:

- Returns null if no match.
    
- Still throws if more than one match.
    

---

## ✅ Video 40 — Select One Item Using `.First()` / `.FirstOrDefault()`

### ✅ First

```csharp
var stock = _Context.Stock.First(s => s.Sector == "Tech");
```

👉 Returns the first matching record.  
⚠️ Throws if no match.

---

### ✅ FirstOrDefault

```csharp
var stock = _Context.Stock.FirstOrDefault(s => s.Sector == "Tech");
```

👉 Returns the first match, or null if no match.

---

## ✅ Video 41 — Select One Item Using `.Last()` / `.LastOrDefault()`

⚠️ `.Last()` requires ordering because SQL does not have a concept of last without `ORDER BY`.

### ✅ Example:

```csharp
var stock = _Context.Stock
    .OrderBy(s => s.Id)
    .Last();
```

👉 Retrieves the last item by Id order.

---

### ✅ LastOrDefault

```csharp
var stock = _Context.Stock
    .OrderBy(s => s.Id)
    .LastOrDefault();
```

👉 Same as above, but returns null if no items.

---

## ⚡ Summary table

|Method|Behavior|
|---|---|
|`.ToList()`|Get all records|
|`.Find()`|Get by primary key (fast)|
|`.Single()`|Get exactly one (error if 0 or >1)|
|`.SingleOrDefault()`|Get one or null (error if >1)|
|`.First()`|Get first matching (error if none)|
|`.FirstOrDefault()`|Get first or null|
|`.Last()`|Get last after ordering (error if none)|
|`.LastOrDefault()`|Get last or null (requires ordering)|

---

## ✅ Video 42 — Filtering Data Using `.Where()`

```csharp
var techStocks = _Context.Stock.Where(s => s.Sector == "Tech").ToList();
```

👉 Filters records matching the condition `(Sector == "Tech")`.

- Returns an `IQueryable` — you can chain more LINQ methods.  
    💡 Note: Nothing is executed until you call `.ToList()`, `.First()`, etc.
    

---

## ✅ Video 43 — `.Any()` vs `.All()` (Boolean checks)

### ✅ .Any()

```csharp
bool hasTech = _Context.Stock.Any(s => s.Sector == "Tech");
```

👉 Checks if at least one record matches.

- Returns true or false.
    

---

### ✅ .All()

```csharp
bool allTech = _Context.Stock.All(s => s.Sector == "Tech");
```

👉 Checks if all records match the condition.

- Returns true or false.
    

---

## ✅ Video 44 — `.Append()` vs `.Prepend()` (do they affect the database?)

### ✅ Example

```csharp
var stocks = _Context.Stock.ToList()
    .Append(new Stock { Name = "Test" })
    .Prepend(new Stock { Name = "First" });
```

👉 Adds items in memory after querying —  
⚠️ does NOT modify the database!

- `.Append()` → adds item at the end.
    
- `.Prepend()` → adds item at the start.  
    💡 Used only in C# memory, not translated to SQL.
    



---

### ✅ Video 45 — `.Average()` vs `.Count()` vs `.Sum()` vs `.LongCount()`

✔️ **.Average()**

```csharp
var avgBalance = _Context.Stock.Average(s => s.Balance);
```

✔️ **.Count()**

```csharp
var count = _Context.Stock.Count();
```

✔️ **.LongCount()**

```csharp
var longCount = _Context.Stock.LongCount();
```

✔️ **.Sum()**

```csharp
var totalBalance = _Context.Stock.Sum(s => s.Balance);
```

---

### ✅ Video 46 — `.Max()` vs `.Min()`

✔️ **.Max()**

```csharp
var maxBalance = _Context.Stock.Max(s => s.Balance);
```

✔️ **.Min()**

```csharp
var minBalance = _Context.Stock.Min(s => s.Balance);
```

---

### ⚡ Summary Table

|Method|Purpose|
|---|---|
|`.Where()`|Filter records|
|`.Any()`|Check if any record matches|
|`.All()`|Check if all records match|
|`.Append()`|Add item at end (in-memory only)|
|`.Prepend()`|Add item at start (in-memory only)|
|`.Average()`|Calculate average|
|`.Count()`|Count items (int)|
|`.LongCount()`|Count items (long)|
|`.Sum()`|Sum values|
|`.Max()`|Get max value|
|`.Min()`|Get min value|

---

### ✅ Video 47 — Data Sorting Using `.OrderBy()`, `.OrderByDescending()`, `.ThenBy()`

✔️ **Ascending sort**

```csharp
var stocks = _Context.Stock.OrderBy(s => s.Name).ToList();
```

👉 Sorts by Name A–Z.

✔️ **Descending sort**

```csharp
var stocks = _Context.Stock.OrderByDescending(s => s.Name).ToList();
```

👉 Sorts by Name Z–A.

✔️ **Multiple levels (secondary sorting)**

```csharp
var stocks = _Context.Stock
    .OrderBy(s => s.Sector)
    .ThenBy(s => s.Name)
    .ToList();
```

👉 First by Sector, then by Name inside each sector.

---

### ✅ Video 48 — Projection Using `.Select()`

✔️ **Example**

```csharp
var stockNames = _Context.Stock.Select(s => s.Name).ToList();
```

👉 Selects only the `Name` column.

✔️ **Select multiple fields into anonymous object**

```csharp
var stocks = _Context.Stock
    .Select(s => new { s.Name, s.Symbol })
    .ToList();
```

👉 Selects only needed fields, improves performance.

---

### ✅ Video 49 — Select Unique Values Using `.Distinct()` with `.Select()`

✔️ **Example**

```csharp
var industries = _Context.Stock
    .Select(s => s.Industry)
    .Distinct()
    .ToList();
```

👉 Returns a list of unique industry names.

⚠️ **Why project first?**  
Because selecting full entity (`Stock`) includes the primary key, making all rows unique.

---

### ✅ Video 50 — `.Take()` vs `.Skip()`

✔️ **.Take() — get first N records**

```csharp
var top5 = _Context.Stock.Take(5).ToList();
```

👉 Fetches the first 5 records.

✔️ **.Skip() — skip N records**

```csharp
var skip5 = _Context.Stock.Skip(5).ToList();
```

👉 Skips the first 5, returns the rest.

✔️ **Combined paging**

```csharp
var page2 = _Context.Stock
    .Skip(5)
    .Take(5)
    .ToList();
```

👉 Skips first 5, then takes next 5 → useful for pagination.

---

### ⚡ Summary Table

|Method|Purpose|
|---|---|
|`.OrderBy()`|Sort ascending|
|`.OrderByDescending()`|Sort descending|
|`.ThenBy()`|Secondary sorting|
|`.Select()`|Project specific fields|
|`.Distinct() + .Select()`|Get unique values (avoid PK interference)|
|`.Take()`|Take top N rows|
|`.Skip()`|Skip first N rows|

---

### ✅ Video 51 — Grouping Data Using `.GroupBy()`

✔️ **Basic example**

```csharp
var groupedStocks = _Context.Stock
    .GroupBy(s => s.Industry)
    .ToList();
```

👉 Groups all stocks by their Industry.

✔️ **Group + Select key + items**

```csharp
var groupedStocks = _Context.Stock
    .GroupBy(s => s.Industry)
    .Select(g => new
    {
        Industry = g.Key,
        Count = g.Count(),
        Stocks = g.ToList()
    })
    .ToList();
```

👉 For each group:

- `g.Key` → Industry name
    
- `g.Count()` → number of items
    
- `g.ToList()` → list of Stock in that group
    

✔️ **Group + Aggregation**

```csharp
var industryBalances = _Context.Stock
    .GroupBy(s => s.Industry)
    .Select(g => new
    {
        Industry = g.Key,
        TotalBalance = g.Sum(s => s.Balance),
        AverageBalance = g.Average(s => s.Balance)
    })
    .ToList();
```

👉 Example aggregations:

- Sum
    
- Average
    
- Min
    
- Max
    

⚡ **Notes:**

- `.GroupBy()` → translates to SQL `GROUP BY` when used before `.ToList()`.
    
- Always combine with `.Select()` for meaningful summaries.
    
- Avoid bringing full groups into memory unless needed.
    

✅ **Summary in one line:**  
👉 Use `.GroupBy()` with `.Select()` to group data by a key and apply counts, sums, or custom projections.



---

## ✅ Video 52 — Inner Join Using `.Join()`

This video shows how to perform INNER JOINs in EF Core using the LINQ `.Join()` extension method.

### ✅ Example Code Breakdown

```csharp
var books = _context.Books
    .Join(
        _context.Authors,
        book => book.AuthorId,
        author => author.Id,
        (book, author) => new
        {
            BookId = book.Id,
            BookName = book.Name,
            AuthorName = author.Name,
            AuthorNationalityId = author.NationalityId
        }
    )
    .Join(
        _context.Nationalities,
        book => book.AuthorNationalityId,
        nationality => nationality.Id,
        (book, nationality) => new
        {
            book.BookId,
            book.BookName,
            book.AuthorName,
            AuthorNationality = nationality.Name
        }
    )
    .ToList();
```

---

### ✅ What’s happening here?

🔹 1 First join (Books + Authors):  
Join Books on Authors: `book.AuthorId == author.Id`  
Select: Book ID, Name, Author Name, Author NationalityId

🔹 2 Second join (Result + Nationalities):  
Join previous result on Nationalities:  
`book.AuthorNationalityId == nationality.Id`  
Select: Book ID, Name, Author Name, Nationality Name

🔹 3 Loop to display

```csharp
foreach (var book in books)
{
    System.Console.WriteLine($"{book.BookId} - {book.BookName} - {book.AuthorName}");
}
```

👉 Displays BookId, BookName, AuthorName.

⚡ Notes:  
✅ `.Join()` is a LINQ extension method.  
✅ It works well for inner joins; for outer joins, you typically use `.GroupJoin()` or navigation properties with `.Include()`.

✅ **Summary in one line:**  
👉 Use `.Join()` to combine data from multiple tables by matching keys and projecting combined results.

---

## ✅ Video 53 — [Arabic] EF Core: Left Join Using `.GroupJoin()` (Extension Method)

This video explains how to simulate a LEFT JOIN in Entity Framework using `.GroupJoin()` + `.SelectMany()`.

---

### 🟦 Query Breakdown:

✅ Step-by-step logic from the image:

```csharp
var books = _context.Books
    .Join(
        _context.Authors,
        book => book.AuthorId,
        author => author.Id,
        (book, author) => new
        {
            BookId = book.Id,
            BookName = book.Name,
            AuthorName = author.Name,
            AuthorNationalityId = author.NationalityId
        }
    )
    .GroupJoin(
        _context.Nationalities,
        book => book.AuthorNationalityId,
        nationality => nationality.Id,
        (book, nationality) => new
        {
            Book = book,
            Nationality = nationality
        }
    )
    .SelectMany(
        b => b.Nationality.DefaultIfEmpty(),
        (b, n) => new { b.Book, Nationality = n }
    );
```

---

### 💡 How it works:

|Step|Operation|Purpose|
|---|---|---|
|Join|Books with Authors|To get author data for each book|
|GroupJoin|With Nationalities|This simulates LEFT JOIN|
|SelectMany + DefaultIfEmpty()|Flatten results|Ensures unmatched authors still appear with null nationality|

---

### ✅ Example Output:

```plaintext
BookId: 1 - BookName: Clean Code - AuthorName: Robert Martin - Nationality: American
BookId: 2 - BookName: Unknown Book - AuthorName: Unknown Author - Nationality:
```

---

✅ **Summary in one line:**  
👉 Use `.GroupJoin()` with `.SelectMany()` and `.DefaultIfEmpty()` to perform LEFT JOINs in LINQ when joining multiple tables and keeping unmatched data.

---

## ✅ Video 54 — Tracking vs. NoTracking (`AsNoTracking`)

This video explains how EF Core tracks entities and when to use no-tracking queries for better performance.

---

### ✅ What is tracking?

By default, EF Core tracks entities you query:

- It keeps them in the `ChangeTracker`.
    
- It can detect changes and save them later using `SaveChanges()`.
    

Example:

```csharp
var stock = _context.Stock.First();
stock.Name = "Updated Name";
_context.SaveChanges(); // EF knows to update the DB
```

---

### ✅ What is no-tracking?

You tell EF Core **not** to track query results, meaning:

- They are read-only objects.
    
- Faster performance, lower memory use.
    
- You cannot update them unless you manually attach them.
    

✅ How to use no-tracking:

🔹 Per query:

```csharp
var stocks = _context.Stock.AsNoTracking().ToList();
```

🔹 Globally (for all queries in this context):

```csharp
_context.ChangeTracker.QueryTrackingBehavior = QueryTrackingBehavior.NoTracking;
```

---

⚡ When to use `AsNoTracking()`?  
✅ For read-only operations (e.g., showing data in UI, reports).  
✅ When you don’t need to modify and save entities.  
❌ Don’t use if you plan to update entities.

✅ **Summary in one line:**  
👉 Use `AsNoTracking()` or global `QueryTrackingBehavior.NoTracking` to speed up read-only queries by disabling EF Core’s change tracking.

---

## ✅ Video 55 — Eager Loading (`Include` / `ThenInclude`)

This video explains how to load related data along with main entities using eager loading.

---

### ✅ Example Code

```csharp
var book = _context.Books
    .Include(b => b.Author)
    .ThenInclude(a => a.Nationality)
    .SingleOrDefault(b => b.Id == 1);
```

👉 What this does:

- Load Book with `Id == 1`.
    
- Include the related `Author`.
    
- Include the related `Author.Nationality`.
    

Without eager loading, EF Core would not load related entities automatically.

---

### ✅ What is eager loading?

- `.Include()` → load related entities (1 level deep).
    
- `.ThenInclude()` → load deeper levels (nested navigation).
    

Example:  
Book → Author → Nationality

---

⚡ Performance impact:  
✅ Benefits:

- Fewer database roundtrips (single query for all needed data).
    

⚠️ Drawbacks:

- More data transferred, even if you don’t use all fields.
    
- Complex SQL queries, which can be slower.
    

👉 Tip: Only eager load what you actually need.

---

💡 Alternatives:

- Lazy loading → load related data only when accessed (requires setup).
    
- Explicit loading → manually load related data with separate queries.
    

✅ **Summary in one line:**  
👉 Use `Include()` and `ThenInclude()` for eager loading related data, but be mindful — it can impact performance if overused.



---

✅ **Video 56 — Explicit Loading**  
This video explains how to manually load related data after querying an entity — this is called **Explicit Loading**.

---

✅ **Example 1 — Load single related entity (Reference)**

```csharp
var book = _context.Books.SingleOrDefault(b => b.Id == 2);

_context.Entry(book)
    .Reference(b => b.Author)
    .Load();

Console.WriteLine(book.Author.Name);
```

👉 What’s happening:

- Query Book with `Id 2`.
    
- Explicitly load related Author.
    
- Access `book.Author.Name`.
    

---

✅ **Example 2 — Load related collection (Collection)**

```csharp
var blog = _context.Blogs.SingleOrDefault(b => b.BlogId == 5);

_context.Entry(blog)
    .Collection(b => b.Posts)
    .Load();

foreach (var post in blog.Posts)
{
    Console.WriteLine(post.Title);
}
```

👉 What’s happening:

- Query Blog with `BlogId 5`.
    
- Explicitly load its `Posts` collection.
    
- Loop through posts and print titles.
    

---

✅ **Example 3 — Filter while loading collection**

```csharp
_context.Entry(blog)
    .Collection(b => b.Posts)
    .Query()
    .Where(p => p.PostId > 2)
    .ToList();
```

👉 What’s happening:

- Load only Posts with `PostId > 2`.
    
- Still attaches the filtered posts to `blog.Posts`.
    

⚡ Notes  
✅ Explicit Loading:

- Gives you control over when and what to load.
    
- Useful when lazy loading is disabled or not configured.
    
- Can be combined with filtering for optimized loading.
    

⚠️ Be careful — if you loop over related collections before loading, they will be empty.

✅ **Summary in one line:**  
👉 Use `.Entry().Reference().Load()` or `.Entry().Collection().Load()` for manually fetching related entities after the main query.

---

✅ **Video 57 — Lazy Loading**  
This video explains lazy loading in EF Core —  
a technique where related data is loaded automatically on first access.

---

✅ **How lazy loading works**  
You query an entity (e.g., Book),  
but related data (like `Book.Author`) is not loaded immediately.

When you first access the navigation property (`book.Author.Name`),  
EF Core automatically fetches it from the database.

---

✅ **How to enable lazy loading**

1️⃣ Install NuGet package

```
Microsoft.EntityFrameworkCore.Proxies
```

2️⃣ Configure in DbContext

```csharp
options.UseLazyLoadingProxies()
    .UseSqlServer("Data Source=(localdb)\\ProjectModels;Initial Catalog=EFCorePractice;Integrated Security=True");
```

👉 This enables lazy loading via proxies.

---

3️⃣ **Mark navigation properties as virtual**

```csharp
public virtual Author Author { get; set; }
public virtual ICollection<Post> Posts { get; set; }
```

👉 EF Core creates proxy classes to intercept property access.

⚡ Notes  
✅ Advantages:

- Very easy, minimal code.
    
- Automatically loads related data when needed.
    

⚠️ Disadvantages:

- Hidden queries can lead to N+1 problems (many small SQL queries).
    
- Harder to control performance.
    
- Not recommended for large or critical queries — prefer eager or explicit loading.
    

✅ **Summary in one line:**  
👉 Enable lazy loading with `.UseLazyLoadingProxies()` and virtual navigation properties, but use carefully to avoid hidden performance costs.

---

✅ **Video 58 — Split Queries in EF Core**  
This video explains how EF Core handles loading related data using `.Include()`, and how you can control performance using split vs single queries.

---

✅ **What is a split query?**  
When you use `.Include()` to load related data (e.g., Blog + Posts),  
EF Core by default tries to generate a single complex SQL query with JOINs.

This can:

- Cause duplicate rows
    
- Be slow or error-prone with large data sets or deep nesting
    

---

✅ **Use `.AsSplitQuery()` to improve performance**

```csharp
var blogs = _Context.Blogs
    .Include(blog => blog.Posts)
    .AsSplitQuery()
    .ToList();
```

👉 EF Core sends multiple simpler queries:

- First: `SELECT * FROM Blogs`
    
- Then: `SELECT * FROM Posts WHERE BlogId IN (...)`
    

✅ Better for large or deeply nested data loads.

---

✅ **Use `.AsSingleQuery()` to force default JOIN behavior**

```csharp
var blogs = _Context.Blogs
    .Include(blog => blog.Posts)
    .AsSingleQuery()
    .ToList();
```

👉 Forces EF Core to load all data in one JOIN query (may be faster in small data sets).

---

✅ **Set default behavior globally**

```csharp
options.UseSqlServer(
    "Data Source=(localdb)\\ProjectModels;Initial Catalog=EFCorePractice;Integrated Security=True",
    o => o.UseQuerySplittingBehavior(QuerySplittingBehavior.SplitQuery)
);
```

👉 Sets `SplitQuery` as the default behavior for the whole context.

---

⚡ **When to use what?**

|Scenario|Use|
|---|---|
|Small data, few includes|`.AsSingleQuery()` (default)|
|Large collections, deep includes|`.AsSplitQuery()`|
|Prevent duplication or memory issues|`.AsSplitQuery()`|

✅ **Summary in one line:**  
👉 Use `.AsSplitQuery()` to load related data in multiple queries to avoid JOIN overload, and `.AsSingleQuery()` when a single JOIN is preferred.

---

✅ **Video 59 — Join Using LINQ (Query Syntax)**  
This video demonstrates how to perform a multi-table JOIN using LINQ query syntax, including a LEFT JOIN using `into` + `DefaultIfEmpty()`.

---

✅ **Full LINQ Query Code:**

```csharp
var books = (
    from books in _Context.Books
    join author in _Context.Author
        on books.AuthorId equals author.Id
    join nationalities in _Context.Nationalities
        on author.NationalitiesId equals nationalities.Id
        into AuthorNationality
    from authNat in AuthorNationality.DefaultIfEmpty()
    select new
    {
        BookId = books.Id,
        BookName = books.Name,
        AuthorName = author.Name
    }
).ToList();
```

---

✅ **Step-by-step Explanation**

1️⃣ **FROM Clause — Main Table**

```csharp
from books in _Context.Books
```

✅ Selects from the Books table as the main source.

---

2️⃣ **INNER JOIN — Books → Author**

```csharp
join author in _Context.Author
    on books.AuthorId equals author.Id
```

✅ Joins Books with Author where `books.AuthorId == author.Id`.  
This is a standard INNER JOIN.

---

3️⃣ **LEFT JOIN — Author → Nationalities**

```csharp
join nationalities in _Context.Nationalities
    on author.NationalitiesId equals nationalities.Id
    into AuthorNationality
```

✅ This creates a group join, preparing for a LEFT JOIN by using `into`.

---

4️⃣ **DefaultIfEmpty — Emulate LEFT JOIN**

```csharp
from authNat in AuthorNationality.DefaultIfEmpty()
```

✅ Converts the group join into a LEFT JOIN:  
If the nationality doesn’t exist, `authNat` will be null.

---

5️⃣ **SELECT projection**

```csharp
select new
{
    BookId = books.Id,
    BookName = books.Name,
    AuthorName = author.Name
}
```

✅ Projects selected fields into an anonymous object.  
You can also include `authNat?.Name` if needed.

---

🧠 **Why use this syntax?**

- More readable for SQL-minded devs
    
- Easier to write complex joins compared to method-based `.Join()` chaining
    
- Great for LEFT JOINs using `into ... DefaultIfEmpty()`
    

✅ **Summary in one line:**  
👉 Use LINQ query syntax for clear multi-table joins, especially when doing LEFT JOINs with `into` and `DefaultIfEmpty()`.



---

## ✅ Video 60 — Select Data Using SQL or Stored Procedure (`Raw SQL Queries`)

This video shows how to execute raw SQL queries in Entity Framework Core using `.FromSqlRaw()`.

### ✅ Basic Example – Raw SQL SELECT

```csharp
var stocks = _Context.Stock
    .FromSqlRaw("SELECT * FROM Stock");

foreach (var stock in stocks)
{
    Console.WriteLine(stock.Name);
}
```

✔ Iterates through the result and prints each stock's name.

---

### ✅ Use with Stored Procedure

```csharp
var stocks = _Context.Stock
    .FromSqlRaw("GetAllStocks"); // Name of stored procedure
```

👉 You can call a stored procedure using the same method.  
⚠ Just make sure:

- The result maps exactly to the `Stock` entity.
    
- The stored procedure returns data with matching column names.
    

---

### ⚠ Limitations

- Must return columns that match the entity (`Stock`) exactly.
    
- No automatic tracking unless configured — acts like `AsNoTracking()` by default in EF Core 5+.
    
- Use parameters securely with `FromSqlInterpolated()` to avoid SQL injection.
    

---

### ✅ Alternative (With parameters)

```csharp
var stocks = _Context.Stock
    .FromSqlInterpolated($"SELECT * FROM Stock WHERE Symbol = {symbol}");
```

✔ Summary in one line:  
👉 Use `.FromSqlRaw()` or `.FromSqlInterpolated()` to run direct SQL queries or call stored procedures in EF Core.

---

### 2️⃣ Add a DbSet for the DTO

```csharp
public DbSet<StockDto> StockDto { get; set; }
```

⚠ Required so EF Core can map the result into `StockDto`.  
⚠ Does not mean a table will be created.

---

### 3️⃣ Execute the stored procedure

```csharp
var stocks = _Context.StockDto
    .FromSqlRaw("StoredProcedure");

foreach (var stock in stocks)
{
    Console.WriteLine(stock.Name);
}
```

✔ EF Core maps the result to `StockDto`.

---

### 4️⃣ Prevent EF from creating a table for the DTO

Inside `OnModelCreating`:

```csharp
modelBuilder.Entity<StockDto>(e =>
{
    e.HasNoKey();
    e.ToView(null);
});
```

✔ This tells EF:

- This DTO has no primary key.
    
- Not tied to a real table or view.
    
- EF should not include it in migrations.
    

⚡ Notes:

- Works with raw SQL and stored procedures.
    
- Useful for reporting, dashboard data, or read-only views.
    
- Columns returned must exactly match DTO properties.
    

👉 Summary in one line:  
Use `DbSet<T>`, mark with `.HasNoKey().ToView(null)`, and run with `.FromSqlRaw()`.

---

## ✅ Video 62 — Global Query Filters

This video shows how to apply automatic, reusable filtering logic across all queries using EF Core's Global Query Filters.

### ✅ What is a Global Query Filter?

A global query filter is a condition EF Core automatically applies to all queries for a specific entity.

---

### ✅ Example: Filter Soft-Deleted Rows

#### 📦 Model Setup

```csharp
public class Stock
{
    public int Id { get; set; }
    public string Name { get; set; }
    public bool isDeleted { get; set; }  // Soft-delete flag
}
```

#### 🛠 Apply Global Filter in DbContext

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Stock>().HasQueryFilter(b => !b.isDeleted);
}
```

✔ EF Core automatically adds `WHERE isDeleted = false`.

---

### ✅ Query Behavior

```csharp
var stocks = _Context.Stock.ToList();
```

Behind the scenes:

```sql
SELECT * FROM Stock WHERE isDeleted = 0
```

Equivalent to:

```csharp
var stocks = _Context.Stock.Where(b => !b.isDeleted).ToList();
```

✔ Applied automatically.

---

### ✅ Use Cases

- Soft deletes (`isDeleted`)
    
- Multi-tenancy (e.g., `TenantId == currentTenantId`)
    
- Archiving or status-based filtering
    

---

### ⚠ How to Bypass Filter

```csharp
var allStocks = _Context.Stock.IgnoreQueryFilters().ToList();
```

✔ Summary in one line:  
👉 Use `HasQueryFilter()` for automatic filters (soft delete, multi-tenancy), and `IgnoreQueryFilters()` to bypass.

---

## ✅ Video 63 — Adding Records & Saving Related Data

This video demonstrates multiple ways to insert data into the database using EF Core:

- Single records
    
- Batches (`AddRange`)
    
- Related entities (1-to-1, 1-to-many)
    

---

### 1️⃣ Add a Single Record

```csharp
var n = new Nationality { Name = "Jordan" };
_Context.nationalities.Add(n);
```

---

### 2️⃣ Add Multiple Records (Batch Insert)

```csharp
var nationalities = new List<Nationality> {
    new Nationality { Name = "Nationality 1" },
    new Nationality { Name = "Nationality 2" },
    new Nationality { Name = "Nationality 3" },
    new Nationality { Name = "Nationality 4" }
};

_Context.nationalities.AddRange(nationalities);
```

---

### 3️⃣ Add Entity + Related Entity (1-to-1)

```csharp
var author = new Author
{
    Name = "New Author",
    Nationality = new Nationality { Name = "Nationality 6" }
};
```

---

### 4️⃣ Add Entity with Related Collection (1-to-many)

```csharp
var blog = new Blog
{
    FormUrlEncodedContent = "www.google.com",
    AddedOn = DateTime.Now,
    Posts = new List<Post>
    {
        new Post { Title = "Title 1", Content = "Content 1" },
        new Post { Title = "Title 2", Content = "Content 2" },
        new Post { Title = "Title 3", Content = "Content 3" },
        new Post { Title = "Title 4", Content = "Content 4" },
        new Post { Title = "Title 5", Content = "Content 5" }
    }
};
```

---

### ✅ Final step — Save all changes

```csharp
_Context.SaveChanges();
```

---

⚡ Best Practices:

- Structure relationships correctly (Author ↔ Nationality, Blog ↔ Posts).
    
- Use `AddRange` for batch inserts.
    

✔ Summary in one line:  
👉 Use `Add`, `AddRange`, and navigation properties to insert single, batch, and related data efficiently in EF Core — all saved with `SaveChanges()`.



---

# 🎥 Video 64 — Update Record(s) in EF Core

This video demonstrates multiple ways to update data in EF Core, including:

* Simple updates
* Detached updates
* Selective property updates
* Batch updates

---

### ✅ 1. Update Using Tracked Entity (Find)

```csharp
var n = _Context.nationalities.Find(10);
n.Name = "new National";
```

👉 EF Core tracks `n`, so changes are saved on `SaveChanges()`.

---

### ✅ 2. Update Using Detached Object

```csharp
var n2 = new Nationality { Id = 10, Name = "New 2" };
_Context.Update(n2);
```

👉 For disconnected state — EF updates all properties.

---

### ✅ 3. Copy Values from Another Object

```csharp
var n3 = _Context.nationalities.Find(10);
var n4 = new Nationality { Id = 10, Name = "New 2" };
_Context.Entry(n3).CurrentValues.SetValues(n4);
```

👉 Copies values from `n4` into tracked `n3`.

---

### ✅ 4. Update Selective Properties

```csharp
var stock = new Stock
{
    Id = 1,
    Name = "new",
    Sector = "salaam",
    Industry = "n/a"
};

_Context.Update(stock);

_Context.Entry(stock).Property(b => b.Name).IsModified = false;
_Context.Entry(stock).Property(b => b.Symbol).IsModified = false;
_Context.Entry(stock).Property(b => b.Balance).IsModified = false;
```

👉 Only `Sector` and `Industry` are updated.

---

### ✅ 5. Bulk Update Using UpdateRange

```csharp
var stocks = _Context.stocks.Where(s => s.Industry == "n/a").ToList();
foreach (var stock1 in stocks)
{
    stock1.Industry = "n/aaa";
}
_Context.UpdateRange(stocks);
```

---

### ✅ Save Changes to DB

```csharp
_Context.SaveChanges();
```

---

### 🧠 Summary of Methods

| Method                              | Use Case                         |
| ----------------------------------- | -------------------------------- |
| `Find() + modify`                   | Recommended if entity is tracked |
| `Update(entity)`                    | For disconnected objects         |
| `Entry().CurrentValues.SetValues()` | Map data from another object     |
| `Entry().Property().IsModified = …` | Update only specific fields      |
| `UpdateRange(list)`                 | Batch update                     |

👉 **Summary in one line:**
EF Core allows updates through tracked objects, detached replacements, selective field changes, and batch processing — finalized via `SaveChanges()`.

---

# 🎥 Video 65 — Remove Record(s) in EF Core

This video shows how to delete one or more records using EF Core.

---

### ✅ 1. Delete a Single Record

```csharp
var stc = _Context.stocks.Find(4);
_Context.stocks.Remove(stc);
```

---

### ✅ 2. Delete Multiple Records (Batch Delete)

```csharp
var stcs = _Context.stocks
    .Where(b => b.Id >= 1 && b.Id <= 8)
    .ToList();

_Context.stocks.RemoveRange(stcs);
```

---

### ✅ 3. Final Step — Commit Deletion

```csharp
_Context.SaveChanges();
```

---

⚠️ **Important Notes**

* Ensure entity exists before deleting.
* Handle cascade deletes if relationships exist.

---

### 🧠 Quick Summary of Methods

| Method            | Description                     |
| ----------------- | ------------------------------- |
| `.Remove(entity)` | Deletes a single tracked entity |
| `.RemoveRange()`  | Deletes multiple entities       |

👉 **Summary in one line:**
Use `.Remove()` or `.RemoveRange()` to delete entities, then call `SaveChanges()`.

---

# 🎥 Video 66 — Delete Related Data

How to configure and handle deleting related entities in EF Core, using different delete behaviors.

---

### ✅ 1. Delete Behavior Configuration in Fluent API

**Cascade Delete (default for required relationships)**

```csharp
modelBuilder.Entity<Stock>()
    .HasMany(b => b.Nationality)
    .WithOne(p => p.Stock)
    .OnDelete(DeleteBehavior.Cascade);
```

✔️ Deleting a `Stock` deletes all related `Nationality`.

---

**Restrict**

```csharp
modelBuilder.Entity<Stock>()
    .HasMany(b => b.Nationality)
    .WithOne(p => p.Stock)
    .OnDelete(DeleteBehavior.Restrict);
```

❌ EF will throw an error unless related `Nationality` is deleted first.

---

**SetNull**

```csharp
modelBuilder.Entity<Stock>()
    .HasMany(b => b.Nationality)
    .WithOne(p => p.Stock)
    .OnDelete(DeleteBehavior.SetNull);
```

✔️ Deletes `Stock`, and sets `StockId` to null in `Nationality`.



### ✅ 2 Manual Deletion (Restrict Mode)

```csharp
var stock = _Context.stocks.Find(2);
var nats = _Context.nationalities.Where(b => b.StockId == 2);

_Context.RemoveRange(nats);  // delete related first
_Context.Remove(stock);      // then delete parent

_Context.SaveChanges();
```

---

⚠ **Common Mistakes:**

- Use `==`, not `=` when filtering
    
- Nullable FK needed for **SetNull**
    

---

### 🧠 Summary of Delete Behaviors

|Behavior|What happens to children|
|---|---|
|Cascade|Automatically deleted|
|Restrict|Throws error unless deleted manually|
|SetNull|FK set to null (must be nullable)|

---

✅ **Summary in one line:**  
👉 Use `.OnDelete()` to control behavior: auto-delete, restrict, or nullify related entities.



---

# 🎥 Video 67 — Transactions in EF Core

Use transactions manually to ensure multi-step consistency and rollback with savepoints.

---

### ✅ Why Use Transactions?

* All-or-nothing success
* Prevents partial saves
* Essential for multi-entity operations

---

### ✅ Code Breakdown

**Start Transaction**

```csharp
using var transaction = _Context.Database.BeginTransaction();
```

**Add + Save**

```csharp
_Context.stocks.Add(new Stock { Name="oso", Balance=222, Industry="skasd", Sector="almds", Symbol="xyz" });
_Context.SaveChanges();

transaction.CreateSavepoint("add first Stock");
```

**More Operations**

```csharp
_Context.stocks.Add(new Stock { Name = "new", Balance = 455, Industry = "vrasd", Sector = "gvtds" });
_Context.SaveChanges();
```

**Commit Transaction**

```csharp
transaction.Commit();
```

**Rollback on Error**

```csharp
catch (Exception ex)
{
    transaction.Rollback();                  // full rollback
    transaction.RollbackToSavepoint("add first Stock"); // partial rollback
    transaction.Commit();
}
```

---

### 🧠 Summary Table

| Action                        | Description              |
| ----------------------------- | ------------------------ |
| `BeginTransaction()`          | Start manual transaction |
| `SaveChanges()`               | Commit step              |
| `CreateSavepoint("name")`     | Midpoint rollback marker |
| `Rollback()`                  | Undo all                 |
| `RollbackToSavepoint("name")` | Undo to last marker      |
| `Commit()`                    | Finalize changes         |

👉 **Summary in one line:**
Use EF Core transactions with optional savepoints to ensure reliable multi-step operations, with full or partial rollback support.



---

## 🎥 Video 68 — Save Data Using Raw SQL (ExecuteSqlRaw)

Execute SQL manually using `ExecuteSqlRaw()` or `FromSqlRaw()` for SELECTs.

---

### ✅ EF Core SQL APIs

|Method|Use Case|
|---|---|
|`FromSqlRaw()`|SELECT (returns data)|
|`ExecuteSqlRaw()`|INSERT, UPDATE, DELETE|

---

### ✅ 1 Read with FromSqlRaw

```csharp
_Context.stocks.FromSqlRaw("SELECT * FROM stocks");
```

---

### ✅ 2 Write with ExecuteSqlRaw

```csharp
_Context.Database.ExecuteSqlRaw("UPDATE stocks SET Balance = 0 WHERE Id = 5");
```

---

### ✅ Use Parameters (Safe from Injection)

```csharp
_Context.Database.ExecuteSqlRaw(
    "UPDATE stocks SET Balance = {0} WHERE Id = {1}", 1000, 5
);

// Or interpolated
_Context.Database.ExecuteSqlInterpolated(
    $"UPDATE stocks SET Balance = {balance} WHERE Id = {id}"
);
```

---

⚠ **Limitations:**

- No EF tracking or validation
    
- Direct execution only
    
- Not ideal for relational inserts
    

---

### 🧠 Summary Table

|Method|Returns Data|Tracks Entity|
|---|---|---|
|`FromSqlRaw()`|✅ Yes|✅ Yes|
|`ExecuteSqlRaw()`|❌ No|❌ No|

---

✅ **Summary in one line:**  
👉 Use `ExecuteSqlRaw()` for SQL write operations, and `FromSqlRaw()` for SELECTs — both allow stored procedures and raw SQL.

---

## 🎥 Video 69 — MaxBy vs MinBy

This video explains how to get the full entity that has the maximum or minimum value using `.MaxBy()` and `.MinBy()`.

---

### ✅ 1 What is .MinBy() and .MaxBy()?

They return the entire object (not just the value) that has:

- Minimum value → `MinBy`
    
- Maximum value → `MaxBy`
    

---

### ✅ 2 Syntax Example

```csharp
var _Context = new ApplicationDbContext();

var cheapest = _Context.stocks.MinBy(b => b.Balance);
var mostExpensive = _Context.stocks.MaxBy(b => b.Balance);
```

|Method|What it does|
|---|---|
|MinBy()|Returns Stock with lowest Balance|
|MaxBy()|Returns Stock with highest Balance|

---

### 🔎 Comparison with `.Min()` and `.Max()`

|Method|Returns|Example Type|
|---|---|---|
|`Min(b => b.Balance)`|Just the value|`decimal`|
|`MinBy(...)`|Full object|`Stock`|


---

### ✅ 3 .NET Support

- Available in **.NET 6+**
    
- Part of **System.Linq**
    
- Works with **LINQ**, not EF-only
    

---

⚠ **Notes:**

- Might run **in memory** (not SQL-translated).
    
- For large data, prefer:
    

```csharp
var cheapest = _Context.stocks.OrderBy(b => b.Balance).FirstOrDefault();
var mostExpensive = _Context.stocks.OrderByDescending(b => b.Balance).FirstOrDefault();
```

---

✅ **Summary in one line:**  
👉 Use **MinBy()** and **MaxBy()** to get the full object with smallest/largest column value — useful for top/bottom records.



---

### **Video 70 — EF Core 7: Bulk Delete & Bulk Update (`ExecuteDelete`, `ExecuteUpdate`)**

EF Core 7 introduced **direct bulk operations** via the new methods:

- `ExecuteDelete()`
    
- `ExecuteUpdate()`
    

These enable **efficient batch processing** directly at the database level — no entity tracking or loading required.

---

### ✅ Bulk Delete — `ExecuteDelete()`

#### 🧪 Delete All:

```csharp
context.Employees.ExecuteDelete();
```

#### ✅ Delete with Condition:

```csharp
context.Employees
    .Where(e => e.Id > 950)
    .ExecuteDelete();
```

🔍 This executes a SQL DELETE directly:

```sql
DELETE FROM Employees WHERE Id > 950
```

No tracking, no loading into memory — just raw SQL at scale.

---

### ✅ Bulk Update — `ExecuteUpdate()`

#### ✅ Update with Condition:

```csharp
context.Employees
    .Where(e => e.Id > 900)
    .ExecuteUpdate(x =>
        x.SetProperty(e => e.FirstName, e => e.FirstName + " updated")
    );
```

#### ✅ Multiple Property Update:

```csharp
context.Employees
    .ExecuteUpdate(x =>
        x.SetProperty(e => e.IsActive, true)
    );
```

☝️ You can chain multiple `.SetProperty(...)` calls to update several fields.

---

### ⚠️ Key Features

- No need to load entities from DB.
    
- Fast and optimized for large data sets.
    
- Cannot trigger entity-level events or validations (like SaveChanges).
    

---

### 📌 Requirements

- Requires **EF Core 7 or later**
    
- Database provider must support translation (SQL Server does)
    

---

### ✅ Summary in one line:

☝️ Use `ExecuteDelete()` and `ExecuteUpdate()` in EF Core 7+ for fast, tracked-free batch operations — ideal for mass updates/deletes.

---



```