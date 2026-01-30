




## **1. Log Levels (Types of Logs)**

ASP.NET Core logging defines different severity levels. From highest to lowest importance:

- **Critical** → System crash, app is down, very severe errors.
    
- **Error** → An error occurred, app still runs but something failed.
    
- **Warning** → Potential problem, recoverable issue (e.g., record not found).
    
- **Information** → General flow of the app (e.g., user logged in, order placed).
    
- **Debug** → Detailed developer-focused logs for debugging during development.
    
- **Trace** → Very detailed logs, often too noisy, used for deep diagnostics.
    

👉 **Rules & Purpose:**

- Use higher levels (`Critical`, `Error`, `Warning`) for production monitoring.
    
- Use lower levels (`Information`, `Debug`, `Trace`) mainly for **development**.
    

---

## **2. Production Log Level**

In production:

- Typically start at **Information** or **Warning**.
    
- **Turn off Debug & Trace** because they generate **too much noise** and affect performance.
    

Example from `appsettings.json`:

```json
"Logging": {
  "LogLevel": {
    "Default": "Information",
    "Microsoft.AspNetCore": "Warning"
  }
}
```

👉 This means: **only log `Information` and above** for the app, and **only log `Warning` and above** for ASP.NET Core framework.

---

## **3. Where to Place Logs in the System**

Best practices for where to put logs:

1. **Critical sections** – startup failures, DB connection issues, payment processing.
    
    ```csharp
    _logger.LogCritical("Database connection failed!");
    ```
    
2. **Error handling blocks** – exceptions in code.
    
    ```csharp
    _logger.LogError(ex, "Error while processing order {orderId}", orderId);
    ```
    
3. **Warnings** – suspicious but non-breaking events.
    
    ```csharp
    _logger.LogWarning("id with #{id} not found", id);
    ```
    
4. **Information** – important milestones in business flow.
    
    ```csharp
    _logger.LogInformation("User {userId} logged in", userId);
    ```
    
5. **Debug/Trace** – deep-dive details, only in dev.
    
    ```csharp
    _logger.LogDebug("Getting By ID : {id}", id);
    ```
    

---

## **4. Log Providers**

Log providers determine **where logs are stored**.

- **Console** – shows logs in terminal.
    
- **Debug** – writes to Visual Studio debug output.
    
- **File / Serilog / Seq / Elastic / Azure** – external logging systems.
    

Example (`Program.cs`):

```csharp
builder.Services.AddLogging(cfg =>
{
    cfg.AddDebug();   // log to Visual Studio debug window
    cfg.AddConsole(); // log to console
});
```

---

## **5. `appsettings.json` LogLevel**

The `LogLevel` section controls **minimum log level** per category.

Example:

```json
"Logging": {
  "LogLevel": {
    "Default": "Information",
    "Microsoft.AspNetCore": "Warning"
  }
}
```

👉 Means:

- Application logs: **Information and higher**
    
- ASP.NET Core logs: **Warning and higher**
    

So:

- If log level = `Information` → **Debug & Trace are ignored**.
    
- If log level = `Warning` → **Error & Critical still logged, Info ignored**.
    

---

## **6. Structured Logging**

Structured logging allows you to **pass parameters** instead of string concatenation.

✅ Good:

```csharp
_logger.LogWarning("id with #{id} not found", id);
```

- Allows searching `id=5` later in log systems like Seq/Elastic.
    

❌ Bad:

```csharp
_logger.LogWarning("id with :" + id + " not found");
```

---

## **7. Adding Another Provider (Console)**

You already have `Debug`. Add console as well:

```csharp
builder.Services.AddLogging(cfg =>
{
    cfg.AddDebug();
    cfg.AddConsole();
});
```

Now logs show in **debug window** + **terminal console**.

---

## **8. Filter Log by Category (Per Provider Level)**

You can **filter logs per provider** in `appsettings.json`.

Example:

```json
"Logging": {
  "LogLevel": {
    "Default": "Information"
  },
  "Console": {
    "LogLevel": {
      "Microsoft.AspNetCore": "Warning",
      "AspNetCoreFinal.Controllers.WeatherForecastController": "Debug"
    }
  },
  "Debug": {
    "LogLevel": {
      "Default": "Error"
    }
  }
}
```

👉 Explanation:

- **Console** provider → will log **Debug** level for `WeatherForecastController`.
    
- **Debug** provider → will log only **Error & Critical** globally.
    
- This way filtering is **per provider**, not global.
    

---



### 🔹 Extra Logging Concepts in **your code**

Looking at your `WeatherForecastController` + `Program.cs` + `appsettings.json`, here’s what you have that wasn’t directly in your 8 points:

1. **`ILogger<T>` Dependency Injection**
    
    - You used `_logger = logger` in your controller’s constructor.
        
    - This is ASP.NET Core’s built‑in logging DI system, automatically provided by the framework.
        
    - Purpose: each controller/service gets a logger tied to its type → so log messages include category (`AspNetCoreFinal.Controllers.WeatherForecastController`).
        
    
    Example in your code:
    
    ```csharp
    public WeatherForecastController(ILogger<WeatherForecastController> logger)
    {
        _logger = logger;
    }
    ```
    
2. **Action Filter `[LogSensitiveAction]`**
    
    - You decorated the controller with a custom filter.
        
    - Even though you didn’t explain it in the 8 points, it looks like a cross‑cutting logging concern.
        
    - Filters are good places to log: request/response, performance, sensitive actions, exceptions.
        
3. **Default Provider Added (`Debug`)**
    
    - In `Program.cs` you used:
        
        ```csharp
        builder.Services.AddLogging(cfg => cfg.AddDebug());
        ```
        
    - This adds **Debug provider** only.
        
    - Without adding Console/File, logs would only appear in **Visual Studio Output window**.
        
4. **Log Categories in `appsettings.json`**
    
    - You already specified `"Microsoft.AspNetCore": "Information"`.
        
    - That is a **category filter** → you limited logs from framework code to `Information+`.
        
    - It aligns with point #8, but you didn’t explicitly say you had it already.
        

---


```