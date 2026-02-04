

---

## المفهوم الجوهري (Core Concept)

الـ Unit Testing المستحيل يصبح ممكناً فقط عند تطبيق مبدأ **Inversion of Control (IoC)** و **Dependency Injection (DI)**، حيث نستبدل الاعتماد المباشر على الكلاسات الحقيقية (مثل قواعد البيانات) بالاعتماد على التجريدات (Interfaces)، مما يسمح لنا باستبدال الأنظمة المعقدة ببدائل وهمية (Fakes/Mocks) أثناء الاختبار.

---

## الشرح التفصيلي (Detailed Breakdown)

### 1. مشكلة "الترابط القوي" (The Tight Coupling Problem)

- **السيناريو:** لدينا `UserService` وبداخل الـ Constructor نقوم بإنشاء `new UserRepository()`.
    
- **الأثر:** عند محاولة اختبار `UserService`، سيقوم الكود تلقائياً بمحاولة الاتصال بقاعدة البيانات الحقيقية لأن `UserRepository` مصمم لذلك.
    
- **النتيجة:** فشل الاختبار (خطأ: `No such table`)، أو الاعتماد على بيئة خارجية، وهذا ينتهك مبدأ الـ Unit Testing (العزل).
    
- **القاعدة:** كلمة `new` داخل الـ Constructor تعتبر "صمغاً" (Glue) يربط الكود ببعضه بشكل يمنع الاختبار.
    

### 2. الحل: قلب التحكم (Inversion of Control & DI)

لجعل الكود قابلاً للاختبار، يجب ألا يقوم الكلاس بإنشاء الـ Dependencies بنفسه، بل يجب أن "يحقن" بها (Injected).

1. **استخراج الواجهة (Extract Interface):** ننشئ `IUserRepository` يحتوي على توقيع الدوال (Signatures).
    
2. **التعديل (Refactor):** نجعل `UserService` يطلب `IUserRepository` في الـ Constructor بدلاً من `UserRepository`.
    
3. **النتيجة:** أصبح بإمكاننا الآن تمرير أي شيء يطبق هذه الواجهة، سواء كان قاعدة بيانات حقيقية (في الـ Production) أو كلاس وهمي (في الـ Testing).
    

### 3. استخدام البدائل اليدوية (Fakes)

- قبل استخدام مكتبات الـ Mocking، يمكننا حل المشكلة يدوياً بإنشاء كلاس اسمه `FakeUserRepository`.
    
- هذا الكلاس يطبق `IUserRepository` لكنه يعيد بيانات وهمية وثابتة (Hardcoded) ولا يتصل بأي قاعدة بيانات.
    
- **المشكلة في الـ Fakes:**
    
    - غير قابلة للتوسع (Not Scalable).
        
    - إذا أردت اختبار سيناريو "قائمة فارغة"، تحتاج لـ Fake يعيد قائمة فارغة.
        
    - إذا أردت اختبار سيناريو "مستخدمين موجودين"، تحتاج لإنشاء Fake آخر أو تعديل الحالي.
        
    - سينتهي بك المطاف بكتابة عشرات الكلاسات الوهمية فقط للاختبار.
        

---

## أمثلة الكود (Code Snippets)

### 1. الكود السيء (Hard to Test)



```csharp
public class UserService
{
    private readonly UserRepository _repository;

    public UserService()
    {
        // ❌ خطأ قاتل: الاعتماد المباشر يمنع استبدال قاعدة البيانات
        _repository = new UserRepository(); 
    }
}
```

### 2. الكود بعد التعديل (Refactored for DI)



```csharp
// 1. التعريف المجرد (The Contract)
public interface IUserRepository
{
    Task<IEnumerable<User>> GetAllAsync();
}

// 2. الكود القابل للاختبار
public class UserService
{
    private readonly IUserRepository _repository;

    // ✅ الحقن: نطلب الواجهة، ولا نهتم بنوع التنفيذ
    public UserService(IUserRepository repository)
    {
        _repository = repository;
    }
}
```

### 3. استخدام الـ Fake في الاختبار (Manual Mocking)



