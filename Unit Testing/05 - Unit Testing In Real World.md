

---

## المفهوم الجوهري (Core Concept)

عند اختبار تطبيق حقيقي متعدد الطبقات (N-Tier Architecture)، يجب أن نكون انتقائيين في **نطاق الاختبار (Testing Scope)**: نركز بقوة على **Service Layer** (المنطق)، ونناقش جدوى اختبار **Controller Layer** (Mapping & Status Codes)، بينما نتجنب تماماً عمل Unit Tests للـ **Data Layer** لأنها تضيع للوقت وتناسب الـ Integration Tests أكثر.

---

## الشرح التفصيلي (Detailed Breakdown)

### 1. نظرة عامة على المشروع (Users API Project)

المشروع عبارة عن **ASP.NET Core Web API** متكامل يتعامل مع قاعدة بيانات حقيقية (**SQLite**).

- **الوظائف (Functionality):** عمليات CRUD كاملة (Get All, Get By ID, Create, Delete).
    
- **التقنيات المستخدمة:**
    
    - **Dependency Injection (DI):** يتم حقن الـ Services و Repositories والـ Logger في ملف `Program.cs`.
        
    - **Database Initializer:** كلاس مسؤول عن تهيئة قاعدة البيانات وإضافة بيانات أولية (Seeding) عند بدء التشغيل.
        

### 2. هيكلية المشروع (Project Layers)

ينقسم المشروع إلى ثلاث طبقات رئيسية، ولكل منها استراتيجية اختبار مختلفة:

- **A. طبقة الـ API (Controller Layer):**
    
    - **الدور:** استقبال الطلبات (Requests)، استدعاء الـ Service، إرجاع الـ Response المناسب (HTTP 200, 404, etc.)، وعمل **Mapping** من `Domain Object` إلى `Contract Object` (مثل `UserResponse`).
        
    - **استراتيجية الاختبار:** _محل جدل (Debatable)_.
        
        - البعض يفضل عدم اختبارها Unit Testing.
            
        - المحاضر يرى قيمة في اختبارها للتأكد من صحة الـ **Mapping** ومنطق الـ **Status Codes** (مثلاً: إرجاع 404 إذا كان المستخدم `null`).
            
- **B. طبقة التطبيق (Service/Domain Layer):**
    
    - **الدور:** تحتوي على "جوهر" المنطق (Business Logic)، التعامل مع الاستثناءات (Exception Handling)، والـ **Logging** (الذي قد يكون حيوياً لأنظمة المراقبة والإنذار).
        
    - **استراتيجية الاختبار:** _عالية الأهمية (Critical)_.
        
        - هنا سنكتب معظم اختبارات الـ Unit Tests.
            
- **C. طبقة البيانات (Data/Repository Layer):**
    
    - **الدور:** تنفيذ استعلامات SQL مباشرة (Select, Insert, etc.) والاتصال بقاعدة البيانات.
        
    - **استراتيجية الاختبار:** _لا تقم باختبارها (Do Not Unit Test)_.
        
        - لماذا؟ لأنك ستقوم بعمل Mock لكل شيء (Connection, Command, Reader)، وفي النهاية لن يتبقى كود حقيقي لاختباره. أنت هنا تختبر "هل يستطيع النظام الاتصال بقاعدة البيانات؟" وهذا دور الـ **Integration Testing**.
            

---

## أمثلة الكود (Code Snippets)

إليك مثال يوضح المنطق الموجود في الـ **Controller** والذي يرى المحاضر أنه يستحق الاختبار (Mapping & Status Logic):



```csharp
[ApiController]
[Route("[controller]")]
public class UsersController : ControllerBase
{
    private readonly IUserService _userService;

    public UsersController(IUserService userService)
    {
        _userService = userService;
    }

    [HttpGet("{id:guid}")]
    public async Task<IActionResult> GetById(Guid id)
    {
        // 1. Call Logic
        var user = await _userService.GetByIdAsync(id);

        // 2. Status Code Logic (Testable Scenario)
        if (user == null)
        {
            return NotFound(); // Returns 404
        }

        // 3. Mapping Logic (Testable Scenario)
        var response = user.ToUserResponse(); // Extension method for mapping
        return Ok(response); // Returns 200 with specific contract
    }
}
```

**شرح المنطق:**

- إذا قمنا باختبار هذا الكلاس، فنحن لا نختبر جلب البيانات (هذا دور الـ Mock)، بل نختبر: "هل يرجع الـ Controller كود 404 فعلاً عندما تعيد الـ Service قيمة null؟".
    

---

## لماذا؟ (The "Why")

- **لماذا نتجنب Unit Testing للـ Repository؟**
    
    - لأن الـ Unit Test يعتمد على العزل (Isolation) والمحاكاة (Mocking). إذا قمت بعمل Mock لاتصال قاعدة البيانات، فأنت فعلياً تختبر "الوهم" وليس استعلام الـ SQL الحقيقي. القيمة المضافة هنا صفر تقريباً.
        
