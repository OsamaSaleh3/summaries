

---

## المفهوم الجوهري (Core Concept)

الـ **Unit Testing** هو حجر الأساس في "هرم الاختبارات" (Testing Pyramid)، وهدفه فحص أصغر وحدة من الكود (Method/Class) بمعزل تام عن العالم الخارجي (Isolating Dependencies) لضمان سرعة التنفيذ واكتشاف الأخطاء مبكراً قبل وصولها للـ Production.

---

## الشرح التفصيلي (Detailed Breakdown)

### 1. ما هو الاختبار (Testing) في جوهره؟

- ببساطة، هو كتابة **كود للتحقق من كود آخر** (Code that validates behavior of code).
    
- يعتمد على مبدأ: **Input** $\rightarrow$ **Processing** $\rightarrow$ **Expected Output**.
    
- إذا اختلف الـ Output الحقيقي عن الـ Expected Output، يفشل الاختبار (Assertion Fails).
    

### 2. أنواع الاختبارات (Testing Types Landscape)

لقد استعرض المحاضر الأنواع المختلفة لتعريف "حدود" الـ Unit Testing:

- **Unit Testing (الاختبار الوحدوي):**
    
    - **الهدف:** فحص "وحدة" (Unit) قد تكون Method أو Class.
        
    - **السمة الأساسية:** أي تعامل مع الـ I/O (قواعد بيانات، API، ملفات Disk) يتم استبداله بـ **Mocking**.
        
    - **السرعة:** فائق السرعة (يُمكن تشغيله مع كل عملية Save للكود).
        
    - **البيئة:** لا يحتاج لبيئة تشغيل معقدة (No complex environment setup).
        
- **Component Testing:**
    
    - نسخة أوسع قليلاً من الـ Unit Testing.
        
    - قد تفحص مجموعة من الـ Classes معاً، لكنها غالباً ما تزال تستخدم الـ Mocks للأنظمة الخارجية.


        
- **Integration Testing (الاختبار التكاملي):**
    
    - **الهدف:** التأكد من أن الكود يعمل بشكل صحيح مع الـ Dependencies الحقيقية.
        
    - **السمة:** يتصل فعلياً بقاعدة البيانات أو الـ API (Real Database/API Calls).
        
    - **البيئة:** يحتاج لبيئة تشغيل (مثل Docker Containers).
        
    - **النطاق:** أوسع (مثلاً: دورة حياة Request كاملة من الـ API وحتى الاستجابة).

        
- **End-to-End Testing (E2E):**
    
    - **الهدف:** محاكاة سلوك المستخدم النهائي (User Behavior) عبر النظام كاملاً.
        
    - **النطاق:** يشمل عدة أنظمة (Microservices, Message Bus, UI, DB).
        
    - **العيوب:** بطيء جداً، ومعقد في الإعداد (يحتاج Timeout, Delays).
        

### 3. هرم الاختبارات (The Testing Pyramid)

هو نموذج مرئي لتوضيح الكمية والأهمية لكل نوع اختبار.

