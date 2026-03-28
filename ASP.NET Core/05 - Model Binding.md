

Perfect 👍 Let’s break down **Model Binding in ASP.NET Core** using your reference code step by step.

---

## 1. **Model Binding in General**

Model binding in ASP.NET Core is the process where the framework **automatically maps incoming HTTP request data (Route, QueryString, Headers, Body, Form, etc.) to controller action parameters**.

- Without binding: you’d manually read from `Request.Query` or `Request.Body`.
    
- With binding: ASP.NET Core automatically gives you strongly typed values.
    

👉 Example:

```csharp
[HttpGet("{id}")]
public IActionResult Get(int id)  // id comes from route {id}
{
    return Ok(id);
}


If you call `/api/products/5`, `id` is automatically **bound** to `5`.

---

## 2. **`[FromQuery(Name ="key")]`**

- `FromQuery` tells the binder to look inside the **Query String** (`?key=value`) of the request.
    
- `Name="key"` allows you to bind a query parameter to a differently named method parameter.
    

👉 Example:


public IActionResult Get([FromQuery(Name="page")] int pageNumber)
```

- `/api/products?page=3` → `pageNumber = 3`.
    

⚠️ In your code, you used `[FromRoute(Name="Key")] int id`.  
That means:

- URL: `/WeatherForecast/10` → `id = 10`.
    
- If you changed to `[FromQuery(Name="Key")]`, then `/WeatherForecast?Key=10`.
    

---

## 3. **Complex Types (not just simple int, bool, etc.)**

- If you pass a **complex type (class)**, model binding tries to map properties from **Query string, Route, or Body** depending on the context.
    

👉 Example:

```csharp
public class UserFilter
{
    public int Age { get; set; }
    public string Name { get; set; }
}
```

```csharp
[HttpGet]
public IActionResult Search([FromQuery] UserFilter filter)
```

- Request: `/api/users?age=25&name=Ali`
    
- Bound to: `filter.Age = 25`, `filter.Name = "Ali"`
    

⚠️ By default:

- `[FromQuery]` → Query String.
    
- `[FromBody]` → Request Body (JSON).
    
- `[FromRoute]` → Route.
    
- `[FromHeader]` → Headers.
    

---

## 4. **Send Two Complex Datatypes with Prefixed Names**

If you need to bind multiple complex objects from the query string, you must **prefix property names**.

👉 Example:

```csharp
public class Address
{
    public string City { get; set; }
    public string Zip { get; set; }
}

public class Contact
{
    public string Email { get; set; }
    public string Phone { get; set; }
}
```

```csharp
[HttpGet]
public IActionResult Get([FromQuery] Address address, [FromQuery] Contact contact)
```

Request must be like:

```
/api/test?address.city=NewYork&address.zip=10001&contact.email=test@mail.com&contact.phone=12345
```

👉 Model binding will correctly split them.

---

## 5. **Headers and `[FromHeader]` Attribute**

You can bind values from HTTP headers.

👉 Example:

```csharp
[HttpGet]
public IActionResult Get([FromHeader(Name="X-API-KEY")] string apiKey)
{
    return Ok(apiKey);
}
```

Request:

```
GET /api/test
X-API-KEY: my-secret
```

- `apiKey = "my-secret"`
    

---

## 6. **Best Practices**

✅ Use attributes explicitly (`[FromQuery]`, `[FromBody]`, `[FromRoute]`) for clarity.  
✅ Only allow **one `[FromBody]` parameter per action** (because body can only be read once).  
✅ For complex input → prefer `[FromBody]` with JSON (cleaner than long query strings).  
✅ Validate models with `[ApiController]` → automatic **400 Bad Request** if validation fails.  
✅ Use **DTOs** (Data Transfer Objects), don’t bind directly to EF entities.  
✅ Use **prefixes** when multiple complex types are used in query.  
✅ Avoid overloading routes with both `[FromQuery]` and `[FromBody]` for the same parameter (confusing).

---

## 7. **Other Useful Notes**

- **Default binding sources** in `[ApiController]`:
    
    - Simple types (`int`, `string`) → Query string or Route.
        
    - Complex types → Body (JSON) if not otherwise specified.
        
- **Binding Priority Order**: Route → Query → Form → Body → Custom Binder.
    
- You can write **Custom Model Binders** if you need special parsing logic.
    
- You can use **`[BindRequired]`** to force a property to be provided.
    

👉 Example:

```csharp
public IActionResult Get([BindRequired][FromQuery] int page)
```

If `page` is missing → automatic **400 Bad Request**.

---

📌 In your code:

```csharp
[HttpGet]
[Route("{key}")]
public IEnumerable<WeatherForecast> Get([FromRoute(Name="Key")] int id)
```

- Route: `/WeatherForecast/10`
    
- Model Binding: `id = 10` from `{key}` in route.
    
- If you switched to `[FromQuery(Name="Key")]`, you’d call `/WeatherForecast?Key=10`.
    

---


```