- **لماذا نختبر الـ Logging في الـ Service؟**
    
    - لأن الـ Logs ليست مجرد نصوص، بل قد تكون مرتبطة بأنظمة مراقبة (Observability Tools) تطلق إنذارات (Alerts) بناءً على رسائل خطأ معينة. التأكد من كتابة الـ Log الصحيح هو جزء من الـ Business Requirements.
        
- **لماذا نستخدم Postman في البداية؟**
    
    - لفهم السلوك الخارجي للـ API (The Contract) قبل الغوص في الكود الداخلي.
        

---

> [!tip] نصيحة: استراتيجية الاختبار
> 
> لا تضيع وقتك في محاولة الوصول لـ 100% Code Coverage باختبار الـ Repositories التي تحتوي فقط على أوامر SQL. ركز جهدك في الـ **Service Layer** حيث يوجد التعقيد الحقيقي، وفي الـ **Controller** للتأكد من الـ API Contract.

> [!warning] تحذير
> 
> الـ Repository Layer مكانه الصحيح هو الـ **Integration Testing** (باستخدام Docker مثلاً لإنشاء قاعدة بيانات حقيقية مؤقتة)، وليس الـ Unit Testing.



---

## المفهوم الجوهري (Core Concept)

كتابة الـ Unit Tests للطبقات التي تحتوي على `ILogger` تشكل تحدياً لأن دوال التسجيل (مثل `LogInformation`) هي دوال ثابتة (Static Extension Methods) لا يمكن عمل Mock لها بسهولة. الحل هو تطبيق نمط **Adapter Pattern** لتغليف الـ Logger داخل واجهة (Interface) خاصة بنا يمكننا اختبارها.

---

## الشرح التفصيلي (Detailed Breakdown)

### 1. إعداد اختبارات الـ Service

- نبدأ بإنشاء مشروع الاختبار `Users.Api.Tests.Unit`.
    
- نقوم بإعداد الـ **SUT (System Under Test)** وهو `UserService`.
    
- نحتاج لعمل Mocks لجميع الاعتماديات:
    
    - `IUserRepository` (سهل باستخدام NSubstitute).
        
    - `ILogger<UserService>` (صعب، وسنرى لماذا).
        

### 2. اختبار المنطق الأساسي (Happy Path & Data)

نقوم بكتابة اختبارين أساسيين للمنطق:

1. **سيناريو القائمة الفارغة:** نتأكد أن الدالة تعيد قائمة فارغة عندما يكون الـ Repository فارغاً.
    
2. **سيناريو وجود بيانات:** نتأكد أن الدالة تعيد المستخدمين الصحيحين وتقارنهم ببعضهم باستخدام `BeEquivalentTo`.
    

### 3. معضلة اختبار الـ Logging

- **المشكلة:** نريد التأكد من أن الكود قام باستدعاء `_logger.LogInformation(...)`.
    
- **العقبة:** الدالة `LogInformation` ليست جزءاً من `ILogger` مباشرة، بل هي **Extension Method** ثابتة (Static) قامت Microsoft بإضافتها.
    
- **لماذا يفشل الاختبار؟** لأن مكتبات الـ Mocking (مثل NSubstitute) تستطيع فقط عمل Mock للدوال الموجودة داخل الـ Interface، ولا تستطيع تتبع استدعاءات الدوال الثابتة (Static Methods). عند محاولة التحقق باستخدام `.Received()`، يفشل الاختبار لأن الاستدعاء الحقيقي يذهب لطبقة أعمق ومعقدة (`FormattedLogValues`) لا يمكننا الوصول إليها.
    

### 4. الحل: نمط الـ Adapter (Testing Untestable Code)

لحل هذه المشكلة، نقوم بإنشاء "غلاف" (Wrapper) خاص بنا حول الـ Logger الأصلي:

1. **إنشاء الواجهة `ILoggerAdapter<T>`:** تحتوي على الدوال التي نريد استخدامها (`LogInformation`, `LogError`).
    
2. **التطبيق `LoggerAdapter<T>`:** يقوم داخلياً باستدعاء الـ `Logger` الحقيقي.
    
3. **الاستخدام:** نستبدل `ILogger` في الـ `UserService` بـ `ILoggerAdapter`.
    
4. **النتيجة:** الآن `LogInformation` أصبحت دالة عادية داخل Interface، ويمكننا عمل Mock لها والتحقق منها بسهولة.
    

---

## أمثلة الكود (Code Snippets)

### 1. إعداد الاختبار (Setup)



