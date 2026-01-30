
---

# 🧩 Step 1 – `CheckPermissionAttribute.cs`

```csharp
[AttributeUsage(AttributeTargets.Method, AllowMultiple = false)]
public class CheckPermissionAttribute : Attribute
{
    public Permission Permission;
    public CheckPermissionAttribute(Permission permission)
    {
        this.Permission = permission;
    }
}
```

### Line by line with purpose:

- **`[AttributeUsage(AttributeTargets.Method, AllowMultiple = false)]`**
    
    - In general: Attributes are “labels” or “tags” you attach to code elements (methods, classes, properties).
        
    - Purpose: Here we say this attribute can **only be placed on methods**. That makes sense because permissions are tied to actions (like “delete product”), not entire classes in this design.
        
    - `AllowMultiple = false`: You want **one required permission per action**. Otherwise, conflict arises (“do I need both or just one?”).
        
- **`public class CheckPermissionAttribute : Attribute`**
    
    - General: In C#, we create **custom attributes** to carry metadata. They **don’t enforce behavior** by themselves. They just store extra info.
        
    - Purpose: This attribute will **mark which permission is needed** to access a method. Later, another system (the filter) reads it and enforces it.
        
- **`public Permission Permission;`**
    
    - General: This is a field holding data inside the attribute.
        
    - Purpose: It stores which specific **permission** the method requires (like `ReadProduct`).
        
- **Constructor (`CheckPermissionAttribute(Permission permission)`)**
    
    - General: Constructors in attributes make certain info **mandatory** when applying the attribute.
        
    - Purpose: Forces developers to always say **which permission** is needed when they use `[CheckPermission]`.
        
    - Example: `[CheckPermission(Permission.DeleteProduct)]`.
        

👉 **General purpose of this file:**  
It’s a **marker system**. In large apps, you don’t want authorization rules hidden inside code. You want them visible and declarative:

- “This method requires AddProduct permission.”  
    This makes the code **self-documenting** and easier to maintain.
    

---

# 🧩 Step 2 – `PermissionBasedAuthorizationFilter.cs`

```csharp
public class PermissionBasedAuthorizationFilter : IAuthorizationFilter
{
    private readonly ApplicationDbContext _dbContext;

    public PermissionBasedAuthorizationFilter(ApplicationDbContext dbContext)
    {
        _dbContext = dbContext;
    }

    public void OnAuthorization(AuthorizationFilterContext context)
    {
        var attribute = (CheckPermissionAttribute)context.ActionDescriptor
            .EndpointMetadata.FirstOrDefault(x => x is CheckPermissionAttribute);

        if (attribute != null)
        {
            var claimIdentity = context.HttpContext.User.Identity as ClaimsIdentity;
            if (claimIdentity == null || !claimIdentity.IsAuthenticated)
            {
                context.Result = new ForbidResult();
            }
            else
            {
                var userId = int.Parse(claimIdentity.FindFirst(ClaimTypes.NameIdentifier).Value);

                var hasPermissions = _dbContext.Set<UserPermissions>()
                    .Any(x => x.UserId == userId && x.PermissionId == attribute.Permission);

                if (!hasPermissions)
                {
                    context.Result = new ForbidResult();
                }
            }
        }
    }
}
```

### Line by line with purpose:

- **`public class PermissionBasedAuthorizationFilter : IAuthorizationFilter`**
    
    - General: Filters in ASP.NET Core run **before or after controller actions**.
        
    - Purpose: `IAuthorizationFilter` runs **before action execution** to check if the user is allowed.
        
    - Why: This is the **enforcement mechanism** for our marker (`CheckPermission`). Without it, the attribute is just “ink on paper.”
        
- **`private readonly ApplicationDbContext _dbContext;` + constructor injection**
    
    - General: We inject dependencies using DI (Dependency Injection).
        
    - Purpose: We need DB access to check what permissions the user actually has.
        
    - Why: Permissions are stored in DB (not hardcoded), so different users can have different rights.
        

---

### Inside `OnAuthorization`

- **Find the attribute:**
    
    ```csharp
    var attribute = (CheckPermissionAttribute)context.ActionDescriptor
        .EndpointMetadata.FirstOrDefault(x => x is CheckPermissionAttribute);
    ```
    
    - General: `context.ActionDescriptor.EndpointMetadata` holds all attributes on the current action.
        
    - Purpose: Check if this method is decorated with `[CheckPermission(...)]`. If yes → extract which permission is required.
        
    - Why: Not all actions require permissions. If no attribute, skip check.
        

---

- **Check if user is authenticated:**
    
    ```csharp
    var claimIdentity = context.HttpContext.User.Identity as ClaimsIdentity;
    if (claimIdentity == null || !claimIdentity.IsAuthenticated)
    {
        context.Result = new ForbidResult();
    }
    ```
    
    - General: `User.Identity` represents the logged-in user. If null or unauthenticated → user is anonymous.
        
    - Purpose: Don’t bother checking permissions for unauthenticated users — just block them.
        
    - Why: Authentication comes **before** authorization. No login = no permissions.
        

---

- **Extract user id from claims:**
    
    ```csharp
    var userId = int.Parse(claimIdentity.FindFirst(ClaimTypes.NameIdentifier).Value);
    ```
    
    - General: Claims are pieces of info about a user (like “id”, “email”, “role”).
        
    - Purpose: Use the claim `NameIdentifier` as the unique ID of the user.
        
    - Why: Needed to check DB for permissions.
        

---

- **Check database for permission:**
    
    ```csharp
    var hasPermissions = _dbContext.Set<UserPermissions>()
        .Any(x => x.UserId == userId && x.PermissionId == attribute.Permission);
    ```
    
    - General: Query EF Core table `UserPermissions`.
        
    - Purpose: See if this user has the required permission.
        
    - Why: This allows **fine-grained, dynamic authorization**. Permissions can be added/removed without changing code.
        