```csharp
// كلاس وهمي مخصص للاختبار فقط
public class FakeUserRepository : IUserRepository
{
    public Task<IEnumerable<User>> GetAllAsync()
    {
        // إرجاع قائمة فارغة دون الاتصال بقاعدة بيانات
        return Task.FromResult(Enumerable.Empty<User>());
    }
}

// الاختبار
[Fact]
public async Task GetAllAsync_ShouldReturnEmptyList_WhenNoUsersExist()
{
    // Arrange
    // نمرر الـ Fake بدلاً من الـ Repository الحقيقي
    var fakeRepo = new FakeUserRepository(); 
    var sut = new UserService(fakeRepo);

    // Act
    var result = await sut.GetAllAsync();

    // Assert
    result.Should().BeEmpty();
}
```

---

## لماذا؟ (The "Why")

- **لماذا نحتاج Interface؟**
    
    - لأن لغة C# (وغيرها من اللغات) لا تسمح باستبدال كلاس بكلاس آخر إلا إذا كانا يشتركان في نفس الأب أو الواجهة. الـ Interface هي "عقد" يضمن أن الـ Fake يملك نفس دوال الـ Real Object.
        
- **لماذا فشلت طريقة الـ Fakes؟**
    
    - تخيل لو لديك 10 سيناريوهات (نجاح، فشل قاعدة البيانات، بيانات ناقصة...). ستضطر لإنشاء 10 كلاسات Fake مختلفة أو جعل الـ Fake معقداً جداً. هذا جهد صيانة ضخم (Maintenance Overhead). الحل هو الـ **Mocking** (الذي سنراه لاحقاً).
        

---

> [!warning] تنبيه: New is Glue
> 
> العبارة الشهيرة "New is Glue" تعني أن استخدام `new` لإنشاء الـ Dependencies داخل كلاسك يلصق الكود ببعضه للأبد ويمنعك من عزله. الحل دائماً هو طلب الـ Dependency عبر الـ Constructor.

> [!tip] نصيحة
> 
> الـ `Fake` هو مجرد تطبيق (Implementation) حقيقي لكنه مبسط للواجهة. بينما الـ `Mock` (الذي سنشرحه لاحقاً) هو كائن ديناميكي يتم بناؤه وتشكيله أثناء التشغيل (Runtime) دون الحاجة لإنشاء كلاسات فعلية.

---





---

## المفهوم الجوهري (Core Concept)

الـ **Mocking** هو عملية إنشاء نسخ افتراضية (In-Memory Implementations) من الـ Interfaces في وقت التشغيل (Runtime) باستخدام مكتبات متخصصة مثل **NSubstitute**، مما يغنينا عن كتابة عشرات الـ Fakes اليدوية ويمنحنا تحكماً كاملاً في سلوك الـ Dependencies بأسطر كود قليلة ونظيفة.

---

## الشرح التفصيلي (Detailed Breakdown)

### 1. ما هو الـ Mocking ولماذا هو الحل الأمثل؟

- **التعريف:** بدلاً من إنشاء كلاس حقيقي (`FakeUserRepository`) وكتابة الكود بداخله يدوياً، نطلب من مكتبة الـ Mocking أن تخلق لنا كائناً (Object) يطبق الـ Interface فوراً في الذاكرة.
    
- **الميزة القاتلة:** المرونة. يمكنك تغيير سلوك هذا الـ Mock (ماذا يرجع دالة `GetAllAsync`؟) داخل كل اختبار على حدة، دون الحاجة لإنشاء كلاسات جديدة.
    

### 2. مكتبة NSubstitute (الخيار المفضل)

اختار المحاضر مكتبة **NSubstitute** لعدة أسباب جوهرية:

- **البساطة (Low Ceremony):** الكود يبدو كأنه C# عادي، لا يوجد تعقيد في الإعداد.
    
- **التعامل المباشر:** الدالة `Substitute.For<IInterface>()` تعيد لك الـ Object مباشرة لتستخدمه في الـ Constructor.
    
- **سهولة القراءة:** إعداد القيم المرجعة يتم بأسلوب مقروء:
    
    - `repository.Method().Returns(value);`
        

### 3. مقارنة: NSubstitute vs Moq