```csharp
public class UserServiceTests
{
    private readonly UserService _sut;
    private readonly IUserRepository _userRepository = Substitute.For<IUserRepository>();
    private readonly ILogger<UserService> _logger = Substitute.For<ILogger<UserService>>(); // سنستبدله لاحقاً

    public UserServiceTests()
    {
        _sut = new UserService(_userRepository, _logger);
    }
}
```

### 2. اختبار الـ Happy Path



```csharp
[Fact]
public async Task GetAllAsync_ShouldReturnUsers_WhenUsersExist()
{
    // Arrange
    var expectedUser = new User { FullName = "Nick Chapsas" };
    _userRepository.GetAllAsync().Returns(new[] { expectedUser });

    // Act
    var result = await _sut.GetAllAsync();

    // Assert
    result.Should().BeEquivalentTo(new[] { expectedUser });
}
```

### 3. محاولة اختبار الـ Logging الفاشلة (للتوضيح)



```csharp
// ❌ هذا الاختبار سيفشل لأن LogInformation دالة Extension
[Fact]
public async Task GetAllAsync_ShouldLogMessages_WhenInvoked()
{
    // Act
    await _sut.GetAllAsync();

    // Assert (Fails!)
    _logger.Received(1).LogInformation("Retrieving all users"); 
}
```

### 4. تطبيق الـ LoggerAdapter (الحل)



```csharp
// 1. الواجهة الجديدة
public interface ILoggerAdapter<T>
{
    void LogInformation(string message, params object[] args);
    void LogError(Exception ex, string message, params object[] args);
}

// 2. التطبيق (Production Code)
public class LoggerAdapter<T> : ILoggerAdapter<T>
{
    private readonly ILogger<T> _logger;
    public LoggerAdapter(ILogger<T> logger) => _logger = logger;

    public void LogInformation(string message, params object[] args) 
        => _logger.LogInformation(message, args); // تفويض للوجر الحقيقي
    
    // ...
}
```

---

## لماذا؟ (The "Why")

- **لماذا لا نستطيع عمل Mock للـ Extension Methods؟**
    
    - لأنها في الحقيقة مجرد دوال ثابتة (`static methods`) معرفة في كلاس آخر. الـ Compiler يحول `logger.LogInformation(...)` إلى `LoggerExtensions.LogInformation(logger, ...)`. مكتبات الـ Mocking تعمل على الـ Instance Methods (Virtual/Interface) ولا تملك سلطة على الـ Static Calls.
        
- **لماذا الـ Adapter Pattern؟**
    
    - لأنه يفصل كودنا عن تعقيدات المكتبات الخارجية (Framework Code). بجعل الكود يعتمد على `ILoggerAdapter` (الذي نملكه نحن)، نصبح أحراراً في تصميمه واختباره كيفما نشاء، ونعزل الاعتماد على `Microsoft.Extensions.Logging` في مكان واحد فقط.
        

---

> [!tip] نصيحة: متى نستخدم الـ LoggerAdapter؟
> 
> إذا كان فريقك يعتمد بشدة على الـ Logs للمراقبة وتتبع الأخطاء، فإن استخدام `LoggerAdapter` ضروري لاختبار هذه الـ Logs. أما إذا كانت الـ Logs مجرد "زينة" ولا تؤثر على البيزنس، فقد يفضل البعض تجاهل اختبارها لتجنب التعقيد الإضافي.

---

هذا هو ملخص **الجزء الثاني (b) من السيكشن الرابع**، وهو الجزء التطبيقي والعملي الذي يربط كل المفاهيم ببعضها. هنا نرى كيف يتم تفعيل الـ `LoggerAdapter` داخل التطبيق الحقيقي، وكيف نستخدم قدرات **NSubstitute** المتقدمة لاختبار السيناريوهات المعقدة (Arguments Matching & Exceptions).

---

## المفهوم الجوهري (Core Concept)

بعد تصميم الـ **Adapter Pattern**، تتبقى خطوتان لجعله فعالاً: أولاً، **ربطه (Wiring)** بنظام الـ DI الخاص بالتطبيق ليعمل في الواقع. وثانياً، استخدام **Argument Matchers** (مثل `Arg.Is` و `Arg.Any`) في الاختبارات للتعامل مع القيم غير المتوقعة (مثل الزمن المتغير) والتحقق من الآثار الجانبية (Side Effects) بدقة.

---

## الشرح التفصيلي (Detailed Breakdown)

### 1. تطبيق الـ Adapter وربطه (Implementation & Registration)

- **التنفيذ:** قام المحاضر بإكمال كلاس `LoggerAdapter` بتمرير الاستدعاءات (`Forwarding Calls`) إلى الـ Logger الحقيقي.
    
- **التسجيل (DI Registration):** لكي يعرف التطبيق كيفية إنشاء `ILoggerAdapter`، يجب تسجيله في ملف `Program.cs` (أو `Startup.cs` قديماً).
    
    - نستخدم `AddTransient` لربط الواجهة (`ILoggerAdapter`) بالتطبيق (`LoggerAdapter`).
        

