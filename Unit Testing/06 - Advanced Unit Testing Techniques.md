

---

## المفهوم الجوهري (Core Concept)

بشكل افتراضي، يقوم **xUnit** بعزل الاختبارات تماماً عبر إنشاء **Instance** جديد من كلاس الاختبار لكل `TestMethod` (لضمان عدم تداخل البيانات)، ولكن يمكننا كسر هذه القاعدة ومشاركة الـ Context (مثل اتصالات قواعد البيانات) لتحسين الأداء باستخدام **Class Fixtures** (لمشاركة البيانات داخل كلاس واحد) أو **Collection Fixtures** (لمشاركتها عبر عدة كلاسات).

---

## الشرح التفصيلي (Detailed Breakdown)

### 1. السلوك الافتراضي لـ xUnit (Default Behavior)

قبل أن نغير أي شيء، يجب أن نفهم كيف يعمل xUnit "خلف الكواليس":

- **العزل التام (Instantiation per Test):**
    
    - إذا كان لديك كلاس يحتوي على 3 اختبارات، سيقوم xUnit بإنشاء الكلاس (`new TestClass()`) ثلاث مرات.
        
    - **النتيجة:** المتغيرات داخل الـ Constructor يتم إعادة تهيئتها لكل اختبار. لا يوجد Shared State افتراضياً.
        
- **التنفيذ المتسلسل vs المتوازي:**
    
    - **داخل الكلاس الواحد (Same Class):** تعمل الاختبارات بالتسلسل (واحد تلو الآخر).
        
    - **بين كلاسات مختلفة (Different Classes):** تعمل الكلاسات بالتوازي (Parallel Execution) افتراضياً.
        

### 2. مشاركة البيانات داخل نفس الكلاس (Class Fixture)

إذا كان لديك عملية "ثقيلة" (Expensive Setup) مثل إنشاء اتصال بقاعدة بيانات، ولا تريد تكرارها لكل اختبار داخل نفس الكلاس:

- **الحل:** استخدام `IClassFixture<T>`.
    
- **الآلية:**
    
    1. ننشئ كلاس منفصل (الـ Fixture) نضع فيه كود التهيئة (Setup).
        
    2. نجعل كلاس الاختبار يطبق `IClassFixture<MyFixture>`.
        
    3. نحقن الـ Fixture عبر الـ Constructor.
        
- **دورة الحياة (Lifecycle):**
    
    - `Fixture Constructor` (مرة واحدة).
        
    - `Test Class Constructor` (لكل اختبار).
        
    - `Fixture Dispose` (مرة واحدة في النهاية).
        

### 3. مشاركة البيانات عبر عدة كلاسات (Collection Fixture)

ماذا لو أردت مشاركة نفس اتصال قاعدة البيانات (أو Docker Container) بين 5 كلاسات اختبار مختلفة؟ `ClassFixture` لن تفيد هنا لأنها تعيد إنشاء نفسها لكل كلاس.

- **الحل:** استخدام **Collection Fixture**.
    
- **الخطوات:**
    
    1. إنشاء كلاس "تعريف" (Definition Class) لا يحتوي على كود، فقط يطبق `ICollectionFixture<T>` ويحمل الـ Attribute `[CollectionDefinition]`.
        
    2. إضافة الـ Attribute `[Collection("Name")]` فوق كلاسات الاختبار التي تريد مشاركة الـ Context معها.
        
- **النتيجة:** سيتم التعامل مع كل الكلاسات التي تحمل نفس اسم الـ Collection كمجموعة واحدة، تشترك في نفس الـ Instance من الـ Fixture، ولن تعمل بالتوازي (ستعمل بالتسلسل لضمان سلامة الـ Shared Resource).
    

---

## أمثلة الكود (Code Snippets)

### 1. Class Fixture (مشاركة داخل كلاس واحد)



