

---

## المفهوم الجوهري (Core Concept)

البدء الصحيح في الـ Unit Testing يتطلب ثلاثة ركائز: اختيار الـ **Tech Stack** المناسب (Runner, Mocker, Asserter)، وتنظيم هيكلية المشروع بملفات منفصلة (`src` vs `test`)، والالتزام الصارم بنمط الـ **AAA Pattern** وتسميات وصفية تجعل من الاختبارات توثيقاً حياً للنظام.

---

## الشرح التفصيلي (Detailed Breakdown)

### 1. الركائز الثلاث للاختبار (The Three Pillars)

أي بيئة Unit Testing متكاملة تحتاج إلى ثلاث مكتبات أساسية لتعمل:

1. **Unit Testing Framework (The Runner):** المسؤول عن تشغيل الاختبارات.
    
    - أشهر اللاعبين: `XUnit`, `NUnit`, `MSTest`.
        
    - _اختيار الكورس:_ **XUnit**.
        
2. **Mocking Library:** المسؤولة عن تزييف الـ Dependencies.
    
    - أشهر اللاعبين: `Moq`, `NSubstitute`, `FakeItEasy`.
        
    - _اختيار الكورس:_ **NSubstitute**.
        
3. **Assertion Library:** المسؤولة عن التحقق من النتائج بطريقة مقروءة.
    
    - أشهر اللاعبين: `FluentAssertions`, `Shouldly` (بالإضافة للـ Built-in assertions).
        
    - _اختيار الكورس:_ **FluentAssertions**.
        

### 2. إعداد المشروع (Project Setup & XUnit)

عند إنشاء مشروع اختبار باستخدام XUnit، نلاحظ في ملف `.csproj` الاعتماديات التالية:

- `Microsoft.NET.Test.Sdk`: حزمة التطوير الأساسية للاختبارات.
    
- `xunit`: المكتبة التي نكتب بها الكود (مثل الـ Attributes).
    
- `xunit.runner.visualstudio`: المشغل الذي يسمح للـ IDE (مثل VS أو Rider) باكتشاف وتشغيل الاختبارات.
    

**السمة المميزة (Attribute):**

- لتحويل أي Method عادية إلى Unit Test، يجب وضع الـ Attribute المسمى `[Fact]` فوقها. بدونه، لن يكتشف الـ Test Runner هذه الدالة.
    

### 3. الهيكلية الفيزيائية للمشروع (Physical Structure)

لا يجب وضع ملفات الاختبار بجانب ملفات الكود المصدري بشكل عشوائي. الهيكلية الاحترافية تتطلب فصلاً جذرياً:

- إنشاء مجلد **`src`**: يوضع بداخله المشاريع الأساسية (Production Code).
    
- إنشاء مجلد **`test`**: يوضع بداخله مشاريع الاختبارات.
    
- يجب تنفيذ هذا الفصل على مستوى نظام الملفات (File System) أولاً، ثم عكسه داخل الـ Solution Folders في الـ IDE.
    

### 4. استراتيجية التسمية (Naming Convention)

التسمية هي ما يحول الاختبار إلى توثيق (Documentation). القواعد كالتالي:

- **Project Name:** `[ProjectToTest].Tests.[Type]`
    
    - مثال: `CalculatorLibrary.Tests.Unit`
        
- **Class Name:** `[ClassToTest]Tests`
    
    - مثال: `CalculatorTests` (لأننا نختبر كلاس الـ Calculator).
        
- **Method Name:** `[MethodName]_Should[ExpectedBehavior]_When[State]`
    
    - مثال: `Add_ShouldAddTwoNumbers_WhenTwoNumbersAreIntegers`
        
    - _الهدف:_ قراءة اسم الدالة يخبرك فوراً ماذا تفعل، ماذا تتوقع، وما هي الحالة، دون النظر للكود.
        

### 5. نمط الـ AAA Pattern

هو الهيكل القياسي لأي Unit Test، ويقسم الكود لثلاث مراحل زمنية:

1. **Arrange:** تهيئة المتغيرات، إنشاء الـ Objects، وتجهيز الـ Mocks.
    
2. **Act:** استدعاء الدالة المراد اختبارها (The actual execution).
    
3. **Assert:** التحقق من أن النتيجة تطابق التوقع.
    

---

## أمثلة الكود (Code Snippets)

إليك مثال شامل يطبق الهيكلية، التسمية، ونمط الـ AAA كما ورد في الشرح:



```csharp
using XUnit; // استدعاء المكتبة
using CalculatorLibrary; // استدعاء المشروع المراد اختباره

namespace CalculatorLibrary.Tests.Unit
{
    // 1. Class Naming: [ClassToTest]Tests
    public class CalculatorTests
    {
        // 2. Method Naming: [Method]_Should[Result]_When[Condition]
        // 3. Attribute: [Fact] is essential for discovery
        [Fact]
        public void Add_ShouldAddTwoNumbers_WhenTwoNumbersAreIntegers()
        {
            // --- Arrange ---
            // تجهيز الكلاس الذي نريد اختباره (SUT - System Under Test)
            var calculator = new Calculator();
            int num1 = 5;
            int num2 = 4;

            // --- Act ---
            // تنفيذ العملية الفعلية
            var result = calculator.Add(num1, num2);

            // --- Assert ---
            // التحقق من النتيجة (هنا نستخدم الـ Built-in Assert مؤقتاً)
            Assert.Equal(9, result); 
        }
    }
}
```

