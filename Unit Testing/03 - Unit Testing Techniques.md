

---

## المفهوم الجوهري (Core Concept)

الانتقال من استخدام `Assert Class` التقليدية والجافة إلى مكتبة **Fluent Assertions**، التي تحول كود الاختبار إلى "جمل إنجليزية مفهومة" (Human-Readable)، وتوفر رسائل خطأ دقيقة جداً تساعد في تشخيص المشاكل فوراً دون الحاجة للـ Debugging.

---

## الشرح التفصيلي (Detailed Breakdown)

### 1. ما هي الـ Assertions؟

- هي عملية مقارنة بين **ما نتوقعه** (Expected) وبين **ما يعيده الكود فعلياً** (Actual).
    
- إذا تطابقت القيمتان $\leftarrow$ ينجح الاختبار (Pass).
    
- إذا اختلفتا $\leftarrow$ يرمي الكود `Exception` ويفشل الاختبار (Fail).
    

### 2. أدوات الـ Assertion (The Players)

- **XUnit Assert Class:** تأتي مدمجة مع XUnit (مثل `Assert.Equal`).
    
    - _رأي المحاضر:_ جيدة لكنها ليست "أنيقة" (Not Elegant) وقراءتها صعبة قليلاً.
        
- **Fluent Assertions & Shouldly:** مكتبات خارجية (NuGet Packages).
    
    - _الاختيار:_ **Fluent Assertions**.
        
    - _السبب:_ هي الأكثر شهرة، الأفضل قراءة، وتعطي رسائل خطأ ممتازة.
        

### 3. قوة الـ Fluent Assertions

الميزة الكبرى لهذه المكتبة هي أنها **Content Aware** (مدركة لنوع البيانات). بمجرد كتابة `.Should()`، تظهر لك دوال مساعدة خاصة بنوع المتغير (String, Int, Date...).

#### أ) التعامل مع النصوص (Strings)

بدلاً من مجرد التحقق من المساواة، يمكنك التحقق من تفاصيل النص:

- **Equality:** هل يساوي "Nick Chapsas"؟
    
- **Emptiness:** هل هو فارغ أم لا؟ (`NotBeEmpty`).
    
- **Content:** هل يبدأ بـ "Nick"؟ هل ينتهي بـ "Chapsas"؟
    

#### ب) التعامل مع الأرقام (Numbers)

التحقق من الأرقام يصبح أكثر مرونة:

- **Equality:** هل يساوي 21؟
    
- **Range:** هل هو بين 18 و 60؟ (`BeInRange`).
    
- **Signs:** هل هو موجب؟ (`BePositive`).
    
- **Comparison:** أكبر من أو يساوي؟ (`BeGreaterThan`).
    

#### ج) التعامل مع التواريخ (Dates)

- مشابه للأرقام، يمكنك التحقق إذا كان التاريخ في نطاق معين، أو بعد تاريخ محدد (`BeAfter` / `BeGreaterThan`).
    

---

## أمثلة الكود (Code Snippets)

### 1. المقارنة: الطريقة القديمة vs الطريقة الحديثة



```csharp
// 1. The Old Way (XUnit Built-in)
// صعبة القراءة: يجب أن تتذكر الترتيب (Expected أولاً أم Actual؟)
Assert.Equal(expected, result); 

// 2. The Fluent Way (FluentAssertions)
// تقرأ كجملة إنجليزية: "النتيجة يجب أن تكون المتوقع"
result.Should().Be(expected);
```

### 2. أمثلة شاملة على أنواع البيانات (Strings, Numbers, Dates)