```csharp
// 1. The Fixture: الكلاس الذي يحتوي على البيانات المشتركة
public class DatabaseFixture : IDisposable
{
    public Guid Id { get; } = Guid.NewGuid();
    
    public DatabaseFixture()
    {
        // Expensive setup here (e.g., DB Connection)
        Console.WriteLine("Fixture Created (Runs Once)");
    }

    public void Dispose()
    {
        // Cleanup here
        Console.WriteLine("Fixture Disposed (Runs Once)");
    }
}

// 2. The Test Class: يستخدم الـ Fixture
public class UserTests : IClassFixture<DatabaseFixture>
{
    private readonly DatabaseFixture _fixture;

    // يتم حقن الـ Fixture هنا
    public UserTests(DatabaseFixture fixture)
    {
        _fixture = fixture;
        Console.WriteLine($"Test Constructor (Runs per Test). Fixture ID: {_fixture.Id}");
    }

    [Fact]
    public void Test1() { /*...*/ }

    [Fact]
    public void Test2() { /*...*/ }
}
```

### 2. Collection Fixture (مشاركة عبر عدة كلاسات)



```csharp
// 1. The Fixture Logic (نفس الكلاس السابق)
public class DatabaseFixture { /* ... */ }

// 2. The Collection Definition (Marker Class)
[CollectionDefinition("Database Collection")] // الاسم المعرف للمجموعة
public class DatabaseCollection : ICollectionFixture<DatabaseFixture>
{
    // هذا الكلاس يبقى فارغاً، وظيفته فقط التعريف
}

// 3. Test Class A
[Collection("Database Collection")] // الربط بالاسم
public class UserTestsA
{
    public UserTestsA(DatabaseFixture fixture) { /* نفس الـ Instance */ }
}

// 4. Test Class B
[Collection("Database Collection")] // الربط بنفس الاسم
public class UserTestsB
{
    public UserTestsB(DatabaseFixture fixture) { /* نفس الـ Instance */ }
}
```

---

## لماذا؟ (The "Why")

- **لماذا يعيد xUnit إنشاء الكلاس لكل اختبار؟**
    
    - لضمان **Test Isolation**. تخيل لو عدّل "اختبار 1" على متغير في الكلاس، واعتمد "اختبار 2" على القيمة القديمة. ستحصل على نتائج عشوائية (Flaky Tests). إعادة الإنشاء تضمن صفحة بيضاء لكل اختبار.
        
- **لماذا نستخدم Fixtures؟**
    
    - **للأداء (Performance):** تشغيل قاعدة بيانات (أو Docker Container) قد يستغرق 5 ثوانٍ. إذا كان لديك 100 اختبار، لا يمكنك الانتظار 500 ثانية. باستخدام Fixture، ننشئها مرة واحدة (5 ثوانٍ) ونعيد استخدامها 100 مرة (0 ثانية)، مما يوفر وقتاً هائلاً.
        

---

> [!warning] تنبيه هام: التوازي (Parallelism)
> 
> عندما تستخدم `[Collection]`، فإنك تخبر xUnit أن هذه الكلاسات مترابطة. لذلك، **سيقوم xUnit بإيقاف التوازي بينها** وتشغيلها بالتسلسل (Sequentially) لضمان عدم حدوث تضارب في استخدام الموارد المشتركة (Thread Safety). هذا هو الثمن الذي تدفعه مقابل مشاركة الـ Context.

> [!tip] نصيحة
> 
> استخدم `Class Fixture` عندما يكون الـ Setup ثقيلاً ولكن خاصاً بمجموعة اختبارات مترابطة منطقياً.
> 
> استخدم `Collection Fixture` فقط للـ Resources الثقيلة جداً (مثل قاعدة بيانات حقيقية في Integration Tests) التي يجب أن تبقى حية طوال مدة تشغيل الـ Test Suite.



---

## المفهوم الجوهري (Core Concept)

يوفر **xUnit** آليات متقدمة للتحكم في كيفية تشغيل الاختبارات (توازي vs تسلسل) عبر الـ **Collections**، ويقدم أدوات قوية لتزويد الاختبارات بالبيانات الديناميكية المعقدة (مثل الكائنات والملفات) باستخدام `[MemberData]` و `[ClassData]` بدلاً من الـ `[InlineData]` المحدودة.

---

## الشرح التفصيلي (Detailed Breakdown)

### 1. التحكم في التوازي (Advanced Parallelization)

- **القاعدة الذهبية:**
    
    - الاختبارات داخل نفس الـ Collection (نفس الكلاس أو المشتركة في `[Collection]`) $\rightarrow$ **تسلسلية (Sequential)**.
        
    - الـ Collections المختلفة $\rightarrow$ **متوازية (Parallel)**.
        