---

## لماذا؟ (The "Why")

- **لماذا XUnit تحديداً؟**
    
    - لأنها صممت من قبل مطوري NUnit الأصليين لتلافي أخطاء التصميم القديمة. وهي المكتبة المعتمدة رسمياً من Microsoft وتستخدم في مشاريعها (مثل .NET Core نفسه).
        
- **لماذا نفصل المجلدات (`src` vs `test`)؟**
    
    - للحفاظ على نظافة المشروع (Clean Solution). مع زيادة عدد المشاريع، وجود الاختبارات مختلطة مع الكود الأصلي يسبب فوضى بصرية وصعوبة في إدارة الاعتماديات.
        
- **لماذا نكتب تعليقات `// Arrange`, `// Act`, `// Assert`؟**
    
    - للفصل الذهني والبصري. في الاختبارات المعقدة، قد تتداخل السطور، ووجود هذه التعليقات يمنعك من ارتكاب أخطاء مثل وضع منطق الـ Assertion داخل مرحلة الـ Act.
        

---

> [!tip] قاعدة ذهبية في التسمية
> 
> التسمية الجيدة للاختبار تغنيك عن قراءة الكود. يجب أن يكون اسم الـ Test Method عبارة عن "جملة إنجليزية مفيدة" تصف السيناريو بالكامل.
> 
> - ✅ `Add_ShouldReturnSum_WhenInputsAreValid`
>     
> - ❌ `TestAdd` (اسم سيء لا يوضح السيناريو).
>     

> [!warning] تنبيه
> 
> عند نقل المشاريع يدوياً إلى مجلدات `src` و `test`، قد يفقد الـ Solution المسار الصحيح للمشاريع. ستحتاج إلى إزالتها (Remove) من الـ Solution ثم إضافتها مجدداً (Add Existing Project) من الموقع الجديد، وقد تحتاج لتعديل الـ Project References داخل ملفات الـ `.csproj`.



---

## المفهوم الجوهري (Core Concept)

يعتمد **XUnit** على نموذج عزل صارم يقوم بإنشاء **Instance** جديد من كلاس الاختبار لكل **Test Method** على حدة لضمان عدم تداخل البيانات (State Isolation)، ويوفر أدوات قوية لإدارة دورة حياة الاختبار (Setup/Teardown) وتقليل تكرار الكود عبر الـ **Data-Driven Testing** باستخدام الـ `[Theory]`.

---

## الشرح التفصيلي (Detailed Breakdown)

### 1. نموذج التنفيذ (Execution Model & Context)

- **قاعدة العزل التام:**
    
    - في XUnit، لا يتم استخدام نفس الـ Instance من كلاس الاختبار لتشغيل جميع الـ Methods بداخله.
        
    - **السلوك:** لكل `[Fact]` أو `[Theory]`، يقوم XUnit بإنشاء `new CalculatorTests()`، يشغل الاختبار، ثم يتخلص منه.
        
    - **النتيجة:** المتغيرات (Fields) المعرفة داخل الكلاس يتم إعادة تعيينها (Reset) مع كل اختبار، مما يضمن أن اختباراً واحداً لا يمكنه التأثير على نتائج اختبار آخر (No Shared State).
        

### 2. مصطلحات هامة: SUT

- **SUT (System Under Test):** هو مصطلح قياسي يستخدم لتسمية الكائن (Object) الذي نقوم باختباره حالياً.
    
- يفضل تعريف متغير `private readonly` باسم `_sut` ليشار بوضوح إلى الهدف من الاختبار.
    

### 3. دورة حياة الاختبار (Setup & Teardown)

كيف ننفذ كوداً قبل كل اختبار (Setup) وبعد كل اختبار (Cleanup)؟

#### أ) الطريقة المتزامنة (Synchronous):

- **الـ Setup:** يتم وضعه داخل الـ **Constructor**.
    
    - أي كود هنا سيعمل _قبل_ تشغيل الـ Test Method.
        
- **الـ Cleanup:** يتم عن طريق تطبيق واجهة `IDisposable`.
    
    - نضع كود التنظيف داخل دالة `Dispose()`. سيعمل هذا الكود _بعد_ انتهاء الاختبار.
        

#### ب) الطريقة غير المتزامنة (Asynchronous):

- إذا كان كود التهيئة يحتاج `await` (مثل تهيئة قاعدة بيانات في الذاكرة)، نستخدم واجهة `IAsyncLifetime`.
    
- **InitializeAsync:** تعمل بدلاً من الـ Constructor للعمليات الـ Async.
    
- **DisposeAsync:** تعمل للتنظيف بشكل Async.
    

**تسلسل التنفيذ (Order of Execution):**