```csharp
using FluentAssertions; // ضروري لاستخدام الدوال
using XUnit;

public class ValueSamplesTests
{
    // SUT: System Under Test (لغايات التوضيح)
    private readonly ValueSamples _sut = new ValueSamples();

    // ملاحظة: هذه ليست اختبارات حقيقية بل استعراض لقدرات المكتبة
    
    [Fact]
    public void StringAssertion_Example()
    {
        var fullName = _sut.FullName; // لنفترض القيمة: "Nick Chapsas"

        // Basic Assertion
        fullName.Should().Be("Nick Chapsas");

        // Advanced String Assertions
        fullName.Should().NotBeEmpty();          // لا يجب أن يكون فارغاً
        fullName.Should().StartWith("Nick");     // يبدأ بـ
        fullName.Should().EndWith("Chapsas");    // ينتهي بـ
    }

    [Fact]
    public void NumberAssertion_Example()
    {
        var age = _sut.Age; // لنفترض القيمة: 21

        // Basic
        age.Should().Be(21);

        // Numeric Specific Assertions (Content Aware)
        age.Should().BePositive();             // رقم موجب
        age.Should().BeGreaterThan(20);        // أكبر من
        age.Should().BeLessOrEqualTo(21);      // أصغر أو يساوي
        age.Should().BeInRange(18, 60);        // يقع ضمن نطاق
    }

    [Fact]
    public void DateAssertion_Example()
    {
        var dob = _sut.DateOfBirth; 

        // Date Specific Assertions
        dob.Should().Be(new DateOnly(2000, 1, 1));
        dob.Should().BeAfter(new DateOnly(1999, 1, 1)); // بعد تاريخ معين
        dob.Should().BeInRange(                         // ضمن فترة زمنية
            new DateOnly(1990, 1, 1), 
            new DateOnly(2010, 1, 1)
        );
    }
}
```

---

## لماذا؟ (The "Why")

- **لماذا ننتقل لـ Fluent Assertions؟**
    
    1. **وضوح الكود (Readability):** الكود يُقرأ مثل القصص أو المحادثة (Natural Language). هذا يقلل العبء الذهني عند مراجعة الاختبارات.
        
    2. **رسائل الخطأ (Error Messages):**
        
        - في `Assert.Equal`، رسالة الخطأ قد تكون غامضة.
            
        - في `FluentAssertions`، الرسالة تكون: _"Expected age to be 22 but found 21"_. تخبرك بالضبط ماذا حدث، مما يسرع عملية الإصلاح.
            
    3. **إدراك المحتوى (Context Awareness):** المكتبة "ذكية". عندما تختبر `int`، تعطيك دوال رياضية (`BePositive`). عندما تختبر `String`، تعطيك دوال نصية (`StartWith`). هذا يسهل كتابة اختبارات دقيقة.
        

---

> [!tip] نصيحة: التثبيت
> 
> لاستخدام هذه الميزات، يجب عليك الذهاب إلى **NuGet Package Manager** والبحث عن `FluentAssertions` وتثبيتها في مشروع الـ **Tests** الخاص بك.

> [!warning] تنبيه
> 
> على الرغم من كثرة الخيارات (StartWith, Range, etc..)، غالباً ما يكون `Should().Be()` هو الأكثر استخداماً بنسبة 90%، ولكن معرفة باقي الأدوات تجعل اختباراتك أكثر قوة وشمولية للحالات المعقدة.



---

## المفهوم الجوهري (Core Concept)

في عالم البرمجة كائنية التوجه (OOP)، مقارنة الكائنات (Reference Types) ببعضها هي عملية خادعة. **Fluent Assertions** تحل هذه المعضلة عبر الـ **Deep Object Graph Comparison** (مقارنة القيم الداخلية بدلاً من عناوين الذاكرة) باستخدام `BeEquivalentTo`، مما يغنيك عن كتابة عشرات الأسطر من كود التحقق اليدوي.

---

## الشرح التفصيلي (Detailed Breakdown)

### 1. معضلة مقارنة الكائنات (Object Assertion)

- **المشكلة:** عند مقارنة كائنين (مثل `User` و `ExpectedUser`)، حتى لو كانت كل البيانات بداخلهما متطابقة تماماً، فإن `Assert.Equal` أو `Should().Be()` ستفشل.
    