- **تخصيص السلوك (Customization):**
    
    - يمكن تغيير السلوك الافتراضي لجعله "Collection لكل Assembly" (إيقاف التوازي تماماً).
        
    - يمكن التحكم في عدد الـ Threads (`MaxParallelThreads`).
        
    - يمكن تعطيل التوازي لمجموعة معينة باستخدام `[CollectionDefinition(DisableParallelization = true)]`.
        
- **متى نستخدم هذا؟** نادراً ما تحتاج لتغيير الإعدادات الافتراضية، إلا إذا كان لديك Shared Resource حساس جداً (مثل قاعدة بيانات واحدة لا تقبل اتصالات متزامنة).
    

### 2. البيانات الديناميكية (Dynamic Test Data)

الـ `[InlineData]` ممتازة للأرقام والنصوص البسيطة، لكن ماذا لو أردت تمرير Object كامل أو قراءة بيانات من ملف JSON؟

#### أ) `[MemberData]`

- **الفكرة:** دالة `static` داخل كلاس الاختبار تعيد `IEnumerable<object[]>`.
    
- **الميزة:** تسمح لك بكتابة كود C# لإنشاء البيانات (Loops, Logic).
    
- **الاستخدام:** عندما تكون البيانات مرتبطة بالكلاس نفسه وليست ضخمة جداً.
    

#### ب) `[ClassData]`

- **الفكرة:** كلاس منفصل تماماً يطبق `IEnumerable<object[]>`.
    
- **الميزة:** فصل البيانات عن الاختبار (Separation of Concerns). الكلاس يصبح "مزود بيانات" متخصص.
    
- **الاستخدام:** عندما تكون البيانات معقدة، كثيرة، أو يتم مشاركتها بين عدة كلاسات اختبار.
    

### 3. التحكم في الزمن (Timeouts)

- **المشكلة:** أحياناً يعلق الاختبار في حلقة لا نهائية (Infinite Loop) أو عملية بطيئة جداً.
    
- **الحل:** خاصية `Timeout` في الـ `[Fact]`.
    
- **النتيجة:** إذا تجاوز الاختبار الزمن المحدد، يفشل فوراً بدلاً من تجميد الـ CI/CD Pipeline.
    

---

## أمثلة الكود (Code Snippets)

### 1. استخدام `[MemberData]`



```csharp
public class CalculatorTests
{
    // مصدر البيانات (يجب أن يكون Static)
    public static IEnumerable<object[]> GetAddData()
    {
        yield return new object[] { 5, 5, 10 };
        yield return new object[] { -5, 5, 0 };
        // يمكن إضافة منطق معقد هنا
    }

    [Theory]
    [MemberData(nameof(GetAddData))] // الإشارة لمصدر البيانات
    public void Add_ShouldCalculateSum(int a, int b, int expected)
    {
        var result = new Calculator().Add(a, b);
        Assert.Equal(expected, result);
    }
}
```

### 2. استخدام `[ClassData]`



```csharp
// 1. كلاس البيانات المنفصل
public class SubtractTestData : IEnumerable<object[]>
{
    public IEnumerator<object[]> GetEnumerator()
    {
        yield return new object[] { 5, 3, 2 };
        yield return new object[] { 10, 10, 0 };
    }
    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();
}

// 2. كلاس الاختبار
public class CalculatorTests
{
    [Theory]
    [ClassData(typeof(SubtractTestData))] // الإشارة للكلاس
    public void Subtract_ShouldCalculateDiff(int a, int b, int expected)
    {
         // ...
    }
}
```

### 3. استخدام `Timeout`



```csharp
[Fact(Timeout = 2000)] // زمن أقصى 2000 ملي ثانية (ثانيتين)
public async Task SlowTest_ShouldFailIfTooSlow()
{
    // هذا الاختبار سيفشل لأن العملية تستغرق 5 ثوانٍ
    await Task.Delay(5000); 
}
```

---

## لماذا؟ (The "Why")

- **لماذا `MemberData` تعيد `object[]`؟**
    
    - لأن الـ Method Signature للاختبار (`void Test(int a, int b)`) تستقبل عدة باراميترات. الـ `object[]` يمثل قائمة هذه الباراميترات لاختبار واحد.
        
