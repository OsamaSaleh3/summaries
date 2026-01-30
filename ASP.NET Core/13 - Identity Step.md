
---

# 1) Project setup

1. Create ASP.NET Core Web API project.
    
2. Add NuGet packages (if not already included):
    
    - `Microsoft.AspNetCore.Identity.EntityFrameworkCore`
        
    - `Microsoft.EntityFrameworkCore.SqlServer` (or your DB provider)
        
    - `Microsoft.EntityFrameworkCore.Tools`
        
    - `AutoMapper.Extensions.Microsoft.DependencyInjection`
        
    - `Microsoft.AspNetCore.Authentication.JwtBearer`
        
    - `Swashbuckle.AspNetCore` (Swagger)
        

---

# 2) Domain & Identity models

1. Create your custom user:
    
    ```csharp
    public class User : IdentityUser
    {
        public string FirstName { get; set; }
        public string LastName  { get; set; }
    }
    ```
    
2. (Optional) Other entities (e.g., `Company`, `Employee`) with relationships.
    

---

# 3) DbContext

1. Inherit from `IdentityDbContext<User>`:
    
    ```csharp
    public class ApplicationDbContext : IdentityDbContext<User>
    {
        public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options) : base(options) {}
    
        protected override void OnModelCreating(ModelBuilder builder)
        {
            base.OnModelCreating(builder);
            // apply configurations (e.g., RoleConfiguration)
        }
    
        public DbSet<Employee> employees { get; set; }
        public DbSet<Company>  companies { get; set; }
    }
    ```
    

---

# 4) Seed roles (optional but common)

1. Create `RoleConfiguration : IEntityTypeConfiguration<IdentityRole>` with `builder.HasData(...)`.
    
2. Apply in `OnModelCreating`: `builder.ApplyConfiguration(new RoleConfiguration());`
    

---

# 5) appsettings.json

1. Add connection string and JWT settings:
    
    ```json
    "ConnectionStrings": { "DefaultConnection": "..." },
    "JWTSettings": {
      "securityKey": "YOUR_LONG_RANDOM_SECRET",
      "validIssuer": "YourApi",
      "validAudience": "YourApiClients",
      "expiryInMinutes": 5
    }
    ```
    


---

# 6) Program.cs — register services (continued)

4. **JWT Authentication**
    

```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;
using System.Text;

var jwt = builder.Configuration.GetSection("JWTSettings");

builder.Services
  .AddAuthentication(options =>
  {
      options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
      options.DefaultChallengeScheme    = JwtBearerDefaults.AuthenticationScheme;
  })
  .AddJwtBearer(options =>
  {
      options.TokenValidationParameters = new TokenValidationParameters
      {
          ValidateIssuer           = true,
          ValidateAudience         = true,
          ValidateLifetime         = true,
          ValidateIssuerSigningKey = true,
          ValidIssuer   = jwt["validIssuer"],
          ValidAudience = jwt["validAudience"],
          IssuerSigningKey = new SymmetricSecurityKey(
              Encoding.UTF8.GetBytes(jwt["securityKey"]!))
          // optional: remove default 5m skew
          // ClockSkew = TimeSpan.Zero
      };
  });
```

5. **JWT factory / helpers**
    

```csharp
builder.Services.AddSingleton<JWTHandler>(); // your token creator
```

6. **Swagger (optional)**
    

```csharp
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();
```

7. **(Optional) CORS for clients**
    

```csharp
builder.Services.AddCors(opt =>
{
    opt.AddPolicy("AllowFrontend", p =>
        p.AllowAnyHeader().AllowAnyMethod().WithOrigins("https://your-frontend"));
});
```

---

# 7) Program.cs — middleware order

```csharp
var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

app.UseCors("AllowFrontend");      // if you added it

app.UseAuthentication();           // 1) builds HttpContext.User from JWT
app.UseAuthorization();            // 2) enforces [Authorize]/policies

app.MapControllers();

app.Run();
```

> Order matters: **Authentication before Authorization**.

---

# 8) DTOs

- **Registration** (`UserForRegistrationDto`): FirstName, LastName, Email [Required], Password [Required], ConfirmPassword [Compare].
    