مكتبة **Moq** هي الأكثر شهرة، لكن المحاضر (والعديد من الخبراء) يفضلون NSubstitute، والسبب يكمن في "الطقوس" (Ceremony) الزائدة في Moq:

|**وجه المقارنة**|**NSubstitute (Clean)**|**Moq (Verbose)**|
|---|---|---|
|**إنشاء الـ Mock**|`Substitute.For<IUserRepo>()`|`new Mock<IUserRepo>()`|
|**الحصول على الكائن**|الكائن نفسه هو الـ Mock|تحتاج لاستدعاء `.Object`|
|**إعداد السلوك**|`repo.GetAll().Returns(...)`|`mock.Setup(x => x.GetAll()).ReturnsAsync(...)`|
|**التقييم**|كود نظيف ومباشر|كود مليء بـ Lambda Expressions و Setup|

---

## أمثلة الكود (Code Snippets)

### 1. إعداد الـ Mock باستخدام NSubstitute

كيف نستبدل الـ Fake اليدوي بـ Mock ذكي:



```csharp
using NSubstitute; // ضروري

[Fact]
public async Task GetAllAsync_ShouldReturnEmptyList_WhenNoUsersExist()
{
    // Arrange
    // 1. إنشاء الـ Mock (Proxy Object)
    var repositoryMock = Substitute.For<IUserRepository>();

    // 2. ضبط السلوك (Configuration)
    // "عندما يتم استدعاء GetAllAsync، أرجع مصفوفة فارغة"
    repositoryMock.GetAllAsync().Returns(Array.Empty<User>());

    // 3. حقن الـ Mock داخل السيرفس (Dependency Injection)
    var sut = new UserService(repositoryMock);

    // Act
    var result = await sut.GetAllAsync();

    // Assert
    result.Should().BeEmpty();
}
```

### 2. سيناريو إرجاع بيانات (Returning Data)

كيف نجعل الـ Mock يعيد مستخدماً محدداً:



```csharp
[Fact]
public async Task GetAllAsync_ShouldReturnUsers_WhenUsersExist()
{
    // Arrange
    var repositoryMock = Substitute.For<IUserRepository>();
    
    var expectedUser = new User { FullName = "Nick Chapsas" };
    var usersList = new[] { expectedUser };

    // ضبط الـ Mock ليرجع القائمة التي أنشأناها
    repositoryMock.GetAllAsync().Returns(usersList);

    var sut = new UserService(repositoryMock);

    // Act
    var result = await sut.GetAllAsync();

    // Assert
    result.Should().ContainSingle(u => u.FullName == "Nick Chapsas");
}
```

---

## لماذا؟ (The "Why")

- **لماذا NSubstitute وليس Moq؟**
    
    - في Moq، أنت تتعامل مع "غلاف" (Wrapper) حول الـ Interface، لذا يجب عليك دائماً استخدام `.Object` لتمريره للكلاس، واستخدام `.Setup(...)` لبرمجته. هذا يضيف "ضوضاء" (Noise) للكود. في NSubstitute، الـ Mock هو نفسه الـ Interface، مما يجعل الكود أنظف وأقرب للغة الطبيعية.
        
- **لماذا نستخدم Mock بدلاً من الـ Database الحقيقية؟**
    
    - السرعة (Milliseconds vs Seconds).
        
    - الموثوقية (لا يوجد Network Flakes).
        
    - العزل (نحن نختبر منطق الـ Service وليس اتصال الـ DB).
        

---

> [!tip] نصيحة: Runtime Proxy
> 
> عندما تستخدم `Substitute.For<T>()`، المكتبة تقوم سحرياً بإنشاء كلاس في الذاكرة (Dynamic Proxy) يطبق الـ Interface T. هذا الكلاس "فارغ" تماماً، وأي دالة تستدعيها ستعيد القيمة الافتراضية (null أو 0) ما لم تقم أنت ببرمجتها باستخدام `.Returns()`.

> [!warning] تحذير
> 
> عند استخدام **Moq** (إذا اضطررت لذلك)، لا تنسَ أبداً استخدام `.Object` عند تمرير الـ Dependency للـ Constructor. تمرير متغير الـ `Mock` نفسه سيسبب خطأ في الترجمة (Compilation Error) لأن النوع غير متطابق.

---