- **لماذا نفصل البيانات في `ClassData`؟**
    
    - للحفاظ على نظافة كلاس الاختبار (Clean Code). إذا كان لديك 50 سيناريو اختبار، وضعها داخل ملف الـ Test سيجعله غير مقروء.
        
- **لماذا نستخدم `Timeout`؟**
    
    - لحماية الـ **Build Pipeline**. لا تريد أن يتعطل الـ Server لمدة ساعة بسبب اختبار واحد دخل في Deadlock.
        

---

> [!tip] نصيحة: متى تستخدم ماذا؟
> 
> - قيم بسيطة وثابتة $\rightarrow$ **InlineData**
>     
> - بيانات معقدة قليلة $\rightarrow$ **MemberData**
>     
> - بيانات ضخمة أو مشتركة أو تحتاج لملفات خارجية $\rightarrow$ **ClassData**
>     

> [!warning] تنبيه
> 
> عند استخدام `MemberData`، تأكد أن الدالة `public static`. نسيان كلمة `static` هو الخطأ الأكثر شيوعاً وسيمنع xUnit من اكتشاف الاختبارات.


---

## المفهوم الجوهري (Core Concept)

لجعل الكود الذي يعتمد على عوامل متغيرة (مثل **الزمن** `DateTime.Now`) قابلاً للاختبار بشكل حتمي (Deterministic)، يجب علينا عزله خلف **Provider Interface**. كما أن أدوات مثل **Continuous Testing** و **Code Coverage** تحول عملية الاختبار من "مهمة يدوية" إلى "رادار دائم" يكشف الثغرات ويضمن سلامة الكود لحظياً.

---

## الشرح التفصيلي (Detailed Breakdown)

### 1. معضلة اختبار الزمن (The Time Problem)

- **المشكلة:** لدينا كلاس `Greeter` يقول "Good Morning" إذا كانت الساعة قبل الـ 12 ظهراً.
    
    - إذا كتبت الاختبار صباحاً $\leftarrow$ سينجح (Pass).
        
    - إذا شغلت نفس الاختبار مساءً $\leftarrow$ سيفشل (Fail).
        
- **السبب:** الكود يعتمد مباشرة على `DateTime.Now` (System Clock)، وهو متغير لا نملك السيطرة عليه. هذا يخلق ما يسمى بـ **Flaky Tests** (اختبارات غير مستقرة).
    
- **الحل:** تطبيق مبدأ **Inversion of Control**:
    
    1. إنشاء `interface IDateTimeProvider` تحتوي على خاصية ترجع الوقت.
        
    2. التطبيق الحقيقي يرجع `DateTime.Now`.
        
    3. في الاختبار، نستخدم **Mock** يرجع وقتاً ثابتاً (مثلاً: دائماً الساعة 10 صباحاً).
        

### 2. الاختبار المستمر (Continuous Testing)

- **الفكرة:** ميزة في الـ IDEs المتقدمة (مثل Rider, VS Enterprise, NCrunch) تقوم بتشغيل الاختبارات تلقائياً بمجرد ضغط زر **Save**.
    
- **الفائدة:** تغذية راجعة فورية (Instant Feedback Loop). تكتشف الخطأ في نفس اللحظة التي تكتبه فيها، بدلاً من انتظاره حتى نهاية اليوم.
    
- **العيوب:** قد تستهلك موارد الجهاز (CPU/RAM) إذا كان المشروع ضخماً.
    

### 3. تغطية الكود (Code Coverage)

- **التعريف:** مقياس بنسبة مئوية يخبرك: "كم سطراً من الكود المصدري تم تنفيذه أثناء تشغيل الاختبارات؟".
    
- **المؤشرات البصرية:**
    
    - **أخضر (Green):** هذا السطر تم اختباره (Covered).
        
    - **أحمر/رمادي (Red/Gray):** هذا السطر لم يمر عليه أي اختبار (Uncovered Gap).
        
- **الهدف:** كشف المناطق المنسية في الـ Logic (مثلاً جملة `else` لم نختبرها، أو `catch` block لم يتم تحفيزه).
    

---

## أمثلة الكود (Code Snippets)

### 1. الحل: عزل الزمن (The Time Provider Pattern)