### 2. اختبار الـ Logging (التحقق من Side Effects)

الآن بعد أن أصبح لدينا `ILoggerAdapter` (والذي هو Interface عادية)، يمكننا عمل Mock له والتحقق من استدعائه:

- **`Received(1)`:** دالة من NSubstitute تتحقق من أن الميثود تم استدعاؤها مرة واحدة بالضبط.
    
- **التحقق الدقيق:** يمكننا التأكد من أن الرسالة المرسلة للوجر تطابق النص المتوقع حرفياً.
    

### 3. التعامل مع المتغيرات (Argument Matchers)

ماذا لو كانت الرسالة تحتوي على جزء متغير؟ (مثلاً: "Retrieved users in **5ms**"). الـ 5ms ستتغير في كل مرة تشغل فيها الاختبار، مما سيكسر مقارنة النصوص الحرفية.

- **الحل:** نستخدم أدوات المطابقة من NSubstitute:
    
    - **`Arg.Is<T>(lambda)`:** للتحقق من شرط معين (مثلاً: النص يبدأ بـ "Retrieving").
        
    - **`Arg.Any<T>()`:** لتجاهل القيمة تماماً (مثلاً: لا يهمنا كم مللي ثانية استغرق، المهم أنه تم تمرير رقم).
        

### 4. اختبار الاستثناءات (Exception Handling Test)

هذا هو السيناريو الأهم للـ Unhappy Path. نريد التأكد من أنه "عندما تفشل قاعدة البيانات، نقوم بتسجيل الخطأ وإعادة رميه".

- **Arrange:** نبرمج الـ Mock ليرمي `SQLiteException`.
    
- **Act:** نستخدم `Func` لأننا نتوقع Exception (كما شرحنا سابقاً).
    
- **Assert:**
    
    1. نتحقق أن الـ Service رمت الـ Exception (`Should().ThrowAsync`).
        
    2. نتحقق أن الـ Logger سجل هذا الخطأ (`Received(1).LogError(...)`).
        

---

## أمثلة الكود (Code Snippets)

### 1. تسجيل الـ Adapter في Program.cs



```csharp
// في ملف Program.cs
builder.Services.AddTransient(typeof(ILoggerAdapter<>), typeof(LoggerAdapter<>));
```

_هذا السطر يخبر الـ .NET: "كلما طلب أحدهم `ILoggerAdapter<T>`، أعطه نسخة من `LoggerAdapter<T>`"._

### 2. اختبار الـ Logging باستخدام Matchers



```csharp
[Fact]
public async Task GetAllAsync_ShouldLogMessages_WhenInvoked()
{
    // Arrange
    _userRepository.GetAllAsync().Returns(Enumerable.Empty<User>());

    // Act
    await _sut.GetAllAsync();

    // Assert 1: التحقق بجزء من النص (Arg.Is)
    _logger.Received(1).LogInformation(
        Arg.Is<string>(msg => msg.StartsWith("Retrieving")) // الشرط
    );

    // Assert 2: تجاهل الزمن المتغير (Arg.Any)
    _logger.Received(1).LogInformation(
        Arg.Is<string>(msg => msg.StartsWith("All users retrieved in")), // الرسالة
        Arg.Any<object[]>() // نتجاهل وسائط الزمن (Arguments)
    );
}
```

### 3. اختبار سيناريو الخطأ (Exception & Error Logging)



```csharp
[Fact]
public async Task GetAllAsync_ShouldLogMessageAndException_WhenExceptionIsThrown()
{
    // Arrange
    var expectedException = new SqliteException("Something went wrong", 500);
    // برمجة الـ Mock ليرمي خطأ
    _userRepository.GetAllAsync().Throws(expectedException); 

    // Act
    // نغلف الاستدعاء في Func لأننا نتوقع فشله
    Func<Task> action = async () => await _sut.GetAllAsync();

    // Assert
    // 1. التحقق من أن الخطأ تم رميه
    await action.Should().ThrowAsync<SqliteException>()
        .WithMessage("Something went wrong");

    // 2. التحقق من أن الخطأ تم تسجيله في الـ Logger
    _logger.Received(1).LogError(
        expectedException, // هل تم تمرير نفس الـ Exception object؟
        "Something went wrong while retrieving all users" // هل الرسالة صحيحة؟
    );
}
```

---

## لماذا؟ (The "Why")

- **لماذا نستخدم `Arg.Any`؟**
    
    - للتغلب على الـ **Non-Determinism**. إذا كان الكود يعتمد على الوقت (Time)، أو أرقام عشوائية (Random)، أو عناوين ذاكرة متغيرة، فإن استخدام القيم الثابتة في الـ Assert سيجعل الاختبارات هشة (Flaky Tests) تفشل عشوائياً. `Arg.Any` تقول: "أنا أهتم بأن هناك قيمة مررت، لكن لا يهمني ما هي بالضبط".
        
