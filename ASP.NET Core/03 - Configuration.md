

# 📌 ASP.NET Core Configuration — Full Example with Explanations

```csharp
using AspNetCoreFinal.Data;
using AspNetCoreFinal.Filter;
using AspNetCoreFinal.Middleware;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

// 🔹 Custom configuration file is added
builder.Configuration.AddJsonFile("Config.json");
```

### Explanation:

* ASP.NET Core automatically loads:

  * **appsettings.json**
  * **appsettings.{Environment}.json** (like Development or Production)
* You can also **add your own JSON file** (here: `Config.json`).
* Since this is added **after** appsettings, it **overrides values from appsettings.json**.

---

```csharp
// Add services to the container.
builder.Services.AddControllers(options => options.Filters.Add<LogActivityFilter>());

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// 🔹 Database connection string from configuration
builder.Services.AddDbContext<ApplicationDbContext>(cfg =>
    cfg.UseSqlServer(builder.Configuration["ConnectionStrings:DefaultConnection"]));
```

### Explanation:

* `builder.Configuration["ConnectionStrings:DefaultConnection"]` reads from configuration.
* The value may come from:

  1. **appsettings.json**
  2. **appsettings.Development.json** (if Development)
  3. **Environment variables** (if set)
  4. **Config.json** (custom file, overrides others if added last)
  5. **User Secrets** (highest priority in Development)

---

```csharp
var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}
```

### Explanation:

* `app.Environment.IsDevelopment()` checks the current **environment**.
* The environment is set by **environment variables** (e.g. `DOTNET_ENVIRONMENT=Development`).
* ASP.NET Core uses this to decide which config file (`appsettings.Development.json`, etc.) to load.

---

```csharp
app.UseMiddleware<ProfilingMiddleware>();
app.UseMiddleware<RateLimitingMiddleware>();

app.UseHttpsRedirection();
app.UseAuthorization();

app.MapControllers();
app.Run();
```

*(No direct configuration here — just normal middleware setup)*

---

## 📂 appsettings.json

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "DefaultConnection": "Data Source=(localdb)\\ProjectModels;Initial Catalog=master;Integrated Security=True;"
  }
}
```

### Explanation:

* Base configuration.
* Contains **logging setup**, **allowed hosts**, and **connection strings**.

---

## 📂 appsettings.Development.json

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  }
}
```

### Explanation:

* **Overrides appsettings.json** when environment = Development.
* Used for environment-specific changes (e.g., detailed logging in Development).

---

## 📂 Config.json (Custom Configuration)

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "TestKey": "TestValue"
}
```

### Explanation:

* A **custom JSON file** you added manually.
* Since you added it last (`AddJsonFile("Config.json")`), it overrides both `appsettings.json` and `appsettings.Development.json`.

---

## 📂 Example Controller Using Configuration

```csharp
using Microsoft.AspNetCore.Mvc;

namespace AspNetCoreFinal.Controllers
{
    [ApiController]
    [Route("[controller]")]
    public class ConfigController : ControllerBase
    {
        private readonly IConfiguration _configuration;
        public ConfigController(IConfiguration configuration)
        {
            _configuration = configuration;
        }

        [HttpGet]
        [Route("")]
        public ActionResult GetConfig()
        {
            var config = new
            {
                AllowedHosts = _configuration["AllowedHosts"],
                DefaultConnection = _configuration["ConnectionStrings:DefaultConnection"],
                DefaultLogLevel = _configuration["Logging:LogLevel:Default"],
                TestKey = _configuration["TestKey"]
            };
            return Ok(config);
        }
    }
}
```

### Explanation:

* Injects `IConfiguration` to read settings.
* Access with **colon `:` notation** for nested values:

  * `"ConnectionStrings:DefaultConnection"`
  * `"Logging:LogLevel:Default"`
* Returns combined values from all configuration sources.

---

# 🔑 General Configuration Priority Order

1. **User Secrets** (in Development)
2. **Custom Config Files** (like `Config.json`)
3. **Environment Variables**
4. **appsettings.{Environment}.json**
5. **appsettings.json**

---

📊 **Visual Diagram (Flow of Configuration Sources):**

```
appsettings.json
        ↓
appsettings.{Environment}.json
        ↓
Environment Variables
        ↓
Custom Config.json
        ↓
User Secrets (highest, dev only)
```

---


## 📂 Code: `ConfigController.cs`

```csharp
using Microsoft.AspNetCore.Mvc;