```csharp
// 1. The Interface (The Contract)
public interface IDateTimeProvider
{
    DateTime Now { get; }
}

// 2. The Implementation (Production Code)
public class DateTimeProvider : IDateTimeProvider
{
    // في الواقع، نستخدم ساعة النظام
    public DateTime Now => DateTime.Now; 
}

// 3. The Refactored Class (Testable)
public class Greeter
{
    private readonly IDateTimeProvider _dateTimeProvider;

    // نحقن الـ Provider بدلاً من استخدام DateTime.Now مباشرة
    public Greeter(IDateTimeProvider dateTimeProvider)
    {
        _dateTimeProvider = dateTimeProvider;
    }

    public string GenerateMessage()
    {
        var now = _dateTimeProvider.Now; // نأخذ الوقت من الـ Provider

        if (now.Hour >= 5 && now.Hour < 12) return "Good morning";
        if (now.Hour >= 12 && now.Hour < 18) return "Good afternoon";
        return "Good evening";
    }
}
```

### 2. الاختبار (The Test with Mocked Time)



```csharp
[Fact]
public void GenerateMessage_ShouldSayGoodMorning_WhenItIsMorning()
{
    // Arrange
    var dateTimeProvider = Substitute.For<IDateTimeProvider>();
    
    // نثبت الوقت ليكون دائماً 10 صباحاً، مهما كان الوقت الحقيقي
    dateTimeProvider.Now.Returns(new DateTime(2020, 1, 1, 10, 0, 0));

    var sut = new Greeter(dateTimeProvider);

    // Act
    var result = sut.GenerateMessage();

    // Assert
    // الآن هذا الاختبار سينجح دائماً سواء شغلته فجراً أو ليلاً
    result.Should().Be("Good morning");
}
```

---

## لماذا؟ (The "Why")

- **لماذا نعزل الزمن؟**
    
    - لضمان **الحتمية (Determinism)**. الـ Unit Test يجب أن يعطي نفس النتيجة 100% من المرات، بغض النظر عن توقيت تشغيله، السنة الكبيسة، أو المنطقة الزمنية للسيرفر.
        
- **لماذا نهتم بالـ Code Coverage؟**
    
    - ليس للوصول لرقم 100% (وهو هدف غير واقعي غالباً)، بل لمعرفة **ما لم يتم اختباره**. إذا كانت الـ Service Layer مغطاة بنسبة 50% فقط، فهذا مؤشر خطر يعني أن نصف المنطق البرمجي قد يحتوي على أخطاء (Bugs) غير مكتشفة.
        

---

> [!tip] نصيحة: Infrastructure Wrappers
> 
> نمط الـ `IDateTimeProvider` (وأحياناً `IGuidProvider`) لا يحل مشكلة الاختبار فحسب، بل يسهل أيضاً التعامل مع سيناريوهات معقدة مثل محاكاة "مرور الزمن" أو التعامل مع المناطق الزمنية (Timezones) المختلفة في الأنظمة الموزعة.

> [!warning] تنبيه حول Code Coverage
> 
> الوصول لنسبة تغطية 100% لا يعني أن الكود خالي من الأخطاء، بل يعني فقط أن كل سطر تم تشغيله مرة واحدة على الأقل. الجودة تعتمد على قوة الـ **Assertions** وليس فقط على عدد السطور التي تم المرور عليها.



---

## المفهوم الجوهري (Core Concept)

لا يكفي تشغيل الاختبارات داخل الـ IDE؛ للحصول على **Continuous Integration** حقيقي، يجب أن نكون قادرين على قياس **Code Coverage** عبر سطر الأوامر (CLI) باستخدام أدوات مثل **Coverlet**، وتحويل النتائج الخام (XML) إلى تقارير HTML تفاعلية مرئية باستخدام **ReportGenerator** ليراها الفريق بأكمله.

---

## الشرح التفصيلي (Detailed Breakdown)

### 1. قياس التغطية عبر سطر الأوامر (Coverlet CLI)

بينما يوفر Rider أو Visual Studio أدوات تغطية مدمجة، إلا أنها لا تعمل على سيرفرات الـ Linux في الـ GitHub Actions أو Azure DevOps. الحل هو استخدام أداة **Coverlet**.

- **التثبيت:** يتم تثبيتها كـ Global Tool (`dotnet tool install -g coverlet.console`).
    