- **لماذا نختبر الـ Exception Logging؟**
    
    - لأن فقدان الـ Logs عند حدوث خطأ في الـ Production هو كابوس. إذا ابتلع التطبيق الخطأ دون تسجيله، لن تتمكن من معرفة سبب المشكلة. هذا الاختبار يضمن أن نظام المراقبة لديك سيعمل عند الأزمات.
        

---

> [!tip] نصيحة: NSubstitute Syntax
> 
> لاحظ الفرق في الكتابة:
> 
> - عند **إعداد** القيمة (Setup): نستخدم `.Returns(...)`.
>     
> - عند **إعداد** الخطأ (Exception): نستخدم `.Throws(...)`.
>     
> - عند **التحقق** من الاستدعاء (Assert): نستخدم `.Received(...)`.
>     

> [!warning] تنبيه
> 
> تأكد دائماً عند استخدام `Arg.Is` أو `Arg.Any` أنك تستخدمها داخل استدعاء الميثود في الـ `Received(...)`. استخدامها خارج هذا السياق لن يعمل وسيسبب سلوكاً غريباً في الاختبار.



---

## المفهوم الجوهري (Core Concept)

الـ Unit Testing هي عملية تكرارية ومنهجية (Iterative & Systematic). بمجرد إتقانك للأنماط الأساسية (Testing Return Values, Logging, Exceptions) لدالة واحدة (مثل `GetAllAsync`)، يمكنك تطبيق نفس الأنماط على باقي الدوال (`GetById`, `Create`, `Delete`) مع تعديلات طفيفة في البيانات والرسائل.

---

## الشرح التفصيلي (Detailed Breakdown)

### 1. اختبار `GetByIdAsync`

- **سيناريو عدم الوجود (Null Scenario):**
    
    - **Arrange:** برمجة الـ Mock ليرجع `null` لأي ID (`Arg.Any<Guid>()`).
        
    - **Assert:** التحقق من أن النتيجة هي `null`.
        
- **سيناريو الوجود (Success Scenario):**
    
    - **Arrange:** إنشاء `User` محدد، وبرمجة الـ Mock ليرجعه عند طلب الـ ID الخاص به.
        
    - **Assert:** التحقق من أن النتيجة تكافئ الـ `User` المتوقع (`BeEquivalentTo`).
        
- **اختبار الـ Logging:** التحقق من رسائل "Retrieving user..." بنفس طريقة الـ Argument Matchers.
    

### 2. اختبار `CreateAsync`

- **سيناريو النجاح:**
    
    - **Arrange:** الـ Mock يرجع `true`.
        
    - **Assert:** النتيجة يجب أن تكون `true`.
        
- **اختبار الـ Logging:** هنا نتحقق من أن رسالة "Creating user with ID and Name" تحتوي على الـ ID والـ Name الصحيحين (`Arg.Is<User>(u => u.Id == ...)`).
    
- **اختبار الاستثناءات:** التأكد من تسجيل الخطأ عند فشل الإنشاء.
    

### 3. اختبار `DeleteAsync`

- **سيناريو الحذف الناجح:** الـ Mock يرجع `true` $\rightarrow$ النتيجة `true`.
    
- **سيناريو الفشل (Not Found):** الـ Mock يرجع `false` $\rightarrow$ النتيجة `false`.
    
- **اختبار الـ Logging:** التحقق من رسائل الحذف والزمن المستغرق.
    

---

## أمثلة الكود (Code Snippets)

### 1. اختبار `GetByIdAsync` (سيناريو الوجود)



```csharp
[Fact]
public async Task GetByIdAsync_ShouldReturnUser_WhenUserExists()
{
    // Arrange
    var user = new User { Id = Guid.NewGuid(), FullName = "Nick" };
    // برمجة الـ Mock: "عندما أطلب هذا الـ ID، أرجع هذا المستخدم"
    _userRepository.GetByIdAsync(user.Id).Returns(user);

    // Act
    var result = await _sut.GetByIdAsync(user.Id);

    // Assert
    result.Should().BeEquivalentTo(user);
}
```

### 2. اختبار `CreateAsync` (Logging Validation)



```csharp
[Fact]
public async Task CreateAsync_ShouldLogMessages_WhenInvoked()
{
    // Arrange
    var user = new User { Id = Guid.NewGuid(), FullName = "Nick" };
    _userRepository.CreateAsync(user).Returns(true);

    // Act
    await _sut.CreateAsync(user);

    // Assert
    _logger.Received(1).LogInformation(
        Arg.Is<string>(s => s.StartsWith("Creating user")), // الرسالة
        Arg.Is<object[]>(args => 
            args[0].Equals(user.Id) && args[1].Equals(user.FullName)) // الباراميترز
    );
}
```