- **السبب:** لأن هذه دوال تقارن **References** (عنوان الذاكرة) وليس المحتوى. الكائنان يقعان في مكانين مختلفين في الذاكرة.
    
- **الحل السحري:** استخدام `Should().BeEquivalentTo(expected)`.
    
    - تقوم المكتبة بمقارنة كل خاصية (Property) بداخل الكائن ديناميكياً (Reflection) للتأكد من تطابق القيم.
        

### 2. التعامل مع القوائم (Collections & Enumerables)

عندما يكون لديك قائمة من الكائنات (List of Objects):

- **للقيم البسيطة (Value Types):** مثل `List<int>`، نستخدم `Should().Contain(5)`.
    
- **للكائنات المعقدة (Reference Types):** مثل `List<User>`، نستخدم `Should().ContainEquivalentOf(expectedUser)`.
    
    - هذا يضمن البحث داخل القائمة عن كائن يطابق "قيم" الكائن المتوقع، بغض النظر عن الـ Reference.
        
- **المرونة (Lambdas):** يمكنك البحث بشرط معين:
    
    - `users.Should().Contain(x => x.Age > 5 && x.Name.StartsWith("Nick"))`.
        

### 3. اختبار الاستثناءات (Exception Assertion)

- **التحدي:** كيف نختبر كوداً يفترض أن يفشل (يرمي Exception)؟ إذا استدعينا الدالة مباشرة، سينهار الاختبار قبل الوصول لسطر الـ Assert.
    
- **الحل:** نستخدم نمط الـ **Action Delegate**.
    
    1. نغلف استدعاء الدالة داخل `Action`.
        
    2. نستخدم `action.Should().Throw<ExceptionType>()`.
        
    3. يمكننا التحقق من رسالة الخطأ أيضاً باستخدام `.WithMessage(...)`.
        

---

## أمثلة الكود (Code Snippets)

### 1. مقارنة الكائنات (Objects)



```csharp
[Fact]
public void ObjectAssertion_Example()
{
    // Arrange
    var expectedUser = new User { Name = "Nick", Age = 21 };
    var actualUser = _sut.AppUser; // لنفترض أنه يعيد كائناً بنفس البيانات

    // ❌ طريقة خاطئة: ستقارن References وتفشل
    // actualUser.Should().Be(expectedUser); 

    // ✅ طريقة صحيحة: تقارن القيم (Values)
    actualUser.Should().BeEquivalentTo(expectedUser);
}
```

### 2. مقارنة القوائم (Collections)



```csharp
[Fact]
public void CollectionAssertion_Example()
{
    // Arrange
    var users = _sut.Users; // IEnumerable<User>

    // 1. التحقق من العدد
    users.Should().HaveCount(3);

    // 2. التحقق من الاحتواء (بحث عن كائن كامل)
    var expectedMe = new User { Name = "Nick", Age = 21 };
    users.Should().ContainEquivalentOf(expectedMe);

    // 3. التحقق بشرط (بحث ذكي)
    users.Should().Contain(u => u.FullName.StartsWith("Nick") && u.Age > 5);
}
```

### 3. اختبار الاستثناءات (Exceptions)



```csharp
[Fact]
public void ExceptionAssertion_Example()
{
    // Arrange
    var calculator = new Calculator();
    
    // Act (تغليف العملية الخطرة داخل Action)
    // لاحظ: لا ننفذ الدالة فوراً، بل نضعها داخل متغير
    Action act = () => calculator.Divide(1, 0);

    // Assert
    act.Should()
       .Throw<DivideByZeroException>()             // نوع الخطأ
       .WithMessage("Attempted to divide by zero."); // رسالة الخطأ
}
```

---

## لماذا؟ (The "Why")

