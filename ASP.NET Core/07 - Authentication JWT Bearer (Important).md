

# 🧠 Step 1 – JWT Theory (from zero)

Before touching your code, let’s understand JWT:

- **JWT** = JSON Web Token.
    
- It’s a compact, URL-safe way to transfer _who a user is_ and _their claims_ in a signed token.
    

👉 A JWT looks like this:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJVc2VyTmFtZSI6ImFkbWluIiwiZW1haWwiOiJ0ZXN0QGVtYWlsLmNvbSJ9.
pYBfQ9u83QnF3g0I_lk0A1M8mzgF_xvD5o7o7yEJ1uE
```

It has **3 parts**:

1. **Header** → algorithm & type (e.g., `HS256`, `JWT`).
    
2. **Payload** → user claims (e.g., username, roles, email).
    
3. **Signature** → generated using a secret key, prevents tampering.
    

⚡ Unlike Basic Auth (where you send username/password every time), JWT is:

- Given to you once (after login).
    
- You send it in every request as a **Bearer token**.
    
- The server checks if it’s valid → if yes, you’re authenticated.
    

That’s why JWT is **stateless**: the server doesn’t need to store sessions. It only validates the token.

---

# 🧠 Step 2 – Your Code, Line by Line (Authentication parts only)

---

## 🔹 AuthenticationRequest.cs

```csharp
public class AuthenticationRequest
{
    public string UserName { get; set; }
    public string Password { get; set; }
}
```

- This is a simple DTO (Data Transfer Object).
    
- When the user logs in, they’ll send a JSON body like:
    

```json
{
  "username": "admin",
  "password": "1234"
}
```

- Why needed? → So you can capture login input from the client.
    

---

## 🔹 UserControllers.cs

```csharp
public class UserControllers(JwtOptions jwtOptions) : ControllerBase
```

- **What it means:**  
    This controller is responsible for authentication.  
    It takes `JwtOptions` (from your config file) so it knows how to generate tokens (issuer, audience, signing key).
    
- **Why:**  
    This is the “login endpoint.” Users hit this to get a JWT token.
    

---

```csharp
[HttpPost]
[Route("auth")]
public ActionResult<string> AuthinticateUser(AuthenticationRequest request)
```

- **What it means:**  
    This method runs when the client POSTs to `/usercontrollers/auth`.  
    It accepts `AuthenticationRequest` (username & password).
    
- **Why:**  
    This is the step where the server checks credentials and issues a token.
    

---

```csharp
var TokenHandler = new JwtSecurityTokenHandler();
```

- This class creates and validates JWTs.
    
- Think of it as “the tool that builds tokens.”
    

---

```csharp
var tokenDescriptor = new SecurityTokenDescriptor
{
    Issuer = jwtOptions.Issuer,
    Audience = jwtOptions.Audience,
    SigningCredentials = new SigningCredentials(
        new SymmetricSecurityKey(Encoding.UTF8.GetBytes(jwtOptions.SigningKey)),
        SecurityAlgorithms.HmacSha256),
    Subject = new ClaimsIdentity(new Claim[]
    {
        new(ClaimTypes.NameIdentifier,request.UserName),
        new (ClaimTypes.Email,"osama@gmail.com")
    })
};
```

🔎 Let’s unpack this carefully:

- **Issuer** → Who created the token. (Your API server).  
    Example: `"https://localhost"`.
    
- **Audience** → Who the token is for. (The client application).  
    Example: `"https://localhost:4000/WebPortal"`.
    
- **SigningCredentials** → How we sign the token.
    
    - Uses a **secret key** (from config).
        
    - Uses algorithm **HMAC SHA256**.
        
    - Without this, anyone could forge a token.
        
- **Subject** → Claims about the user.
    
    - `NameIdentifier = request.UserName` (the user’s ID).
        
    - `Email = "osama@gmail.com"` (static for now, but usually from DB).
        

👉 These claims go inside the **JWT payload**.

---

```csharp
var securityToken = TokenHandler.CreateToken(tokenDescriptor);
var accessToken= TokenHandler.WriteToken(securityToken);
return Ok(accessToken);
```

- `CreateToken` → builds the token object.
    
- `WriteToken` → converts it into a string like `eyJhbGciOi...`
    
- Return it → the client now has a JWT token.
    

🔑 From now on, the client must send it in every request:

```
Authorization: Bearer eyJhbGciOi...
```

---

## 🔹 JwtOptions.cs

```csharp
public class JwtOptions
{
    public string Issuer { get; set; }
    public string Audience { get; set; }
    public int LifeTime { get; set; }
    public string SigningKey { get; set; }
}
```

- A simple class to hold config values (loaded from `Config.json`).
    
- Why? → So you don’t hardcode secrets (like signing key) in code.
    

---

## 🔹 Config.json

```json
"Jwt": {
  "Issuer": "https://localhost", 
  "Audience": "https://localhost:4000/WebPortal",
  "LifeTime": 30,
  "SigningKey": "KTuEPbSYpKh7VLBpmeaKqgfwW0DUCjIK"
}
```

- **Issuer** → Your API server identity.
    
- **Audience** → Who’s supposed to use the token.
    
- **LifeTime** → Expiration in minutes (not used yet in your code, but can be added).
    
- **SigningKey** → Secret used to sign JWT.
    

⚠️ In real apps, keep this secret **very safe** (not in plain config). Use Azure Key Vault, environment variables, etc.

---

## 🔹 Program.cs

```csharp
var Jwtoptions = builder.Configuration.GetSection("Jwt").Get<JwtOptions>();
builder.Services.AddSingleton(Jwtoptions);
```

- Reads `"Jwt"` section from `Config.json`.
    
- Registers it in DI container so controllers can use it.
    

---

```csharp
builder.Services.AddAuthentication()
    .AddJwtBearer(JwtBearerDefaults.AuthenticationScheme, option =>
{
    option.SaveToken = true;
    option.TokenValidationParameters = new TokenValidationParameters
    {
        ValidateIssuer = true,
        ValidIssuer = Jwtoptions.Issuer,
        ValidateAudience = true,
        ValidAudience = Jwtoptions.Audience,
        ValidateIssuerSigningKey = true,
        IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(Jwtoptions.SigningKey))
    };
});
```

This is **the most important part**. Let’s unpack:

1. `AddAuthentication().AddJwtBearer(...)` → tells ASP.NET Core:  
    “We’re using JWT Bearer tokens as our authentication method.”
    
2. `option.SaveToken = true;` → Saves the token in `HttpContext` (optional, useful for debugging).
    
3. `TokenValidationParameters` → Rules for validating tokens:
    
    - `ValidateIssuer = true` → Check token’s Issuer matches our config.
        
    - `ValidateAudience = true` → Check token’s Audience matches our config.
        
    - `ValidateIssuerSigningKey = true` → Ensure token signature is valid.
        
    - `IssuerSigningKey` → Our secret key to verify signature.
        

👉 This ensures that only tokens generated by **your server** (with the correct key) are valid.

---

```csharp
app.UseAuthorization();
```

⚠️ **Important** → You’re missing `app.UseAuthentication();` before this.

- `UseAuthentication()` → checks the token, sets `HttpContext.User`.
    
- `UseAuthorization()` → checks `[Authorize]` attributes and uses the user claims.
    

Without `UseAuthentication`, `[Authorize]` won’t work properly.

The correct order is:

```csharp
app.UseAuthentication();
app.UseAuthorization();
```

---

# 🧠 Step 3 – Request Flow with JWT

1. **Login** → Client sends `username/password` to `/usercontrollers/auth`.
    
2. **Server validates** (right now you don’t check password, but you could).
    
3. **Server issues JWT** → signed with secret key.
    
4. **Client stores token** → in localStorage (web), SecureStorage (mobile).
    
5. **Client sends requests** with:
    
    ```
    Authorization: Bearer eyJhbGciOi...
    ```
    
6. **ASP.NET Core middleware** checks the token (issuer, audience, signature, expiration).
    
7. If valid → sets `HttpContext.User`.
    
8. `[Authorize]` → allows access to protected controllers.
    
9. If invalid → 401 Unauthorized.
    

---

# 🧠 Step 4 – JWT vs Basic (Big Picture)

- **Basic Authentication** → sends username & password every time.  
    ❌ Insecure (credentials exposed).
    
- **JWT Authentication** → send token instead of credentials.  
    ✅ More secure (password not sent again).  
    ✅ Stateless (server doesn’t store sessions).  
    ✅ Can include extra claims (roles, email, etc.).
    

---

✅ So now you know:

- What JWT is (theory).
    
- What each line in your code does.
    
- How ASP.NET Core uses it to authenticate.
    
- Why it’s better than Basic.
    

---
# **Create JWT Expiration Theory**
# 🧠 Step 1 – JWT Expiration Theory

- A JWT should never last forever.
    
- Why? Because if an attacker steals it, they can use it as long as it’s valid.
    
- To solve this:
    
    - We add an **`exp` claim** → expiration date/time.
        
    - When the token is expired, ASP.NET Core rejects it automatically.
        

Typical lifetime:

- **Short-lived Access Token** (e.g., 15 minutes – 1 hour).
    
- **Refresh Token** (stored separately, lasts days/weeks, used to get new access tokens).
    

👉 Your config already has `"LifeTime": 30` (minutes). Perfect — we’ll use it.

---

# 🧠 Step 2 – Modify Token Creation (`UserControllers`)

Right now, your token descriptor looks like this:

```csharp
var tokenDescriptor = new SecurityTokenDescriptor
{
    Issuer = jwtOptions.Issuer,
    Audience = jwtOptions.Audience,
    SigningCredentials = new SigningCredentials(
        new SymmetricSecurityKey(Encoding.UTF8.GetBytes(jwtOptions.SigningKey)),
        SecurityAlgorithms.HmacSha256),
    Subject = new ClaimsIdentity(new Claim[]
    {
        new(ClaimTypes.NameIdentifier, request.UserName),
        new(ClaimTypes.Email, "osama@gmail.com")
    })
};
```

👉 Add **Expires**:

```csharp
var tokenDescriptor = new SecurityTokenDescriptor
{
    Issuer = jwtOptions.Issuer,
    Audience = jwtOptions.Audience,
    Expires = DateTime.UtcNow.AddMinutes(jwtOptions.LifeTime), // 👈 NEW
    SigningCredentials = new SigningCredentials(
        new SymmetricSecurityKey(Encoding.UTF8.GetBytes(jwtOptions.SigningKey)),
        SecurityAlgorithms.HmacSha256),
    Subject = new ClaimsIdentity(new Claim[]
    {
        new(ClaimTypes.NameIdentifier, request.UserName),
        new(ClaimTypes.Email, "osama@gmail.com")
    })
};
```

- **`DateTime.UtcNow`** → current time in UTC (important, because tokens must use UTC).
    
- **`AddMinutes(jwtOptions.LifeTime)`** → adds the lifetime from your config (e.g., 30 minutes).
    

Now your token will expire in 30 minutes.

---

# 🧠 Step 3 – Modify Validation (`Program.cs`)

Your current validation:

```csharp
option.TokenValidationParameters = new TokenValidationParameters
{
    ValidateIssuer = true,
    ValidIssuer = Jwtoptions.Issuer,
    ValidateAudience = true,
    ValidAudience = Jwtoptions.Audience,
    ValidateIssuerSigningKey = true,
    IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(Jwtoptions.SigningKey))
};
```

👉 Add **ValidateLifetime**:

```csharp
option.TokenValidationParameters = new TokenValidationParameters
{
    ValidateIssuer = true,
    ValidIssuer = Jwtoptions.Issuer,
    ValidateAudience = true,
    ValidAudience = Jwtoptions.Audience,
    ValidateIssuerSigningKey = true,
    IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(Jwtoptions.SigningKey)),
    ValidateLifetime = true, // 👈 NEW
    ClockSkew = TimeSpan.Zero // 👈 Optional: no grace period
};
```

- **`ValidateLifetime = true`** → ASP.NET Core will check the `exp` claim.
    
- **`ClockSkew`** → by default, ASP.NET Core allows 5 minutes of "leeway" (for server clock differences). Setting `TimeSpan.Zero` removes that grace period (stricter).
    

Now, expired tokens will be rejected automatically with **401 Unauthorized**.

---

# 🧠 Step 4 – Request Flow with Expiration

1. User logs in → gets token valid for 30 minutes.
    
2. User sends requests with:
    
    ```
    Authorization: Bearer eyJhbGciOi...
    ```
    
3. ASP.NET Core checks:
    
    - Signature ✅
        
    - Issuer ✅
        
    - Audience ✅
        
    - Expiration ✅ (must be in the future).
        
4. If expired → 401 Unauthorized.
    
5. Client must **re-authenticate** or use a **refresh token** to get a new access token.
    

---

# 🧠 Step 5 – Example Token Payload with Expiration

A JWT with expiration (`exp`) will have this payload:

```json
{
  "unique_name": "admin",
  "email": "osama@gmail.com",
  "nbf": 1691177300,
  "exp": 1691179100,
  "iss": "https://localhost",
  "aud": "https://localhost:4000/WebPortal"
}
```

- **nbf** = "not before" (when token becomes valid).
    
- **exp** = expiration time (in UNIX timestamp).
    
- **iss** = issuer.
    
- **aud** = audience.
    

ASP.NET Core will automatically reject if `exp` is in the past.

---

# ✅ Final Updated Code

### In **UserControllers.cs**

```csharp
var tokenDescriptor = new SecurityTokenDescriptor
{
    Issuer = jwtOptions.Issuer,
    Audience = jwtOptions.Audience,
    Expires = DateTime.UtcNow.AddMinutes(jwtOptions.LifeTime),
    SigningCredentials = new SigningCredentials(
        new SymmetricSecurityKey(Encoding.UTF8.GetBytes(jwtOptions.SigningKey)),
        SecurityAlgorithms.HmacSha256),
    Subject = new ClaimsIdentity(new Claim[]
    {
        new(ClaimTypes.NameIdentifier, request.UserName),
        new(ClaimTypes.Email, "osama@gmail.com")
    })
};
```

### In **Program.cs**

```csharp
option.TokenValidationParameters = new TokenValidationParameters
{
    ValidateIssuer = true,
    ValidIssuer = Jwtoptions.Issuer,
    ValidateAudience = true,
    ValidAudience = Jwtoptions.Audience,
    ValidateIssuerSigningKey = true,
    IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(Jwtoptions.SigningKey)),
    ValidateLifetime = true,
    ClockSkew = TimeSpan.Zero
};
```

---

# 🧠 Step 6 – Real-World Note

- **Access tokens (short-lived, 15–60 min)** → safer, even if stolen, attacker can’t use forever.
    
- **Refresh tokens (long-lived, stored securely)** → let users get new access tokens without logging in again.
    

👉 Your code right now only has **access tokens**. Later, you can add **refresh tokens** for full real-world JWT setup.

---



```