### 3. اختبار `DeleteAsync` (Exception Logging)



```csharp
[Fact]
public async Task DeleteByIdAsync_ShouldLogException_WhenExceptionIsThrown()
{
    // Arrange
    var userId = Guid.NewGuid();
    var exception = new SqliteException("Error", 500);
    _userRepository.DeleteByIdAsync(userId).Throws(exception);

    // Act
    Func<Task> action = async () => await _sut.DeleteByIdAsync(userId);

    // Assert
    await action.Should().ThrowAsync<SqliteException>();
    
    _logger.Received(1).LogError(
        exception,
        Arg.Is<string>(msg => msg.Contains("Something went wrong while deleting"))
    );
}
```

---

## لماذا؟ (The "Why")

- **لماذا نكرر نفس الاختبارات لكل دالة؟**
    
    - لأن كل دالة لها منطقها الخاص ورسائلها الخاصة. قد تكون `GetAll` سليمة لكن `Delete` بها خطأ في رسالة الـ Log أو لا تتعامل مع الـ Exception بشكل صحيح. الـ Copy-Paste مع التعديل (Copy-Modify) هنا هو ممارسة مقبولة لضمان التغطية الكاملة.
        
- **لماذا نستخدم `Arg.Is` مع الـ Objects؟**
    
    - في دالة `CreateAsync`، الـ Logger يستقبل الـ `User` كـ Argument. للتحقق من أن القيم المسجلة (Logged Values) صحيحة، نحتاج لفتح المصفوفة `args[]` والتحقق من العنصر الأول (ID) والعنصر الثاني (Name).
        

---

> [!tip] نصيحة: Refactoring Tests
> 
> عندما تجد نفسك تكرر إنشاء `User` أو `Guid` في كل اختبار، يمكنك نقل هذه العمليات إلى دوال مساعدة (Helper Methods) أو استخدام مكتبات مثل `AutoFixture` (موضوع متقدم) لتقليل حجم الكود.

> [!warning] تنبيه
> 
> انتبه دائماً للرسائل النصية (String Messages). خطأ إملائي بسيط في الاختبار ("Creatng" بدلاً من "Creating") سيسبب فشل الاختبار حتى لو كان الكود سليماً. حاول استخدام الثوابت (Constants) للرسائل إذا أمكن.



---

## المفهوم الجوهري (Core Concept)

على الرغم من الجدل حول جدوى اختبار الـ **Controllers** (حيث يفضل البعض الـ Integration Tests)، يرى المحاضر قيمة في كتابة Unit Tests سريعة للتحقق من شيئين أساسيين:

1. **HTTP Status Codes:** هل نرجع `404` عند عدم وجود بيانات؟ و `200` عند النجاح؟
    
2. **Contract Mapping:** هل يتم تحويل الـ Domain Models إلى API Contracts بشكل صحيح؟
    

---

## الشرح التفصيلي (Detailed Breakdown)

### 1. لماذا نختبر الـ Controllers؟

- **السرعة:** اختبار الـ Unit Test ينفذ في ملي ثانية، بينما الـ Integration Test قد يستغرق ثواني أو دقائق. اكتشاف خطأ `404` مبكراً يوفر الوقت.
    
- **التحقق من العقد (Contracts):** الـ Controller هو "واجهة المحل" للـ API. التأكد من أنه يرجع البيانات بالشكل المتفق عليه (DTOs) أمر حيوي لتجنب الـ Breaking Changes مع الـ Front-end.
    

### 2. استراتيجية اختبار الـ Controller

على عكس الـ Service، هنا نتعامل مع `IActionResult`. لذا، نحتاج لخطوة إضافية وهي **Casting**:

- نستدعي الـ Method.
    
- نقوم بتحويل النتيجة (Cast) إلى النوع المتوقع (مثل `OkObjectResult` أو `NotFoundResult`).
    
- نتحقق من الـ `StatusCode` والـ `Value` (الجسم العائد).
    

### 3. سيناريوهات الاختبار (Test Scenarios)

- **GetById (User Exists):**
    
    - نتوقع `OkObjectResult` (200).
        
    - نتوقع أن الـ Body يحتوي على `UserResponse` (وليس `User` مباشرة).
        
- **GetById (User Not Found):**
    
    - نتوقع `NotFoundResult` (404).
        
- **GetAll (With Data):**
    
    - نتوقع `OkObjectResult` (200).
        
    - نتوقع قائمة من `UserResponse`.
        
- **GetAll (Empty):**
    
    - نتوقع `OkObjectResult` (200) مع قائمة فارغة (وليس `null` أو `204 No Content` حسب التصميم).
        

---

## أمثلة الكود (Code Snippets)

### 1. إعداد الاختبار (Setup)



