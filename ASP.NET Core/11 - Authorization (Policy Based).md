

---

# 🔐 Policy-Based Authorization in ASP.NET Core (Deep Dive)

We’ll go step by step through the authorization-related code and explain:

- **What it does in code**
    
- **The general purpose** (why it exists in security/software)
    
- **When you’d use it**
    

---

## 🧩 Step 1 – Program.cs (Policy Registration)

```csharp
builder.Services.AddAuthorization(option =>
{
    option.AddPolicy("AgeGreaterThan25", builder => builder.AddRequirements(new AgeGreaterThan25Requirement()));

    option.AddPolicy("EmployeesOnly", builder =>
    {
        builder.RequireClaim("UserType","Employee");
    });
});
builder.Services.AddSingleton<IAuthorizationHandler, AgeAuthorizationHandler>();
```

---

### Explanation

- **`builder.Services.AddAuthorization(...)`**
    
    - Registers the **authorization system**.
        
    - This is where you define **policies** (rules) that your actions can require.
        

---

- **`option.AddPolicy("AgeGreaterThan25", builder => builder.AddRequirements(new AgeGreaterThan25Requirement()));`**
    
    - Defines a **policy named `AgeGreaterThan25`**.
        
    - The policy requires a **custom requirement** called `AgeGreaterThan25Requirement`.
        
    - This means: “To pass this policy, a handler must succeed at validating that requirement.”
        

---

- **`option.AddPolicy("EmployeesOnly", builder => builder.RequireClaim("UserType","Employee"));`**
    
    - Defines another policy named `"EmployeesOnly"`.
        
    - This policy requires that the user has a **claim** with key `"UserType"` and value `"Employee"`.
        
    - Example: User token contains `UserType=Employee` → allowed.
        

---

- **`builder.Services.AddSingleton<IAuthorizationHandler, AgeAuthorizationHandler>();`**
    
    - Registers your **custom handler** (where you write the logic for the requirement).
        
    - ASP.NET Core will call this handler whenever a policy references `AgeGreaterThan25Requirement`.
        

---

### 🔄 Alternative using `RequireAssertion`

Instead of creating a **Requirement class** + **Handler class**, you can write the rule **inline** with `RequireAssertion`:

```csharp
option.AddPolicy("AgeGreaterThan25", builder =>
{
    builder.RequireAssertion(context =>
    {
        var dob = DateTime.Parse(context.User.FindFirstValue("DateOfBirth"));
        return (DateTime.Today.Year - dob.Year > 25);

        // ✅ Use RequireAssertion for simple one-liner checks
        // ❌ For complex logic (multiple checks, DB queries, reusable logic), use Requirement + Handler
    });
});
```

👉 Use this when your rule is **short and simple**.  
👉 Use `Requirement + Handler` when your rule is **more complex or reused across the app**.

---

## 🧩 Step 2 – ProductController.cs

```csharp
[HttpGet]
[Route("")]
[Authorize(Policy = "AgeGreaterThan25")] 
public ActionResult<IEnumerable<Product>> GetProducts()
{
    return Ok(_context.Products.ToList());
}
```

---

### Explanation

- **`[Authorize(Policy = "AgeGreaterThan25")]`**
    
    - Tells ASP.NET Core: “This action requires the policy named `AgeGreaterThan25`.”
        
    - That means the system will:
        
        - Look at the policy definition in Program.cs.
            
        - Run the requirement/handler or assertion logic.
            

👉 General purpose:  
Policies let you enforce **real-world business rules** (like age, employment status, subscription type, etc.) instead of just roles.

---

## 🧩 Step 3 – UserControllers.cs (JWT Claims Setup)

```csharp
new ClaimsIdentity(new Claim[]
{
    new(ClaimTypes.NameIdentifier,User.Id.ToString()),
    new(ClaimTypes.Name,User.Name),
    new (ClaimTypes.Email,"osama@gmail.com"),
    new (ClaimTypes.Role,"Admin"),   // for role-based
    new ("UserType","Employee"),     // for EmployeesOnly policy
    new ("DateOfBirth","2005-01-01") // for AgeGreaterThan25 policy
})
```

---

### Explanation

- **`UserType=Employee` claim** → Used in `EmployeesOnly` policy.
    
- **`DateOfBirth=2005-01-01` claim** → Used in `AgeGreaterThan25` policy.
    

👉 General purpose:  
Policies depend on **claims**. Claims are like “facts about the user” (name, role, email, DOB, department).  
Instead of hardcoding rules in code, you carry user info in the JWT → then policies evaluate it.

---

## 🧩 Step 4 – AgeGreaterThan25Requirement.cs

```csharp
public class AgeGreaterThan25Requirement : IAuthorizationRequirement
{
    // any data the policy needs we write it here
}
```

---

### Explanation

- **`IAuthorizationRequirement`** = marker interface.
    
- A **Requirement** is just a description of a rule that needs to be evaluated.
    
- It doesn’t contain logic, just “this rule exists.”
    

👉 Think of it as: _“We need a rule that checks if age > 25.”_  
The actual logic comes from the **Handler**.

---

## 🧩 Step 5 – AgeAuthorizationHandler.cs

```csharp
public class AgeAuthorizationHandler : AuthorizationHandler<AgeGreaterThan25Requirement>
{
    protected override Task HandleRequirementAsync(AuthorizationHandlerContext context, AgeGreaterThan25Requirement requirement)
    {
        var dob = DateTime.Parse(context.User.FindFirstValue("DateOfBirth"));
        if (DateTime.Today.Year - dob.Year > 25)
        {
            context.Succeed(requirement);
        }
        return Task.CompletedTask;
        // don't use Fail (InvokeHandlersAfterFailure explains it)
    }
}
```

---

### Explanation

- **AuthorizationHandler** = where the **rule logic** lives.
    
- Reads the `"DateOfBirth"` claim.
    
- Checks if user’s age > 25.
    
- If yes → calls `context.Succeed(requirement)` → policy succeeds.
    
- If not → does nothing → policy fails.
    

👉 General purpose:  
Handlers allow you to implement **complex, reusable business rules**.

---

## 🎯 Big Picture – Policy-Based Authorization

### 🔹 Workflow

1. **User logs in** → gets JWT with claims (`UserType=Employee`, `DateOfBirth=2005-01-01`).
    
2. **User requests API** with token.
    
3. **ASP.NET validates token** (authentication).
    
4. **Action has `[Authorize(Policy = "AgeGreaterThan25")]`**.
    
5. **ASP.NET looks up policy** in Program.cs.
    
6. **Policy logic executes**:
    
    - Either `RequireAssertion` inline code.
        
    - Or Requirement + Handler code.
        
7. **Decision:**
    
    - If condition passes → allow (200 OK).
        
    - If not → block (403 Forbidden).
        

---

### 🔹 Why Policy-Based?

- **Role-Based:** Easy, but too broad (“Admin can do everything”).
    
- **Permission-Based:** More fine-grained (“User can AddProduct”).
    
- **Policy-Based:** Most flexible (“User must be Employee AND older than 25 AND from HR dept”).
    

👉 Policy-based = **best for business rules** that go beyond simple roles/permissions.

---

✅ Now your explanation contains **both approaches**:

- Full Requirement + Handler structure (best for complex/reusable rules).
    
- Inline `RequireAssertion` (best for quick, simple rules).
    

---