- **الاستخدام:** تقوم هذه الأداة "بتغليف" عملية التشغيل، فتطلب منك مسار ملف الـ DLL الخاص بالاختبارات، وتقوم هي بتشغيله وحساب النسبة.
    
- **الاستبعاد (Exclusion):**
    
    - **الطريقة 1 (Attributes):** وضع `[ExcludeFromCodeCoverage]` فوق الكلاسات التي لا نريد قياسها (مثل Repositories).
        
    - **الطريقة 2 (CLI Args):** تمرير باراميتر `--exclude` للأمر، وهذا أفضل لأنه يبقي الكود نظيفاً.
        

### 2. توليد ملفات التقارير (XPlat Code Coverage)

بدلاً من مجرد طباعة النسبة في الشاشة، نريد ملفاً يمكن حفظه وأرشفته.

- **الأداة:** `coverlet.collector` (تأتي مثبتة غالباً مع قوالب xUnit الحديثة).
    
- **الأمر:** `dotnet test --collect:"XPlat Code Coverage"`.
    
- **الناتج:** ملف `coverage.cobertura.xml` يحتوي على تفاصيل دقيقة لكل سطر تم تنفيذه.
    

### 3. تحويل التقرير إلى HTML (ReportGenerator)

ملفات الـ XML غير مقروءة للبشر. هنا يأتي دور أداة **ReportGenerator**.

- **الوظيفة:** تأخذ ملف الـ XML وتحوله لموقع ويب كامل (HTML).
    
- **المميزات:**
    
    - تظهر لك الكود المصدري ملوناً (أخضر للمغطى، أحمر لغير المغطى).
        
    - تعطيك إحصائيات دقيقة للكلاسات والفروع (Branch Coverage).
        
    - يمكن رفع هذا المجلد كـ Artifact في الـ CI/CD Pipeline.
        

---

## أمثلة الكود (Code Snippets)

### 1. التثبيت والتشغيل عبر الـ CLI (للحصول على نسبة مئوية)



```Bash
# 1. تثبيت الأداة
dotnet tool install -g coverlet.console

# 2. بناء المشروع للحصول على الـ DLL
dotnet build

# 3. تشغيل التغطية (مع استثناء الـ Repositories)
coverlet ./bin/Debug/net6.0/Users.Api.Tests.Unit.dll \
    --target "dotnet" \
    --targetargs "test --no-build" \
    --exclude "[Users.Api]Users.Repositories.*"
```

### 2. توليد التقرير المرئي (HTML Report)



```Bash
# 1. تشغيل الاختبارات وتوليد ملف XML
dotnet test --collect:"XPlat Code Coverage"

# 2. تثبيت أداة توليد التقارير
dotnet tool install -g dotnet-reportgenerator-globaltool

# 3. تحويل الـ XML إلى HTML
reportgenerator \
    -reports:"./TestResults/*/coverage.cobertura.xml" \
    -targetdir:"./CodeCoverageReport" \
    -reporttypes:Html
```

---

## لماذا؟ (The "Why")

- **لماذا الـ CLI؟**
    
    - لأن سيرفرات الـ CI (مثل Jenkins, GitHub Actions) ليس لديها واجهة رسومية (GUI). يجب أن تكون قادراً على تشغيل كل شيء عبر أوامر نصية.
        
- **لماذا نستثني الـ Repositories؟**
    
    - لأنها تحتوي على كود "توصيل" (Boilerplate) فقط، وشمولها في التقرير سيخفض النسبة العامة للتغطية ظلماً، مما قد يحبط الفريق أو يعطي إنذارات كاذبة.
        
- **لماذا HTML؟**
    
    - لأنه يسهل على المدير التقني أو الزملاء مراجعة الجودة بلمحة سريعة دون الحاجة لتثبيت وتشغيل المشروع بأنفسهم.
        

---

> [!tip] نصيحة: Open Report
> 
> بعد توليد تقرير الـ HTML، يمكنك ببساطة فتح ملف `index.html` الموجود داخل المجلد الناتج في أي متصفح لترى لوحة تحكم كاملة بجودة الكود الخاص بك.

> [!warning] تنبيه
> 
> تأكد من وجود الباكيج `coverlet.collector` في ملف `.csproj` الخاص بمشروع التست، وإلا فلن يعمل الأمر `--collect`.

---