```csharp
public class UserControllerTests
{
    private readonly UserController _sut;
    private readonly IUserService _userService = Substitute.For<IUserService>();

    public UserControllerTests()
    {
        _sut = new UserController(_userService);
    }
}
```

### 2. اختبار GetById (سيناريو النجاح)



```csharp
[Fact]
public async Task GetById_ReturnsOkAndObject_WhenUserExists()
{
    // Arrange
    var user = new User { Id = Guid.NewGuid(), FullName = "Nick" };
    _userService.GetByIdAsync(user.Id).Returns(user);
    
    // محاكاة عملية الـ Mapping المتوقعة
    var expectedResponse = user.ToUserResponse();

    // Act
    var result = await _sut.GetById(user.Id);

    // Assert
    // 1. التحقق من النوع والـ Status Code
    var okResult = result.Should().BeOfType<OkObjectResult>().Subject;
    okResult.StatusCode.Should().Be(200);

    // 2. التحقق من المحتوى (Mapping)
    okResult.Value.Should().BeEquivalentTo(expectedResponse);
}
```

### 3. اختبار GetById (سيناريو 404)



```csharp
[Fact]
public async Task GetById_ReturnsNotFound_WhenUserDoesNotExist()
{
    // Arrange
    _userService.GetByIdAsync(Arg.Any<Guid>()).Returns((User)null);

    // Act
    var result = await _sut.GetById(Guid.NewGuid());

    // Assert
    // التحقق من أنه NotFoundResult (404)
    result.Should().BeOfType<NotFoundResult>();
    
    // ملاحظة: NotFoundResult ليس له Value، فقط Status Code
    ((NotFoundResult)result).StatusCode.Should().Be(404);
}
```

---

## لماذا؟ (The "Why")

- **لماذا نحتاج لعمل Cast للنتيجة (`as OkObjectResult`)؟**
    
    - لأن `IActionResult` هو مجرد Interface عام. للوصول لخاصية `.Value` أو `.StatusCode`، يجب أن نخبر الكومبايلر بالنوع المحدد الذي نتوقعه.
        
- **لماذا نختبر الـ Mapping هنا؟**
    
    - لأن الـ Controller هو المسؤول عن تحويل البيانات من "لغة الداخل" (Domain) إلى "لغة الخارج" (Contracts). إذا تغير الـ Domain Model، هذا الاختبار سيحميك من كسر الـ API Clients عن طريق الخطأ.
        

---

> [!tip] نصيحة: Fluent Assertions مع الأنواع
> 
> مكتبة Fluent Assertions توفر طريقة أنيقة للـ Casting والتحقق في سطر واحد:
> 
> `result.Should().BeOfType<OkObjectResult>().Subject.Value.Should()...`
> 
> هذا يغنيك عن الـ Casting اليدوي `(OkObjectResult)result`.

> [!warning] تنبيه
> 
> تذكر أنك هنا تختبر منطق الـ Controller فقط. لا تقم باختبار منطق الـ Service مرة أخرى (لقد قمنا بذلك سابقاً). افترض أن الـ Service تعمل بشكل صحيح (عبر الـ Mock) وركز على استجابة الـ HTTP.



---

## المفهوم الجوهري (Core Concept)

اختبار الـ `Create Action` يتطلب دقة عالية لأنه لا يرجع مجرد `200 OK`، بل `201 Created` مع `Location Header` (يحتوي على رابط المورد الجديد). التحدي الأكبر هنا هو **"تطابق البيانات"**، حيث يقوم الـ Controller بإنشاء كائن جديد (New ID) يختلف عن الكائن الموجود في الاختبار، مما يستدعي استخدام تقنيات **Callback** (`Arg.Do`) لالتقاط الكائن الداخلي ومزامنته.

---

## الشرح التفصيلي (Detailed Breakdown)

### 1. اختبار دالة `Create` (سيناريو النجاح - 201 Created)

العملية معقدة لأنها تتضمن عدة خطوات:

1. **Request:** استقبال `CreateUserRequest`.
    
2. **Mapping:** تحويله إلى `User` (هنا يتم توليد ID جديد عشوائي).
    
3. **Service Call:** استدعاء الخدمة.
    
4. **Response:** إرجاع `CreatedAtAction` الذي يحتوي على:
    
    - Status Code: `201`.
        
    - Body: `UserResponse`.
        
    - Header: `Location` (RouteValues).
        

#### مشكلة عدم تطابق الـ IDs (The ID Mismatch Problem)

- **المشكلة:** في الاختبار، نقوم بإنشاء `User` (بـ ID رقم 1 مثلاً). لكن داخل الـ Controller، يتم إنشاء `new User` (بـ ID رقم 2).
    
- **النتيجة:** عند مقارنة النتيجة (`Expected` vs `Actual`)، يفشل الاختبار لأن الـ IDs مختلفة، وبالتالي الـ `RouteValues` لن تتطابق.
    