namespace AspNetCoreFinal.Controllers
{
    [ApiController]
    [Route("[controller]")]
    public class ConfigController : ControllerBase
    {
        private readonly IConfiguration _configuration;
        public ConfigController(IConfiguration configuration)
        {
            _configuration = configuration;
        }

        [HttpGet]
        [Route("")]
        public ActionResult GetConfig()
        {
            var config = new
            {
                AllowedHosts = _configuration["AllowedHosts"],
                DefaultConnection = _configuration["ConnectionStrings:DefaultConnection"],
                DefaultLogLevel = _configuration["Logging:LogLevel:Default"],
                TestKey = _configuration["TestKey"]
            };
            return Ok(config);
        }
    }
}
```

---

## 🔍 Detailed Explanation

### 1. `using Microsoft.AspNetCore.Mvc;`

- Brings in ASP.NET Core MVC attributes and base classes.
    
- Needed for `[ApiController]`, `[Route]`, `ControllerBase`, and `ActionResult`.
    

---

### 2. `namespace AspNetCoreFinal.Controllers`

- Defines a **namespace** to logically group your controllers.
    
- ASP.NET Core looks inside this namespace to discover API controllers.
    

---

### 3. `[ApiController]`

- Marks this class as an **API controller**.
    
- Benefits:
    
    - Automatic model binding & validation.
        
    - Provides helpful responses for invalid input (e.g., 400 Bad Request if model is invalid).
        
    - Makes routing and parameter binding easier.
        

---

### 4. `[Route("[controller]")]`

- Defines the **base route** for this controller.
    
- `[controller]` is a placeholder → it will be replaced with the controller class name **without "Controller"**.
    
    - Example: `ConfigController` → `"/config"`.
        

👉 So the base route for this controller is:

```
/config
```

---

### 5. `public class ConfigController : ControllerBase`

- This is your **controller class**.
    
- Inherits from `ControllerBase` (lightweight, without Razor views).
    
- Perfect for APIs that return JSON responses.
    

---

### 6. `private readonly IConfiguration _configuration;`

- Defines a private field to hold the configuration object.
    
- `IConfiguration` is the **interface for all configuration providers** (JSON, env variables, secrets, etc.).
    

---

### 7. `public ConfigController(IConfiguration configuration)`

- This is the **constructor**.
    
- ASP.NET Core’s **Dependency Injection (DI)** automatically injects `IConfiguration`.
    
- Saves it into `_configuration` for later use in API actions.
    

---

### 8. `[HttpGet]` & `[Route("")]`

- Marks the action as a **GET request**.
    
- `[Route("")]` means it maps to the **root of the controller’s route**.
    

👉 Full endpoint becomes:

```
GET /config
```

---

### 9. `public ActionResult GetConfig()`

- Defines the action method that handles GET requests.
    
- `ActionResult` allows returning different types of HTTP responses (`200 OK`, `400 BadRequest`, etc.).
    

---

### 10. Inside `GetConfig()`:

```csharp
var config = new
{
    AllowedHosts = _configuration["AllowedHosts"],
    DefaultConnection = _configuration["ConnectionStrings:DefaultConnection"],
    DefaultLogLevel = _configuration["Logging:LogLevel:Default"],
    TestKey = _configuration["TestKey"]
};
```

- Creates an **anonymous object** (`config`) with values pulled from configuration.
    
- Keys are retrieved with `"Section:SubSection"` notation:
    
    - `_configuration["AllowedHosts"]` → reads from root level of JSON.
        
    - `_configuration["ConnectionStrings:DefaultConnection"]` → reads from nested section `"ConnectionStrings"`.
        
    - `_configuration["Logging:LogLevel:Default"]` → goes two levels deep inside `"Logging" → "LogLevel"`.
        
    - `_configuration["TestKey"]` → custom value from `Config.json`.
        

---

### 11. `return Ok(config);`

- Returns a **200 OK HTTP response** with the `config` object as JSON.
    

Example JSON response:

```json
{
  "allowedHosts": "*",
  "defaultConnection": "Data Source=(localdb)\\ProjectModels;Initial Catalog=master;Integrated Security=True;",
  "defaultLogLevel": "Information",
  "testKey": "TestValue"
}
```

---

## 🚀 Final Summary

- `ConfigController` is a simple API endpoint that exposes **current configuration values**.
    
- It demonstrates:
    
    - **Dependency Injection** of `IConfiguration`.
        
    - **Hierarchical key access** (`:` for nested values).
        
    - **Combining multiple configuration sources** (appsettings, env, custom JSON, secrets).
        
- Useful for **debugging configuration** and ensuring your app is reading values correctly.
    

---


# 🔹 Summary of New Configuration Concepts

---

## 1. **Creating a Strongly Typed Configuration Class**

You created a POCO class (`AttachmentOptions`) that maps configuration values from the `appsettings.json`.

```csharp
namespace AspNetCoreFinal
{
    public class AttachmentOptions
    {
        // if you add a constructor with parameters,
        // you must still have a parameterless constructor 
        // to allow mapping
        public AttachmentOptions() { }