---

- **Forbid if no permission:**
    
    ```csharp
    if (!hasPermissions)
    {
        context.Result = new ForbidResult();
    }
    ```
    
    - General: `ForbidResult()` → tells ASP.NET Core to stop and return 403 Forbidden.
        
    - Purpose: Deny access if the user lacks permission.
        

👉 **General purpose of this filter:**  
It’s the **guard** standing at the door of each action.

- Attribute says: “This door requires X permission.”
    
- Filter checks: “Does this person have the key (permission)?”
    
- If not → reject request.
    

---

# 🧩 Step 3 – `ProductController.cs`

Example:

```csharp
[Authorize]
[CheckPermission(Permission.ReadProduct)]
public ActionResult<IEnumerable<Product>> GetProducts()
{
    return Ok(_context.Products.ToList());
}
```

### Explanation with purpose:

- **`[Authorize]`**
    
    - General: Built-in attribute that requires authentication.
        
    - Purpose: Ensures only logged-in users can even enter this controller.
        
    - Why: Security principle → don’t show product APIs to anonymous users.
        
- **`[CheckPermission(Permission.ReadProduct)]`**
    
    - General: Our custom attribute requiring specific permission.
        
    - Purpose: Marks this action as requiring “read product” rights.
        
    - Why: Not all users should be able to read product data. Example: Admin vs Customer.
        

👉 General purpose of controller usage:  
The controller actions declare **what permissions are needed** in a clean, declarative way. No messy if-statements inside actions.

---

# 🧩 Step 4 – Data Models

```csharp
public enum Permission
{
    ReadProduct=1,
    AddProduct,
    EditProduct,
    DeleteProduct
}
```

- General: An enum is a fixed set of constants.
    
- Purpose: Define all possible permissions in the system.
    
- Why: Centralized list avoids typos (“ReadProdcut” vs “ReadProduct”).
    

```csharp
public class UserPermissions
{
    public int UserId { get; set; }
    public Permission PermissionId { get; set; }
}
```

- General: A many-to-many relationship (Users ↔ Permissions).
    
- Purpose: Store which user has which permissions.
    
- Why: Flexible, scalable. Each user can have multiple permissions.
    

---

# 🧩 Step 5 – Program.cs (registration)

```csharp
builder.Services.AddControllers(op =>
{
    op.Filters.Add<PermissionBasedAuthorizationFilter>();
});
```

- General: Registers our custom filter globally.
    
- Purpose: Makes sure **every controller action** is checked for permission if it has `[CheckPermission]`.
    
- Why: Without this, the filter never runs.
    

---

# 🎯 Big Picture Purpose

Let’s step back:

1. **Authentication vs Authorization**
    
    - Authentication = _Who are you?_ (login, JWT, etc.)
        
    - Authorization = _What can you do?_ (permissions, roles).
        
    - This project focuses on **authorization**.
        
2. **Why Permission-Based (not just Roles)?**
    
    - Roles = broad (Admin, User).
        
    - Permissions = fine-grained (AddProduct, EditProduct, etc.).
        
    - Example: An “Editor” role might have only `AddProduct` + `EditProduct`, not `DeleteProduct`.
        
3. **Flow in this project:**
    
    - User logs in (authentication, JWT gives identity).
        
    - Request hits controller.
        
    - `[Authorize]` checks authentication.
        
    - `[CheckPermission]` marks required permission.
        
    - `PermissionBasedAuthorizationFilter` checks DB → allow or forbid.
        
4. **General software engineering purpose:**
    
    - Centralized, declarative, flexible authorization.
        
    - Easy to maintain: want a new permission? Just add to enum + DB.
        
    - Security best practice: enforce least privilege (users only get the actions they’re permitted).
        



---

# 🔐 Workflow of Permission-Based Authorization

1. **User sends a request**
    
    - The request goes to your API (example: `GET /api/product/1`).
        
    - The request includes a **JWT token** in the header (`Authorization: Bearer ...`).
        

---

2. **Authentication check – `[Authorize]`**
    
    - ASP.NET Core checks if the JWT is **valid** (signature, issuer, audience, expiration, etc.).
        
    - ✅ If valid → user is considered **authenticated**.
        
    - ❌ If invalid or missing → return **401 Unauthorized** immediately.
        

---

3. **Controller action is selected**
    
    - Example: `GetProduct(int id)` in `ProductController`.
        
    - The framework inspects the method and sees if any custom attributes (like `[CheckPermission(...)]`) are applied.
        

---

4. **`PermissionBasedAuthorizationFilter` runs**
    
    - The filter looks at the current action and checks if it has a `[CheckPermission]` attribute.
        
    - If yes → it reads **which permission is required** (e.g., `Permission.ReadProduct`).
        
    - Then:
        
        1. Extracts the **user id** from the JWT claims (`ClaimTypes.NameIdentifier`).
            
        2. Queries the **UserPermissions table** in the database to see if this user has the required permission.
            

---

5. **Authorization decision**
    
    - ✅ If the user has the permission → request continues.
        
    - ❌ If the user doesn’t have it → return **403 Forbidden**.
        

---

6. **Action executes**
    
    - If both authentication and authorization succeed, the actual controller action runs.
        
    - Example: load the product from DB and return it.
        

---

7. **Response returned**
    
    - The action result (200 OK, product data, etc.) is sent back to the client.
        

---

👉 In short:

- Step 2 = **Authentication** (“Who are you?”).
    
- Steps 3–5 = **Authorization** (“Are you allowed to do this?”).
    

---