![Image of software testing pyramid](https://encrypted-tbn0.gstatic.com/licensed-image?q=tbn:ANd9GcQbbVtjG1ZGe4nsWCOOIbcbBnqMyvvQ55XXj4mo89wXF_UqyPsuKvsdVNMp7-6lZ8692dl0MSGiwDi5-URR4IXTmDN0lANNR00DfPlhC_xkyTSVovs)

Getty Images

- **القاعدة (Unit Tests):**
    
    - الحجم: الأكبر (الأكثر عدداً).
        
    - السرعة: الأسرع.
        
    - التكلفة: الأقل.
        
- **الوسط (Integration Tests):**
    
    - الحجم: متوسط.
        
    - أبطأ وأعلى تكلفة من الـ Unit Tests.
        
- **القمة (E2E Tests):**
    
    - الحجم: الأقل (مثلاً: بضعة اختبارات للسيناريوهات الحرجة فقط).
        
    - السرعة: الأبطأ.
        
    - التركيز: Business Logic بحت.
        

---

## أمثلة الكود (Code Snippets)

المحاضر ذكر مثالاً شفهياً عن دالة تقوم بفصل الاسم الكامل إلى اسم أول واسم أخير. إليك تحويل هذا المثال إلى كود C# يوضح الفكرة:



```Csharp
// 1. The Production Code (The Logic we want to test)
public class NameParser
{
    public (string FirstName, string LastName) Parse(string fullName)
    {
        var parts = fullName.Split(' ');
        return (parts[0], parts[1]);
    }
}

// 2. The Unit Test (The Validation Logic)
[Fact] // xUnit attribute example
public void Parse_ShouldReturnFirstAndLastName_WhenFullNameIsValid()
{
    // Arrange: تهيئة المتغيرات
    var parser = new NameParser();
    string input = "Nick Chapsas";

    // Act: تنفيذ الكود المراد فحصه
    var result = parser.Parse(input);

    // Assert: التحقق من أن النتيجة تطابق التوقعات
    // نحن هنا نتأكد أن الكود يعمل كما صمم له
    Assert.Equal("Nick", result.FirstName);
    Assert.Equal("Chapsas", result.LastName);
}
```

**شرح المنطق:**

- إذا قام شخص بتغيير الـ Logic في المستقبل وأصبح الـ Split يعتمد على الفاصلة `,` بدلاً من المسافة، سيفشل هذا الاختبار فوراً، منبهاً المطور بوجود خطأ (Regression).
    

---

## لماذا؟ (The "Why")

### لماذا نستخدم Mocking في الـ Unit Tests؟

- **السرعة:** الاتصال بقاعدة بيانات حقيقية بطيء. الـ Unit Tests يجب أن تنتهي في أجزاء من الثانية.
    
- **العزل (Isolation):** لا نريد أن يفشل الاختبار لأن الشبكة (Network) انقطعت أو قاعدة البيانات توقفت. نحن نفحص "منطق الكود" فقط.
    
- **تجنب الإعداد المعقد:** لا حاجة لتثبيت قاعدة بيانات على جهاز كل مطور لتشغيل الاختبارات.
    

### لماذا الـ Unit Testing مهم جداً (Business Value)؟

1. **توفير المال:** اكتشاف الـ Bug أثناء التطوير أرخص بآلاف المرات من اكتشافه بعد الـ Production (حيث يؤثر على العملاء والسمعة).
    
2. **التوثيق الحي (Documentation):** اسم الاختبار (مثل `Parse_ShouldReturn...`) يشرح وظيفة الكود أفضل من أي ملف نصي قديم.
    
3. **شبكة أمان (Safety Net):** يسمح لك بعمل **Refactoring** وتحسين الكود بقلب قوي، لأنك تعلم أن الاختبارات ستخبرك فوراً إذا كسرت شيئاً ما.
    
4. **تحسين جودة الكود:** كتابة كود قابل للاختبار (Testable Code) تجبرك تلقائياً على اتباع ممارسات جيدة مثل **Dependency Injection** و **Single Responsibility**.
    

---

> [!warning] خطأ شائع: الخلط بين Unit و Integration
> 
> إذا كان الاختبار الخاص بك يتصل بقاعدة بيانات حقيقية، أو ملفات على القرص (File System)، أو شبكة خارجية... فهذا **ليس** Unit Test. هذا Integration Test.
> 
> **القاعدة الذهبية:** الـ Unit Test يجب أن يعمل حتى لو لم يكن لديك اتصال بالإنترنت أو قاعدة بيانات مثبتة.

> [!tip] نصيحة: الـ TDD (Test Driven Development)
> 
> ذكر المحاضر أسلوب الـ TDD (كتابة الاختبار قبل الكود الفعلي). على الرغم من أنه يراه "Extreme" (متطرفاً) قليلاً أحياناً، إلا أنه يضمن أن الكود المكتوب هو كود قابل للاختبار ومصمم بشكل جيد منذ البداية.

---