- **الحل الاحترافي (`Arg.Do` Pattern):**
    
    - نستخدم ميزة `Arg.Do` في NSubstitute لتنفيذ كود معين عند استدعاء الدالة.
        
    - نقوم بـ "التقاط" المستخدم الذي تم إنشاؤه _داخل_ الـ Controller وتحديث متغير الاختبار ليطابقه.
        

### 2. اختبار دالة `Create` (سيناريو الفشل - 400 Bad Request)

- إذا أعادت الـ Service قيمة `false` (فشل الإنشاء)، يجب أن يعيد الـ Controller كود `400 Bad Request`.
    
- نوع النتيجة المتوقع: `BadRequestResult`.
    

### 3. اختبار دالة `Delete` (200 vs 404)

- **النجاح:** إذا أعادت الخدمة `true` $\rightarrow$ نتوقع `OkResult` (وليس `OkObjectResult` لأنه لا يوجد Body).
    
- **الفشل:** إذا أعادت الخدمة `false` $\rightarrow$ نتوقع `NotFoundResult`.
    

---

## أمثلة الكود (Code Snippets)

### 1. اختبار `Create` مع حل مشكلة الـ ID (تقنية Arg.Do)



```csharp
[Fact]
public async Task Create_ReturnsCreated_WhenDataIsValid()
{
    // Arrange
    var request = new CreateUserRequest { FullName = "Nick" };
    var user = new User { Id = Guid.NewGuid(), FullName = "Nick" };
    
    // التقنية المتقدمة: Arg.Do
    // "عندما يتم استدعاء CreateAsync، خذ المستخدم (x) وحدث المتغير user ليساويه"
    _userService.CreateAsync(Arg.Do<User>(x => user = x)).Returns(true);

    // Act
    var result = await _sut.Create(request);

    // Assert
    // 1. التحقق من النوع والحالة
    var createdResult = result.Should().BeOfType<CreatedAtActionResult>().Subject;
    createdResult.StatusCode.Should().Be(201);

    // 2. التحقق من الـ Route Values (Location Header)
    // الآن ستنجح لأننا قمنا بتحديث الـ user ليطابق ما حدث داخل الـ Controller
    createdResult.RouteValues["id"].Should().BeEquivalentTo(user.Id);

    // 3. التحقق من الجسم (Body)
    createdResult.Value.Should().BeEquivalentTo(user.ToUserResponse());
}
```

### 2. اختبار `Delete` (الفرق بين أنواع النتائج)



```csharp
[Fact]
public async Task Delete_ReturnsOk_WhenUserDeleted()
{
    // Arrange
    _userService.DeleteByIdAsync(Arg.Any<Guid>()).Returns(true);

    // Act
    var result = await _sut.DeleteById(Guid.NewGuid());

    // Assert
    // انتبه: OkResult وليس OkObjectResult
    result.Should().BeOfType<OkResult>(); 
    ((OkResult)result).StatusCode.Should().Be(200);
}
```

---

## لماذا؟ (The "Why")

- **لماذا استخدمنا `Arg.Do`؟**
    
    - لأننا لا نملك السيطرة على الكود داخل الـ Controller الذي يقوم بـ `new User()`. الـ `Arg.Do` تسمح لنا بـ "التجسس" (Spying) على القيمة التي تم تمريرها للـ Service واستخدامها في التحقق. بدون هذه الحركة، كنا سنضطر لاستثناء الـ ID من المقارنة (`Excluding(x => x.Id)`), وهو حل "ساذج" (Naive) يقلل من دقة الاختبار.
        
- **لماذا `CreatedAtAction` وليس `Ok`؟**
    
    - هذا هو معيار **RESTful API**. عند إنشاء مورد جديد، يجب إخبار العميل "أين" تم إنشاؤه عبر الـ Header، وكود 201 أدق دلالياً من 200.
        

---

> [!warning] تنبيه: أنواع الـ OK
> 
> - **`OkObjectResult`**: يُستخدم عندما ترجع بيانات (Body) مع الرد (مثل `GetAll`, `GetById`, `Create`).
>     
> - **`OkResult`**: يُستخدم عندما تنجح العملية ولكن لا توجد بيانات لإرجاعها (مثل `Delete`).
>     
> - الخلط بينهما سيسبب فشل الـ Casting في اختباراتك.
>     

> [!tip] نصيحة: الـ Refactoring Safety
> 
> ختم المحاضر بتوضيح أهمية هذه الاختبارات المملة: إذا قرر مطور آخر تغيير الـ Logic (مثلاً إزالة الـ Stopwatch Log أو تغيير الـ Status Code)، ستفشل الاختبارات فوراً. هذا يمنحك "شبكة أمان" (Safety Net) تسمح لك بتطوير وتحسين الكود بجرأة دون خوف من كسر الوظائف الأساسية.

---

