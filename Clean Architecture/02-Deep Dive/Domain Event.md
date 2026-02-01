
## 1. What is a Domain Event?

ببساطة، **Domain Event** هو شيء مهم من وجهة نظر الـ Business قد **حدث بالفعل** داخل النظام.

### الخصائص الجوهرية:

1. **حدث في الماضي:** دائماً نعبر عنه بصيغة الماضي (e.g., `SubscriptionDeleted` وليس `DeleteSubscription`).
    
2. **أهمية بيزنس:** ليس مجرد تفصيل تقني (مثل "تم النقر على زر")، بل تغيير في حالة النظام يهم البيزنس.
    

### المثال العملي: حذف الاشتراك (Deleting a Subscription)

تخيل عملية حذف اشتراك لمدير جيم. هذه العملية تتطلب سلسلة من الإجراءات:

1. حذف الاشتراك نفسه.
    
2. حذف جميع الجيمات التابعة له.
    
3. حذف الغرف والجلسات داخل تلك الجيمات.
    

**في الأسلوب التقليدي (Orchestration):**

كنا نكتب كوداً طويلاً في الـ Handler يقول: "احذف الاشتراك، ثم احذف الجيم 1، ثم الجيم 2، ثم الغرفة..." خطوة بخطوة.

**في أسلوب Domain Events:**

1. الـ Admin يقوم بحذف الاشتراك.
    
2. بمجرد الحذف، يطلق النظام "صرخة" أو حدث: **"يا جماعة، تم حذف اشتراك!" (`SubscriptionDeleted`)**.
    
3. أجزاء النظام الأخرى (Event Handlers) تسمع هذه الصرخة وتتصرف بناءً عليها (مثل حذف الجيمات المرتبطة) بشكل منفصل.
    

---

## 2. The Tactical Pattern: How it Works?

آلية عمل هذا النمط داخل **Clean Architecture** تتكون من ثلاث مراحل:

### المرحلة 1: الالتقاط (Capture) 📸

عندما يحدث تغيير في الـ Domain Object (مثل `admin.DeleteSubscription()`):

- لا نقوم بتنفيذ الآثار الجانبية (Side Effects) فوراً.
    
- بدلاً من ذلك، نضيف "حدثاً" (`new SubscriptionDeletedEvent`) إلى **قائمة داخلية** في كائن الـ Admin.
    
- هذا الحدث يبقى "نائماً" في الذاكرة مؤقتاً.
    

### المرحلة 2: الحفظ (Persist) 💾

عندما ننادي `SaveChangesAsync` لحفظ التغييرات في قاعدة البيانات:

- يتم حفظ حالة الـ Admin الجديدة (بدون اشتراك).
    
- **فقط بعد نجاح الحفظ**، نقوم بنشر (Publish) الأحداث النائمة. هذا يضمن عدم إطلاق أحداث لعمليات فشلت في التخزين.
    

### المرحلة 3: التفاعل (React) ⚡

بمجرد نشر الحدث:

- يستقبله واحد أو أكثر من **Event Handlers**.
    
- يقوم كل Handler بتنفيذ منطق معين (حذف جيمات، إرسال إيميل، إلخ).
    
- **ملاحظة مهمة:** العلاقة هنا هي **One-to-Many**. حدث واحد يمكن أن يطلق عدة Handlers.
    

---

## 3. Implementation Steps: Setting the Foundation

لنبدأ بكتابة الكود التأسيسي لهذا النمط في طبقة **Domain Layer**.

### Step 1: Create the Event Class

في مشروع الـ Domain، ننشئ مجلد `Events` داخل `Admin` ونعرف الحدث:



```C#
// Domain/Admin/Events/SubscriptionDeletedEvent.cs
public record SubscriptionDeletedEvent(Guid SubscriptionId) : IDomainEvent;
```

- نستخدم `record` لأنه يمثل بيانات ثابتة (Immutable).
    
- يحتوي على البيانات الضرورية فقط (مثل `SubscriptionId`).
    

### Step 2: Define `IDomainEvent` & MediatR Integration

لكي نتمكن من نشر الأحداث بسهولة، سنعتمد على مكتبة **MediatR** التي تدعم نمط Publisher/Subscriber.

1. نضيف مكتبة `MediatR` لمشروع **Domain**.
    
2. نعرف واجهة `IDomainEvent` التي ترث من `INotification` (الخاصة بـ MediatR):
    



```C#
// Domain/Common/IDomainEvent.cs
using MediatR;

public interface IDomainEvent : INotification
{
}
```

### Step 3: Base Entity with Event List

نحتاج لمكان لتخزين الأحداث "النائمة" في كل Entity. بدلاً من تكرار القائمة في كل كلاس، ننشئ **Base Class**:



```C#
// Domain/Common/Entity.cs
public abstract class Entity
{
    public Guid Id { get; init; } // Init-only for immutability

    // القائمة الداخلية لتخزين الأحداث
    private readonly List<IDomainEvent> _domainEvents = new();

    // خاصية للوصول للأحداث (للنشر لاحقاً)
    public IReadOnlyList<IDomainEvent> DomainEvents => _domainEvents.AsReadOnly();

    protected Entity(Guid id)
    {
        Id = id;
    }

    // كونستركتور فارغ لـ EF Core
    protected Entity() { }

    // دالة لإضافة حدث للقائمة
    protected void AddDomainEvent(IDomainEvent domainEvent)
    {
        _domainEvents.Add(domainEvent);
    }

    // دالة لمسح الأحداث بعد نشرها
    public void ClearDomainEvents()
    {
        _domainEvents.Clear();
    }
}
```

### Step 4: Update the Admin Entity

الآن نجعل `Admin` يرث من `Entity` ويستخدم الآلية الجديدة:



```C#
// Domain/Admin/Admin.cs
public class Admin : Entity
{
    public void DeleteSubscription(Guid subscriptionId)
    {
        // 1. Perform the business logic (removing subscription)
        SubscriptionId = null;

        // 2. Capture the event
        AddDomainEvent(new SubscriptionDeletedEvent(subscriptionId));
    }
}
```

**النتيجة:** الآن الـ Admin يقوم فقط بتسجيل أن "الاشتراك حُذف"، دون أن يهتم بمن سيقوم بتنظيف الجيمات أو إرسال الإيميلات. هذا يحقق **Decoupling** (فصل) ممتاز بين أجزاء النظام.

_في الدفعة القادمة، سنرى كيف سيتم نشر هذه الأحداث تلقائياً عند الحفظ، وكيفية التعامل مع الـ Side Effects._


---

# Deep Dive: Domain Events Pattern - Part 2 (Orchestration vs. Events)

في هذه الدفعة الثانية، سنقارن بين النهج الحالي (Orchestration) والنهج الجديد (Domain Events)، وسنقوم بتطبيق الـ Event Handlers بشكل عملي لفهم كيف تتحول عملية حذف الاشتراك من "عملية واحدة ضخمة" إلى "سلسلة تفاعلات منفصلة".

---

## 1. Orchestration vs. Domain Events 🥊

قبل كتابة الكود، يجب أن نفهم الفرق الجوهري بين الأسلوبين.

### A. Orchestration Approach (النهج الحالي)

في هذا الأسلوب، الـ Command Handler يلعب دور "المايسترو" أو القائد الذي يأمر الجميع.

- **كيف يعمل؟**
    
    - يا `AdminRepository`، احذف الاشتراك من الأدمن.
        
    - يا `SubscriptionRepository`، احذف الاشتراك نفسه.
        
    - يا `GymsRepository`، احذف كل الجيمات.
        
- **الميزة:** سهل الفهم (Low Cognitive Load). تقرأ الكود وتعرف كل شيء يحدث خطوة بخطوة.
    
- **العيوب:**
    
    - **تداخل المسؤوليات (Coupling):** الـ Subscription Feature تعرف تفاصيل عن الـ Gyms Feature.
        
    - **صعوبة إعادة الاستخدام:** لو أردنا حذف جيم لوحده، سنكتب كود الحذف مرة أخرى. لو أردنا حذف أدمن (الذي سيحذف اشتراكاً، الذي سيحذف جيماً...)، سيصبح الكود معقداً ومتشابكاً.
        

### B. Domain Events Approach (النهج الجديد)

في هذا الأسلوب، كل جزء يقوم بعمله فقط، ثم يخبر النظام "لقد انتهيت".

- **كيف يعمل؟**
    
    - الـ Command Handler يقول: "يا `AdminRepository`، احذف الاشتراك من الأدمن". **فقط.**
        
    - الأدمن يحذف الرابط ويرفع حدث: `SubscriptionDeleted`.
        
    - الـ Gyms Feature تسمع الحدث وتقول: "أوه! تم حذف اشتراك؟ سأحذف الجيمات التابعة له".
        
    - الـ Subscriptions Feature تسمع الحدث وتقول: "سأحذف سجل الاشتراك نفسه".
        
- **الميزة:** فصل تام (Decoupling). يمكنك إضافة ميزات جديدة (مثل إرسال إيميل) دون لمس كود الحذف الأصلي.
    

---

## 2. Refactoring the Command Handler 🛠️

سنقوم الآن بتنظيف `DeleteSubscriptionCommandHandler` ليتحول من "Orchestrator" إلى مجرد "Initiator".

### قبل (The Old Way):