1. `Constructor`
    
2. `InitializeAsync` (if implemented)
    
3. `The Test Method` ([Fact] or [Theory])
    
4. `DisposeAsync` (if implemented)
    
5. `Dispose` (if implemented)
    

### 4. الاختبارات المبنية على البيانات (Parameterization)

بدلاً من كتابة اختبار منفصل لكل حالة (جمع 1+1، جمع 5+5، جمع 0+0)، نستخدم الـ **Theories**.

- **[Fact]:** اختبار ثابت لا يقبل مدخلات (Parameters).
    
- **[Theory]:** اختبار يقبل مدخلات، ويسمح بتكرار نفس المنطق مع بيانات مختلفة.
    
- **[InlineData]:** الـ Attribute الذي نمرر من خلاله البيانات.
    

### 5. تخطي الاختبارات (Skipping Tests)

- يمكن تجاهل اختبار معين (لأنه مكسور حالياً أو لا يعمل في بيئة الـ CI) باستخدام خاصية `Skip`.
    
- يمكن وضعها على مستوى الـ `[Fact]` أو على مستوى `[InlineData]` محدد.
    

---

## أمثلة الكود (Code Snippets)

### مثال 1: دورة الحياة (Setup/Teardown)



```csharp
public class LifeCycleTests : IDisposable, IAsyncLifetime
{
    private readonly ITestOutputHelper _output;

    // 1. Setup (Sync) - Runs First
    public LifeCycleTests(ITestOutputHelper output)
    {
        _output = output;
        _output.WriteLine("1. Constructor: Setup completed.");
    }

    // 2. Setup (Async) - Runs Second
    public async Task InitializeAsync()
    {
        await Task.Delay(1); // Simulate async work
        _output.WriteLine("2. InitializeAsync: Async Setup completed.");
    }

    [Fact]
    public async Task MyTest()
    {
        _output.WriteLine("3. Test Execution: Logic runs here.");
    }

    // 4. Teardown (Async) - Runs Fourth
    public async Task DisposeAsync()
    {
        await Task.Delay(1);
        _output.WriteLine("4. DisposeAsync: Async Cleanup completed.");
    }

    // 5. Teardown (Sync) - Runs Last
    public void Dispose()
    {
        _output.WriteLine("5. Dispose: Final Cleanup.");
    }
}
```

### مثال 2: الـ Theories والـ InlineData



```csharp
public class CalculatorTests
{
    private readonly Calculator _sut = new Calculator(); // SUT: System Under Test

    // تغيير Fact إلى Theory يسمح لنا باستقبال Parameters
    [Theory]
    [InlineData(5, 4, 9)]       // Scenario 1: Positive numbers
    [InlineData(0, 0, 0)]       // Scenario 2: Zeros
    [InlineData(-5, -5, -10)]   // Scenario 3: Negative numbers
    [InlineData(10, 5, 15, Skip = "Waiting for fix")] // Skipped Scenario
    public void Add_ShouldAddTwoNumbers_WhenNumbersAreIntegers(int a, int b, int expected)
    {
        // Act
        var result = _sut.Add(a, b);

        // Assert
        result.Should().Be(expected); // FluentAssertions syntax
    }
}
```

---

## لماذا؟ (The "Why")

- **لماذا ينشئ XUnit نسخة جديدة لكل اختبار؟**
    
    - لضمان **Test Isolation**. لو كان الكلاس واحداً (Singleton) وتشارك الاختبارات نفس المتغيرات، فإن تغيير قيمة متغير في "اختبار 1" قد يتسبب في فشل "اختبار 2" بشكل عشوائي (Flaky Tests). هذا النموذج يمنع ذلك تماماً.
        
- **لماذا نستخدم `[Theory]` بدلاً من تكرار الـ `[Fact]`؟**
    
    - اتباعاً لمبدأ **DRY (Don't Repeat Yourself)**. لو اكتشفت خطأ في منطق الاختبار، ستصلحه في مكان واحد بدلاً من تعديله في 10 دوال مختلفة.
        
- **لماذا نستخدم `IAsyncLifetime`؟**
    
    - لأن الـ Constructor في C# لا يقبل أن يكون `async`. إذا احتجت لانتظار عملية (مثل `await database.EnsureCreatedAsync()`)، فلا مفر من استخدام هذه الواجهة.
        

---

> [!tip] نصيحة: SUT Naming
> 
> اعتيادك على تسمية المتغير الرئيسي بـ `_sut` يجعل قراءة الاختبار أسرع بكثير لأي شخص آخر. سيعرفون فوراً أن هذا هو الكائن الذي يتم فحصه، والباقي هم مجرد Helpers أو Mocks.

> [!warning] تحذير: Static State
> 
> على الرغم من أن XUnit يعزل الـ Instance Fields، إلا أنه **لا يعزل** الـ **Static Fields**. إذا قمت بتعديل متغير `static` داخل اختبار، سيتأثر كل الاختبارات الأخرى التي تعمل بالتوازي. تجنب الـ Static State في اختباراتك قدر الإمكان.

---