        public string AllowedExtension { get; set; }
        public int MaxSizeInMegaBytes { get; set; }
        public bool EnableCompresion { get; set; }
    }
}
```

**Explanation:**

- This class is used to map the `Attachments` section from **appsettings.json**.
    
- Having a parameter less constructor ensures configuration binding can work automatically.
    
- Each property must match the keys under `Attachments`.
    

---

## 2. **Configuration Section in appsettings.json**

```json
"Attachments": {
  "AllowedExtension": "jpg;jpeg;bmb;png",
  "MaxSizeInMegaBytes": 1,
  "EnableCompresion": true
}
```

**Explanation:**

- These values are mapped into the `AttachmentOptions` class.
    
- This allows you to work with configuration in a **typed and safe way**, instead of using string keys everywhere.
    

---

## 3. **Registering Options in the DI Container**

You showed **3 ways** to register configuration values.

### Way 1 – Directly bind and register as Singleton

```csharp
var AttachmentOptions = builder.Configuration
    .GetSection("Attachments")
    .Get<AttachmentOptions>();

builder.Services.AddSingleton(AttachmentOptions);
```

✔ Loads config and registers the object directly.  
❌ Fixed at startup, doesn’t support reload-on-change.

---

### Way 2 – Bind manually then register

```csharp
var AttachmentOptions2 = new AttachmentOptions();
builder.Configuration.GetSection("Attachments").Bind(AttachmentOptions2);
builder.Services.AddSingleton(AttachmentOptions2);
```

✔ Same as Way 1, but you explicitly **create the object and bind properties**.  
✔ Gives you more control if you need to manipulate values before registration.

---

### Way 3 – (Best Practice) Using `Configure<T>`

```csharp
builder.Services.Configure<AttachmentOptions>(
    builder.Configuration.GetSection("Attachments"));
```

✔ Registers the section properly.  
✔ Supports **reload-on-change** automatically.  
✔ Allows usage of `IOptions`, `IOptionsSnapshot`, and `IOptionsMonitor`.  
👉 This is the recommended and scalable way.

---

## 4. **Consuming Configuration in a Controller**

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Options;

namespace AspNetCoreFinal.Controllers
{
    [ApiController]
    [Route("[controller]")]
    public class ConfigController : ControllerBase
    {
        private readonly IConfiguration _configuration;
        private readonly IOptions<AttachmentOptions> _attachmentOptions;

        // IOptions is registered as Singleton by default
        // IOptionsSnapshot -> new instance per request
        // IOptionsMonitor -> allows live reload when config changes
        public ConfigController(
            IConfiguration configuration,
            IOptions<AttachmentOptions> attachmentOptions)
        {
            _configuration = configuration;
            _attachmentOptions = attachmentOptions;
        }

        [HttpGet]
        [Route("")]
        public ActionResult GetConfig()
        {
            var config = new
            {
                AttachmentOptions = _attachmentOptions.Value,
                mxSize = _attachmentOptions.Value.MaxSizeInMegaBytes
            };
            return Ok(config);
        }
    }
}
```

**Explanation:**

- `IOptions<T>` → Reads configuration once at startup.
    
- `IOptionsSnapshot<T>` → Creates a **new instance per request**, useful in scoped services.
    
- `IOptionsMonitor<T>` → Supports **real-time reload** when configuration changes without restarting the app.
    
- You’re exposing the `AttachmentOptions` values through an API endpoint.
    

---

# ✅ Key Learnings from This Code

1. Use **POCO classes** for clean configuration mapping.
    
2. Add configuration to **appsettings.json** under structured sections.
    
3. Multiple registration options exist, but **`Configure<T>`** is the best practice.
    
4. Use `IOptions`, `IOptionsSnapshot`, and `IOptionsMonitor` depending on your needs (singleton, per-request, or live reload).
    
5. You can safely expose or consume configuration values in controllers/services without string-based keys.
    

---

```