- **لماذا `BeEquivalentTo` هي "Library Seller" (الميزة القاتلة)؟**
    
    - لأن البديل هو كتابة كود بهذا الشكل القبيح:
        
        
        
        ```csharp
        Assert.Equal(expected.Name, actual.Name);
        Assert.Equal(expected.Age, actual.Age);
        Assert.Equal(expected.Address.Street, actual.Address.Street);
        // ... وهكذا لكل خاصية!
        ```
        
    - `BeEquivalentTo` توفر عليك ساعات من الكتابة والصيانة، وتجعل الاختبار مرناً (حتى لو تغير ترتيب الخواص).
        
- **لماذا نستخدم `Action` مع الـ Exceptions؟**
    
    - لأن الـ Unit Test هو برنامج C# عادي. إذا حدث Exception غير معالج (Unhandled)، سيتوقف البرنامج فوراً. تغليفه بـ `Action` يسمح لمكتبة Fluent Assertions بـ "التقاط" هذا الخطأ والتحقق منه بأمان.
        

---

> [!tip] نصيحة: Deep Comparison
> 
> دالة `BeEquivalentTo` قوية جداً لدرجة أنها لا تشترط أن يكون الكائنان من نفس الـ Type! طالما أن الكائنين يملكان نفس أسماء الـ Properties ونفس القيم، ستعتبرهما متطابقين. هذا مفيد جداً لمقارنة DTOs مع Domain Objects.

> [!warning] تنبيه: الـ Exception Message
> 
> عند استخدام `.WithMessage("...")`، تأكد من نسخ الرسالة حرفياً (شاملة النقاط والمسافات). أي اختلاف بسيط سيسبب فشل الاختبار.


---

## المفهوم الجوهري (Core Concept)

يغطي هذا الجزء سد الفجوات في إمكانية الوصول للاختبار (Testability Gaps): استخدام `Monitor` لاختبار الـ **Events**، الالتزام بقاعدة "عدم اختبار الـ **Private Methods** مباشرة"، وكيفية كشف الـ **Internal Members** لمشروع الاختبار بشكل آمن ونظيف.

---

## الشرح التفصيلي (Detailed Breakdown)

### 1. اختبار الأحداث (Asserting Raised Events)

في البرمجة المعتمدة على الأحداث (Event-Driven)، قد لا يكون للدالة قيمة مرجعة (Return Value) لنتحقق منها، بل يكون أثرها هو إطلاق حدث (Event).

- **المشكلة:** الـ Assertions العادية تفحص القيم ولا "تسمع" الأحداث.
    
- **الحل:** استخدام ميزة `Monitor` في Fluent Assertions.
    
    - نقوم بإنشاء "مراقب" (Monitor Subject) يلتصق بالكائن (SUT).
        
    - هذا المراقب يسجل أي حدث يتم إطلاقه.
        
    - نتحقق باستخدام `Should().Raise("EventName")`.
        

### 2. معضلة الدوال الخاصة (Testing Private Methods)

- **السؤال الأزلي:** "كيف أختبر Private Method؟"
    
- **الإجابة القاطعة:** **لا تفعل ذلك مباشرة.**
    
    - محاولة استدعاء دالة `private` من مشروع الاختبار ستفشل (Compiler Error).
        
- **القاعدة الذهبية:** لا تقم أبداً بتغيير الدالة من `private` إلى `public` فقط من أجل اختبارها.
    
- **الاستراتيجية الصحيحة:** الـ Private Methods هي تفاصيل تنفيذية (Implementation Details). يتم اختبارها **بشكل غير مباشر** (Indirectly) عن طريق اختبار الدالة الـ `public` التي تستدعيها.
    
    - مثال: إذا كانت دالة `Divide` (عامة) تستدعي `EnsureNotZero` (خاصة)، فعندما تختبر `Divide` وترسل صفراً، أنت فعلياً تختبر منطق الدالة الخاصة.
        

### 3. اختبار الأعضاء الداخلية (Testing Internal Members)

- **السيناريو:** لديك كلاس أو دالة معرفة كـ `internal`. هذا يعني أنها مرئية فقط داخل نفس الـ Assembly (المشروع نفسه)، وليست مرئية لمشروع الـ Tests الخارجي.
    