كان الكود يحقن `IGymsRepository` و `ISubscriptionsRepository` ويقوم بكل الحذوفات بنفسه.

### بعد (The New Way):

سنحذف كل شيء ونبقي فقط على تحديث الـ Admin.


```C#
public class DeleteSubscriptionCommandHandler : IRequestHandler<DeleteSubscriptionCommand, ErrorOr<Success>>
{
    private readonly IAdminRepository _adminRepository;
    private readonly IUnitOfWork _unitOfWork; // (اختياري كما سنرى لاحقاً)

    public async Task<ErrorOr<Success>> Handle(...)
    {
        // 1. Get Admin
        var admin = await _adminRepository.GetByIdAsync(...);
        
        // 2. Delete Subscription (This captures the event internally)
        admin.DeleteSubscription(command.SubscriptionId);

        // 3. Update Admin Only
        await _adminRepository.UpdateAsync(admin);
        
        // 4. Save Changes
        // عند الحفظ، سيتم إطلاق الحدث تلقائياً (سننفذ هذا في السكشن القادم)
        await _unitOfWork.CommitChangesAsync();

        return Result.Success;
    }
}
```

- **ملاحظة:** تم حذف كل الكود المتعلق بحذف الجيمات والاشتراكات من هنا.
    

---

## 3. Implementing Event Handlers 🧩

الآن، من سيقوم بالعمل القذر (حذف الجيمات والاشتراكات)؟ إنهم الـ **Notification Handlers**.

### Handler 1: حذف سجل الاشتراك نفسه

في `Application/Subscriptions/Events`:



```C#
public class SubscriptionDeletedEventHandler : INotificationHandler<SubscriptionDeletedEvent>
{
    private readonly ISubscriptionsRepository _subscriptionsRepository;
    private readonly IUnitOfWork _unitOfWork;

    public async Task Handle(SubscriptionDeletedEvent notification, CancellationToken cancellationToken)
    {
        // 1. Get the subscription
        var subscription = await _subscriptionsRepository.GetByIdAsync(notification.SubscriptionId);

        // 2. Remove it
        await _subscriptionsRepository.RemoveSubscriptionAsync(subscription);

        // 3. Save
        await _unitOfWork.CommitChangesAsync();
    }
}
```

### Handler 2: حذف الجيمات التابعة

في `Application/Gyms/Events`:



```C#
public class SubscriptionDeletedEventHandler : INotificationHandler<SubscriptionDeletedEvent>
{
    private readonly IGymsRepository _gymsRepository;
    private readonly IUnitOfWork _unitOfWork;

    public async Task Handle(SubscriptionDeletedEvent notification, CancellationToken cancellationToken)
    {
        // 1. List all gyms for this subscription
        var gyms = await _gymsRepository.ListBySubscriptionIdAsync(notification.SubscriptionId);

        // 2. Remove them all
        await _gymsRepository.RemoveRangeAsync(gyms);

        // 3. Save
        await _unitOfWork.CommitChangesAsync();
    }
}
```

---

## 4. Key Concepts & Side Notes 💡

### A. One-to-Many Relationship

لاحظ أن حدثاً واحداً (`SubscriptionDeletedEvent`) تسبب في تشغيل **اثنين** من الـ Handlers في أماكن مختلفة تماماً من النظام (`Gyms` و `Subscriptions`). هذا هو جوهر الـ Decoupling.

### B. Unit of Work Redundancy?

المحاضر أشار لنقطة متقدمة: بما أننا في DDD نحاول دائماً تعديل **Aggregate واحد فقط** في كل عملية (Transaction)، فإن الـ `IUnitOfWork` التي بنيناها قد تكون زائدة عن الحاجة.

- الـ Command Handler عدل الـ Admin فقط.
    
- الـ Event Handler الأول عدل الـ Subscription فقط.
    
- الـ Event Handler الثاني عدل الـ Gyms فقط.
    
    كل واحد منهم يمكنه استخدام `SaveChangesAsync` الخاص بـ EF Core مباشرة، لأن كل عملية هي Transaction مستقلة وصغيرة.
    

### C. Eventual Consistency (الاتساق النهائي)

بما أن العمليات أصبحت منفصلة، النظام لم يعد "Atomic" بالمعنى التقليدي (كل شيء يحدث في لحظة واحدة).

- قد يتم حذف الاشتراك من الأدمن، ولكن الجيمات تحذف بعده بـ 100 ميلي ثانية.
    
- هذا يسمى **Eventual Consistency**: النظام سيصبح متسقاً "في النهاية"، وليس بالضرورة "الآن".
    
- سنتحدث عن كيفية التعامل مع الأخطاء في هذا السيناريو في السكشن القادم (مثلاً: ماذا لو حذفنا الأدمن وفشل حذف الجيمات؟).
    

---