- **Login** (`UserForAuthenticationDto`): Email [Required], Password [Required].
    
- **Responses**: `RegistrationResponseDto`, `AuthResponseDto` (token, success flag, errors).
    

---

# 9) AutoMapper Profile

- Map registration DTO → `User`.
    
- Important rule you used: **`UserName = Email`** so login by username uses the email.
    

```csharp
CreateMap<UserForRegistrationDto, User>()
    .ForMember(u => u.UserName, opt => opt.MapFrom(x => x.Email));
```

---

# 10) JWT handler (token creation)

- Read `JWTSettings` (issuer, audience, key, expiry).
    
- Create **signing credentials** (HMAC‑SHA256 + secret).
    
- Build **claims** (you added `ClaimTypes.Name` with `UserName`).
    
- Create `JwtSecurityToken` with issuer, audience, claims, expiry, signing creds.
    
- Return **serialized token string**.
    

> Claims are the data inside the token (e.g., name, email, roles). They allow stateless auth: the API can trust what’s inside after verifying the signature.

---

# 11) Controllers (Auth)

- **Register**: map DTO → `User`, call `UserManager.CreateAsync(user, password)`.
    
    - Identity enforces your password rules + unique email.
        
    - On errors, return 400 with the `IdentityResult.Errors`.
        
- **Authenticate**: find by username (email), check password, issue JWT via `JWTHandler`.
    

> Client uses the token on subsequent calls: `Authorization: Bearer <jwt>`.

---

# 12) Protecting endpoints

- Add `[Authorize]` to any controller/action that needs an authenticated user.
    

```csharp
[Authorize]
[HttpGet]
public IActionResult SecureThing() => Ok("Only with a valid JWT");
```

> Without a valid token, the middleware returns **401 Unauthorized**.

---

# 13) Role seeding (optional but common)

- `RoleConfiguration : IEntityTypeConfiguration<IdentityRole>` with `HasData(...)` (e.g., Admin, Manager).
    
- Apply in `OnModelCreating`: `builder.ApplyConfiguration(new RoleConfiguration());`
    
- After migrating, assign roles to users with `UserManager.AddToRoleAsync(user, "Admin")`.
    
- Protect with roles:
    

```csharp
[Authorize(Roles = "Admin")]
```

---

# 14) Database migrations

1. **Add migration**  
    `dotnet ef migrations add Init`
    
2. **Update database**  
    `dotnet ef database update`
    

> This creates Identity tables, your app tables, and inserts seeded roles (if you seeded).

---

# 15) Test flow (happy path)

1. **Register** user  
    `POST /api/AuthenticationController/register` with JSON body.
    
2. **Authenticate**  
    `POST /api/AuthenticationController/authenticate` → receive `token`.
    
3. **Call a protected endpoint**  
    `GET /api/Test` with header `Authorization: Bearer <token>` → 200 OK.
    

---

# 16) Useful enhancements (next steps)

- **Add more claims** to the token (e.g., `ClaimTypes.Email`, `ClaimTypes.NameIdentifier` with `user.Id`, roles).
    
- **Authorization policies**: `RequireRole`, `RequireClaim`, custom requirements.
    
- **Email confirmation** before login (use `AddDefaultTokenProviders()` + confirmation flow).
    
- **Refresh tokens** (issue short‑lived access token + long‑lived refresh token to renew).
    
- **Password reset** (generate reset token, reset endpoint).
    
- **Lockout** options (protect against brute force).
    
- **Logging & audit** (who logged in, when).
    

---

# 17) Production checklist

- **Secrets**: don’t store real keys in `appsettings.json`. Use **User Secrets**, environment variables, or a secrets manager.
    
- **Key quality**: long, random `securityKey`. Consider **key rotation**.
    
- **HTTPS** everywhere.
    
- **Token lifetime**: balance UX vs security (you used 5 minutes). Set `ClockSkew` if you want exact expiry.
    
- **CORS**: restrict origins to trusted frontends.
    
- **Swagger**: hide in production or secure it.
    
- **Error messages**: be careful not to leak info (e.g., generic login errors).
    

---