- **هل يجب اختبارها؟** نعم، هذا كود خاص بك ومن حقك اختباره (على عكس الـ Private).
    
- **الحل (كشف الـ Internals):** يجب إخبار المشروع الأصلي بأن "يسمح" لمشروع الاختبار برؤية محتوياته الداخلية.
    

#### طرق التنفيذ (InternalsVisibleTo):

1. **الطريقة القديمة (The Old Way):**
    
    - إنشاء ملف كود (مثلاً `AssemblyInfo.cs`) وإضافة Attribute:
        
    - `[assembly: InternalsVisibleTo("Project.Tests")]`
        
2. **الطريقة الحديثة والموصى بها (.csproj Way):**
    
    - التعديل مباشرة في ملف المشروع (`.csproj`) الخاص بالكود المصدري (Source Project).
        
    - هذه الطريقة أنظف ولا تتطلب إنشاء ملفات كود وهمية.
        

---

## أمثلة الكود (Code Snippets)

### 1. اختبار الـ Events (بواسطة Fluent Assertions)



```csharp
[Fact]
public void Method_ShouldRaiseEvent_WhenCalled()
{
    // Arrange
    var sut = new Calculator();
    // نقوم بإنشاء "جاسوس" أو مراقب على الكائن
    using var monitor = sut.Monitor(); 

    // Act
    sut.RaiseExampleEvent(); // الدالة التي تطلق الحدث

    // Assert
    // نتحقق أن الحدث المسمى "ExampleEvent" قد تم إطلاقه فعلاً
    monitor.Should().Raise("ExampleEvent");
}
```

### 2. كشف الـ Internals (عن طريق ملف .csproj)

نذهب إلى ملف المشروع الأساسي (وليس ملف التيست) ونضيف التالي:



```XML
<Project Sdk="Microsoft.NET.Sdk">
  
  <ItemGroup>
    <InternalsVisibleTo Include="CalculatorLibrary.Tests.Unit" />
  </ItemGroup>

</Project>
```

---

## لماذا؟ (The "Why")

- **لماذا نستخدم `Monitor` مع الأحداث؟**
    
    - لأن البديل هو كتابة كود يدوي معقد: الاشتراك في الحدث (`+=`)، وتغيير قيمة متغير `bool eventRaised = true` عند حدوثه، ثم التحقق من المتغير. `Monitor` تختصر كل هذا في سطر واحد.
        
- **لماذا لا نختبر الـ Private Methods؟**
    
    - لأن ذلك يكسر مبدأ **Encapsulation**. اختباراتك يجب أن تهتم بـ "ماذا يفعل النظام" (Behavior) وليس "كيف يفعله" (Implementation). إذا ربطت اختبارك بدالة خاصة، فإن أي Refactoring داخلي بسيط سيكسر الاختبارات حتى لو كان السلوك الخارجي سليماً (Brittle Tests).
        
- **لماذا نستخدم `InternalsVisibleTo`؟**
    
    - لأننا نريد حماية الـ API الخاص بمكتبتنا (لا نريد للمستخدمين رؤية الكلاسات الداخلية)، لكننا في نفس الوقت نحتاج للتأكد من جودة هذه الكلاسات عبر الـ Unit Tests.
        

---

> [!warning] تنبيه هام جداً
> 
> لا تخلط بين `Private` و `Internal`.
> 
> - **Private:** ممنوع اللمس. اختبره عبر الـ Public API.
>     
> - **Internal:** مسموح اللمس فقط لمشروع الاختبار (عبر `InternalsVisibleTo`).
>     

> [!tip] نصيحة: استخدام الـ .csproj
> 
> استخدام ملف المشروع (`.csproj`) لإضافة `InternalsVisibleTo` أفضل من استخدام الـ Attributes في الكود، لأنه يبقي الكود المصدري نظيفاً (Clean Code) ويجعل إعدادات التكوين (Configuration) مركزية في مكان واحد.

---

