

### 🎯 هدف المشروع (Project Goal)

بناء تطبيق يسمح للمستخدم بإدخال بيانات استثماره (المبلغ، الفائدة، المدة) وعرض جدول يوضح كيف سينمو هذا الاستثمار عاماً بعد عام.

### 🏗️ هيكلية التطبيق (App Structure)

نحتاج لتقسيم التطبيق إلى 3 مكونات رئيسية:

1. **Header Component:** لعرض العنوان والشعار (Logo).
    
2. **User Input Component:**
    
    - يحتوي على نموذج (Form) لجمع البيانات:
        
        - `Initial Investment`
            
        - `Annual Investment`
            
        - `Expected Return`
            
        - `Duration`
            
    - يستخدم **Two-Way Binding** لربط البيانات.
        
3. **Investment Results Component:**
    
    - يعرض النتائج النهائية في جدول (Table).
        
    - يستقبل البيانات المحسوبة ويعرضها باستخدام التكرار (`@for` أو `*ngFor`).
        

---

### 💡 ملاحظات تقنية هامة (Crucial Hints)

**1. مجلد الصور (Public Folder):**

- في هذا المشروع، الصور موجودة في مجلد `public` وليس `assets`.
    
- **القاعدة:** عند استدعاء الصورة في الـ HTML، لا تكتب `public/`.
    
    - ❌ خطأ: `src="public/investment-calculator-logo.png"`
        
    - ✅ صح: `src="investment-calculator-logo.png"`
        

**2. ملفات التصميم (Pre-written CSS):**

- لتوفير الوقت، ملفات الـ CSS لكل مكون جاهزة ومرفقة. عليك فقط إنشاء المكونات، ثم نسخ كود الـ CSS ولصقه في ملفات `component.css` الخاصة بك، والتركيز على قراءة أسماء الـ Classes لتكتب الـ HTML المناسب لها.
    

**3. منطق الحساب (Calculation Logic):**

- يوجد ملف اسمه `investment-results.ts` يحتوي على كود الحسابات المعقدة. لا تضيع وقتك في كتابة خوارزمية الفائدة المركبة، ركز فقط على كود **Angular**.
    

---

### 🤔 نقطة القرار (Decision Point)


لديك خياران رئيسيان للهيكلة:

1. **الطريقة الحديثة (Modern Approach):** استخدام **Standalone Components** مع **Signals** (هذا سيجعلك تتدرب على أحدث ميزات Angular 17/18).
    
2. **الطريقة الكلاسيكية (Classic Approach):** استخدام **NgModules** (لتثبيت المعلومات التي تعلمناها في السيكشن السابق).
    


>[!Tip]
>End


---


# Step 1: Building the Header Component

## 🎯 الغرض من هذه الخطوة (The Goal)

إنشاء مكون يعرض عنوان التطبيق والشعار (Logo).

- **التحدي:** التعامل مع الصور الموجودة في مجلد `public` (الجديد في Angular) بدلاً من `assets`.
    

---

## 💻 التطبيق العملي (Step-by-Step)

### 1. إنشاء المكون (Generate Component)

نستخدم Angular CLI لإنشاء المكون بسرعة.



```Bash
ng g c header --skip-tests
```

- **ملاحظة:** سيتم إنشاؤه كـ **Standalone Component** افتراضياً (إلا إذا كنت تستخدم Angular 19 ولم تحدد ذلك، لكننا سنفترض الإعدادات القياسية للمشروع المرفق).
    

### 2. هيكلة القالب (Header Template)

نقوم ببناء الهيكل وعرض الصورة.

**src/app/header/header.component.html:**



```HTML
<header>
  <img src="investment-calculator-logo.png" alt="Money Bag" />
  <h1>Investment Calculator</h1>
</header>
```

### 3. التنسيق (Styling)

_(قم بنسخ كود CSS المرفق مع الدرس ولصقه في ملف `header.component.css`)_. هذا سيقوم بضبط حجم الصورة ومحاذاة النص في المنتصف.

### 4. الاستخدام (Integration)

بما أننا نستخدم **Standalone Components** حالياً، يجب استيراد المكون يدوياً داخل `AppComponent`.

**src/app/app.component.ts:**



```TypeScript
import { Component } from '@angular/core';
import { HeaderComponent } from './header/header.component'; // 1. Import

@Component({
  selector: 'app-root',
  standalone: true,
  templateUrl: './app.component.html',
  styleUrl: './app.component.css',
  imports: [HeaderComponent], // 2. Add to imports array
})
export class AppComponent {}
```

**src/app/app.component.html:**



```HTML
<app-header />
```

---

## ⚠️ ملاحظة تقنية (Assets vs Public)

> [!INFO] لماذا لا نكتب `public/`؟
> 
> في المشاريع الحديثة (Vite-based builders)، المجلد `public` يتم نسخه إلى **جذر** السيرفر (Root) عند البناء.
> 
> - المسار في المشروع: `public/logo.png`
>     
> - المسار في المتصفح: `localhost:4200/logo.png`
>     
>     لذلك نكتب اسم الملف مباشرة.
>     

**الخطوة التالية:** الرأس يظهر بشكل جميل الآن. الخطوة القادمة هي التحدي الأكبر: إنشاء مكون **User Input** لجمع البيانات (Initial Investment, Annual Investment, etc..) باستخدام **Two-Way Binding**. هل أنت مستعد للتعامل مع النماذج؟

>[!Tip]
>End

---


# Step 2: User Input Component (Structure & Layout)

## 🎯 الغرض من هذه الخطوة (The Goal)

إنشاء نموذج (Form) يحتوي على 4 حقول إدخال رقمية وزر للحساب، وعرضه في الصفحة الرئيسية.

- **الحقول المطلوبة:**
    
    1. Initial Investment (الاستثمار المبدئي).
        
    2. Annual Investment (الاستثمار السنوي).
        
    3. Expected Return (العائد المتوقع %).
        
    4. Duration (المدة بالسنوات).
        

---

## 💻 التطبيق العملي (Step-by-Step)

### 1. إنشاء المكون (Generate Component)

نستخدم CLI لإنشاء المكون الجديد بجانب `header`.



```Bash
ng g c user-input --skip-tests
```

### 2. هيكلة القالب (HTML Structure)

نحتاج لتقسيم الحقول إلى مجموعات باستخدام `input-group` لتظهر بشكل مرتب (حقلين في كل سطر حسب التنسيق الجاهز).

**src/app/user-input/user-input.component.html:**



```HTML
<form>
  <div class="input-group">
    <p>
      <label for="initial-investment">Initial Investment</label>
      <input id="initial-investment" type="number" />
    </p>
    <p>
      <label for="annual-investment">Annual Investment</label>
      <input id="annual-investment" type="number" />
    </p>
  </div>

  <div class="input-group">
    <p>
      <label for="expected-return">Expected Return</label>
      <input id="expected-return" type="number" />
    </p>
    <p>
      <label for="duration">Duration</label>
      <input id="duration" type="number" />
    </p>
  </div>

  <p class="actions">
    <button type="button">Calculate</button>
  </p>
</form>
```

### 3. التنسيق (Styling)

_(قم بنسخ كود CSS المرفق مع ملفات المشروع ولصقه في `user-input.component.css`)_.

هذا التنسيق سيجعل النموذج يظهر بجمالية، ويضع الحقول بجانب بعضها البعض.

### 4. الاستخدام (Integration)

الآن نعرض المكون في الصفحة الرئيسية تحت الرأس.

**src/app/app.component.ts:**



```TypeScript
import { Component } from '@angular/core';
import { HeaderComponent } from './header/header.component';
import { UserInputComponent } from './user-input/user-input.component'; // 1. Import

@Component({
  selector: 'app-root',
  standalone: true,
  templateUrl: './app.component.html',
  styleUrl: './app.component.css',
  imports: [HeaderComponent, UserInputComponent], // 2. Add to imports
})
export class AppComponent {}
```

**src/app/app.component.html:**



```HTML
<app-header />
<app-user-input /> ```

---

## ⚠️ ملاحظات (Observations)

1.  **نوع الحقل (`type="number"`):** استخدمنا هذا النوع ليظهر للمستخدم لوحة مفاتيح رقمية على الهاتف، وليمنع إدخال النصوص العشوائية، رغم أن القيمة تعود كـ `string` برمجياً في HTML العادي (سنعالج هذا لاحقاً).
2.  **الزر (`type="button"`):** حالياً الزر لا يفعل شيئاً. بمجرد الضغط عليه قد يحاول المتصفح إعادة تحميل الصفحة (Default Behavior) إذا لم نحدد نوعه بدقة أو نعالج حدث الإرسال.

**الخطوة التالية:** الشكل جاهز! لكن الحقول "غبية" لا تحتفظ بالبيانات، والزر يقوم بتحديث الصفحة. الخطوة القادمة هي تفعيل **Two-Way Binding** باستخدام `[(ngModel)]` ومعالجة حدث **Form Submission**. هل أنت مستعد للربط؟
```


>[!Tip]
>End

---


# Step 3: Handling Form Submission with `(ngSubmit)`

## 🎯 الغرض من هذه الخطوة (The Goal)

حالياً، الضغط على زر "Calculate" لا يفعل شيئاً (أو قد يعيد تحميل الصفحة).

الهدف هو:

1. استيراد `FormsModule` (لأنه المفتاح السحري لكل خصائص النماذج في Angular).
    
2. الاستماع لحدث `ngSubmit` على النموذج.
    
3. تنفيذ دالة `onSubmit` عند الإرسال.
    

---

## 💡 الشرح المفصل (The Logic)

### 1. دور `FormsModule`

حدث `(ngSubmit)` ليس حدثاً قياسياً في HTML، بل هو حدث خاص بـ Angular. لكي يفهم Angular هذا الحدث، يجب تفعيل `FormsModule` داخل المكون. بمجرد تفعيله، يقوم Angular تلقائياً بمنع السلوك الافتراضي للمتصفح (Prevent Default) عند الإرسال.

---

## 💻 التطبيق العملي (Step-by-Step)

### 1. تفعيل النماذج (User Input Component TS)

نستورد الموديول ونعرف دالة المعالجة.

**src/app/user-input/user-input.component.ts:**



```TypeScript
import { Component } from '@angular/core';
import { FormsModule } from '@angular/forms'; // 1. Import Module

@Component({
  selector: 'app-user-input',
  standalone: true,
  imports: [FormsModule], // 2. Add to imports array
  templateUrl: './user-input.component.html',
  styleUrl: './user-input.component.css'
})
export class UserInputComponent {
  
  // 3. دالة المعالجة (للتجربة حالياً)
  onSubmit() {
    console.log('Submitted!');
  }
}
```

### 2. الربط في القالب (User Input Component HTML)

نضيف `(ngSubmit)` للوسم `<form>`.

**تنبيه هام:** لكي يعمل زر الإرسال، تأكد من أنه **ليس** `type="button"`. يجب أن يكون `type="submit"` (أو بدون type لأن الافتراضي هو submit). في الدرس السابق وضعناه `button` لمنع التحديث، الآن نحتاج تغييره أو إزالته ليعمل كزناد (Trigger) للإرسال.

**src/app/user-input/user-input.component.html:**



```HTML
<form (ngSubmit)="onSubmit()">
  
  <p class="actions">
    <button type="submit">Calculate</button>
  </p>
</form>
```

---

## ✅ التحقق (Testing)

1. افتح أدوات المطور (Console).
    
2. اضغط على زر **Calculate**.
    
3. يجب أن ترى كلمة `Submitted!` تظهر في الكونسول دون أن يتم إعادة تحميل الصفحة.
    

**الخطوة التالية:** الآن بعد أن سيطرنا على حدث الإرسال، نحتاج لجمع البيانات المكتوبة في الحقول الأربعة لنقوم بالحساب عليها. سنستخدم **Two-Way Binding** لهذا الغرض. هل أنت مستعد؟

>[!Tip]
>End

---

# Step 4: Implementing Two-Way Binding & `ngModel`

## 🎯 الغرض من هذه الخطوة (The Goal)

نريد تحقيق تدفق بيانات ثنائي الاتجاه:

1. **TS -> HTML:** القيم الابتدائية (مثل 5% و 10 سنوات) تظهر تلقائياً في الحقول عند فتح الصفحة.
    
2. **HTML -> TS:** عندما يغير المستخدم الأرقام، تتحدث المتغيرات في الكلاس تلقائياً، لنتمكن من استخدامها في الحساب.
    

---

## 💡 الشرح المفصل (The Logic)

### 1. لماذا نستخدم String مع `type="number"`؟

رغم أننا حددنا `type="number"` في HTML، إلا أن قيمة حقل الإدخال (`input.value`) تعود دائماً كـ **نص (String)** في عالم الويب.

لذلك، من الأمان تعريف المتغيرات في TypeScript كـ `string` في البداية لتجنب مشاكل النوع، وسنقوم بتحويلها لأرقام (`+value`) فقط عند إجراء العمليات الحسابية.

### 2. أهمية الخاصية `name`

هذه نقطة يقع فيها الكثيرون:

- عند استخدام `[(ngModel)]` داخل وسم `<form>`، **يشترط** Angular وجود خاصية `name` لكل حقل.
    
- السبب: `FormsModule` يحتاج لاسم فريد لكل حقل لكي يدير حالته (Valid/Invalid/Touched) داخل النموذج. بدون الاسم، سيظهر خطأ في الكونسول.
    

---

## 💻 التطبيق العملي (Step-by-Step)

### 1. تعريف المتغيرات (User Input Component TS)

نضيف الخصائص بقيم ابتدائية (Defaults).

**src/app/user-input/user-input.component.ts:**



```TypeScript
export class UserInputComponent {
  // 1. تعريف المتغيرات بقيم نصية ابتدائية
  enteredInitialInvestment = '0';
  enteredAnnualInvestment = '0';
  enteredExpectedReturn = '5'; // قيمة افتراضية 5%
  enteredDuration = '10';      // قيمة افتراضية 10 سنوات

  onSubmit() {
    // 2. طباعة القيم للتأكد من الربط
    console.log('Submitted!');
    console.log(this.enteredInitialInvestment);
    console.log(this.enteredAnnualInvestment);
    console.log(this.enteredExpectedReturn);
    console.log(this.enteredDuration);
  }
}
```

### 2. الربط في القالب (User Input Component HTML)

نطبق `[(ngModel)]` ولا ننسى إضافة `name`.

**src/app/user-input/user-input.component.html:**



```HTML
<form (ngSubmit)="onSubmit()">
  <div class="input-group">
    <p>
      <label for="initial-investment">Initial Investment</label>
      <input 
        id="initial-investment" 
        type="number"
        name="initialInvestment" 
        [(ngModel)]="enteredInitialInvestment" 
      />
    </p>
    <p>
      <label for="annual-investment">Annual Investment</label>
      <input 
        id="annual-investment" 
        type="number"
        name="annualInvestment"
        [(ngModel)]="enteredAnnualInvestment" 
      />
    </p>
  </div>

  <div class="input-group">
    <p>
      <label for="expected-return">Expected Return</label>
      <input 
        id="expected-return" 
        type="number" 
        name="expectedReturn"
        [(ngModel)]="enteredExpectedReturn"
      />
    </p>
    <p>
      <label for="duration">Duration</label>
      <input 
        id="duration" 
        type="number" 
        name="duration"
        [(ngModel)]="enteredDuration"
      />
    </p>
  </div>
  </form>
```

---

## ✅ التحقق (Verification)

1. افتح التطبيق: ستجد أن حقلي "العائد" و"المدة" يحتويان بالفعل على **5** و **10** (نجاح الاتجاه الأول TS -> HTML).
    
2. غيّر القيم واضغط **Calculate**.
    
3. انظر للكونسول: ستجد القيم الجديدة التي أدخلتها مطبوعة (نجاح الاتجاه الثاني HTML -> TS).
    

**الخطوة التالية:** البيانات جاهزة في يدنا! الآن نحتاج لتمرير هذه البيانات إلى "مكان ما" ليتم حسابها وعرض النتائج. سنقوم بإنشاء **Service** لإدارة هذه الحسابات، لأن هذا هو المكان الأنسب للمنطق المعقد. هل أنت مستعد لإنشاء الخدمة؟


>[!Tip]
>End


---

# Step 5: Implementing Logic in `AppComponent`

## 🎯 الغرض من هذه الخطوة (The Goal)

نريد نقل كود الحسابات (الذي وفره لنا المشروع في ملف `investment-results.ts`) إلى داخل `AppComponent`.

لماذا `AppComponent`؟ لأنه يمثل "الأب" الذي يربط بين نموذج الإدخال (`UserInput`) وجدول النتائج (الذي سنبنيه لاحقاً). هذا النمط يسمى **Lifting State Up** (رفع الحالة للأعلى).

---

## 💡 الشرح المفصل (The Logic)

### 1. استقبال كائن واحد (Single Object Parameter)

بدلاً من استقبال 4 معاملات منفصلة (`val1, val2, val3, val4`) وهو ما قد يسبب ارتباكاً في الترتيب عند الاستدعاء، سنستقبل معاملاً واحداً اسمه `data`، ونحدد نوعه كـ **Object** يحتوي على الخصائص الأربعة المطلوبة.

### 2. التفكيك (Destructuring)

الكود الأصلي للحسابات يستخدم متغيرات بأسماء مباشرة (مثل `annualInvestment`). لكي لا نضطر لتغيير كل سطر في المعادلة إلى `data.annualInvestment`، سنستخدم ميزة **Destructuring** في JavaScript لاستخراج القيم من الكائن `data` وتخزينها في متغيرات بنفس الأسماء الأصلية.

---

## 💻 التطبيق العملي (Step-by-Step)

### 1. نقل الكود وتعديله (App Component TS)

نفتح ملف `app.component.ts` ونضيف الدالة التالية.

**src/app/app.component.ts:**



```TypeScript
import { Component } from '@angular/core';
import { HeaderComponent } from './header/header.component';
import { UserInputComponent } from './user-input/user-input.component';

@Component({
  selector: 'app-root',
  standalone: true,
  templateUrl: './app.component.html',
  styleUrl: './app.component.css',
  imports: [HeaderComponent, UserInputComponent],
})
export class AppComponent {
  
  // دالة الحساب (تم نسخ المنطق وتعديل المدخلات)
  calculateInvestmentResults(data: {
    initialInvestment: number;
    duration: number;
    expectedReturn: number;
    annualInvestment: number;
  }) {
    // 1. استخراج القيم من الكائن (Destructuring)
    const { initialInvestment, duration, expectedReturn, annualInvestment } = data;
    
    const annualData = [];
    let investmentValue = initialInvestment;

    // 2. منطق الحساب (كما هو من الملف المرفق)
    for (let i = 0; i < duration; i++) {
      const year = i + 1;
      const interestEarnedInYear = investmentValue * (expectedReturn / 100);
      investmentValue += interestEarnedInYear + annualInvestment;
      const totalInterest =
        investmentValue - annualInvestment * year - initialInvestment;
      
      annualData.push({
        year: year,
        interest: interestEarnedInYear,
        valueEndOfYear: investmentValue,
        annualInvestment: annualInvestment,
        totalInterest: totalInterest,
        totalAmountInvested: initialInvestment + annualInvestment * year,
      });
    }

    // 3. طباعة النتائج مؤقتاً للتأكد
    console.log(annualData); 
    // return annualData; // سنستخدم هذا لاحقاً
  }
}
```

---

## ⚠️ ملاحظات (Analysis)

1. **النوع (Type Safety):** لاحظ أننا حددنا نوع البيانات بوضوح (`number`) داخل تعريف الكائن `data`. هذا سيساعدنا لاحقاً لتجنب تمرير نصوص بالخطأ.
    
2. **الاستخدام:** هذه الدالة موجودة الآن في `AppComponent`، لكن لا أحد يناديها حتى الآن. المكون `UserInputComponent` هو من يملك البيانات، ولكنه لا يعرف بوجود هذه الدالة.
    

**الخطوة التالية:** نحتاج لإنشاء جسر تواصل. المكون `UserInputComponent` يجب أن يرسل البيانات للأعلى (للأب `AppComponent`) عندما يضغط المستخدم على زر "Calculate". سنستخدم **Custom Event (`@Output`)** لتحقيق ذلك. هل أنت مستعد لربط الأسلاك؟



>[!Tip]
>End

---


# Step 6: Passing Data Up via `@Output` & `EventEmitter`

## 🎯 الغرض من هذه الخطوة (The Goal)

المكون `UserInput` يمتلك البيانات (من النموذج)، والمكون `App` يمتلك منطق الحساب.

الهدف هو:

1. إنشاء حدث مخصص اسمه `calculate` في مكون الإدخال.
    
2. عند ضغط الزر، نطلق هذا الحدث ونرسل معه البيانات (بعد تحويلها لأرقام).
    
3. المكون الأب يستمع لهذا الحدث ويستلم البيانات ليقوم بالحساب.
    

---

## 💡 الشرح المفصل (The Logic)

### 1. التحويل للأرقام (Unary Plus Operator)

البيانات القادمة من `input` هي دائماً `string`، حتى لو كان النوع `number`.

لتحويل النص "100" إلى الرقم 100، نستخدم علامة الزائد (`+`) قبل المتغير: `+this.enteredDuration`. هذه أسرع طريقة في JavaScript.

### 2. تعريف نوع البيانات (Type Definition)

لكي نكون دقيقين (Type Safe)، سننسخ تعريف شكل الكائن (Object Shape) من `AppComponent` ونضعه في `EventEmitter` لنخبر Angular وشريكنا في الفريق: "هذا الحدث سيرسل كائناً يحتوي بالضبط على هذه الخصائص الأربعة".

---

## 💻 التطبيق العملي (Step-by-Step)

### 1. تجهيز المُرسِل (User Input Component TS)

نستورد `Output` و `EventEmitter` ونطلق الحدث.

**src/app/user-input/user-input.component.ts:**



```TypeScript
import { Component, Output, EventEmitter } from '@angular/core'; // 1. Import
import { FormsModule } from '@angular/forms';

@Component({
  selector: 'app-user-input',
  standalone: true,
  imports: [FormsModule],
  templateUrl: './user-input.component.html',
  styleUrl: './user-input.component.css',
})
export class UserInputComponent {
  // 2. تعريف الحدث المخصص
  @Output() calculate = new EventEmitter<{
    initialInvestment: number;
    duration: number;
    expectedReturn: number;
    annualInvestment: number;
  }>();

  enteredInitialInvestment = '0';
  enteredAnnualInvestment = '0';
  enteredExpectedReturn = '5';
  enteredDuration = '10';

  onSubmit() {
    // 3. إطلاق الحدث وإرسال البيانات (بعد التحويل لأرقام)
    this.calculate.emit({
      initialInvestment: +this.enteredInitialInvestment,
      annualInvestment: +this.enteredAnnualInvestment,
      expectedReturn: +this.enteredExpectedReturn,
      duration: +this.enteredDuration,
    });
  }
}
```

### 2. تجهيز المستقبِل (App Component HTML)

نربط الحدث `(calculate)` بالدالة في الأب.

**src/app/app.component.html:**



```HTML
<app-header />

<app-user-input (calculate)="onCalculateInvestmentResults($event)" />
```

### 3. تحديث المنطق (App Component TS)

نقوم بتغيير اسم الدالة (لتتبع معايير التسمية) وطباعة النتائج بدلاً من إرجاعها.

**src/app/app.component.ts:**



```TypeScript
export class AppComponent {
  // تم تغيير الاسم ليناسب الحدث (Convention: onEventName)
  onCalculateInvestmentResults(data: {
    initialInvestment: number;
    duration: number;
    expectedReturn: number;
    annualInvestment: number;
  }) {
    const { initialInvestment, duration, expectedReturn, annualInvestment } = data;
    const annualData = [];
    let investmentValue = initialInvestment;

    for (let i = 0; i < duration; i++) {
      const year = i + 1;
      const interestEarnedInYear = investmentValue * (expectedReturn / 100);
      investmentValue += interestEarnedInYear + annualInvestment;
      const totalInterest =
        investmentValue - annualInvestment * year - initialInvestment;
      annualData.push({
        year: year,
        interest: interestEarnedInYear,
        valueEndOfYear: investmentValue,
        annualInvestment: annualInvestment,
        totalInterest: totalInterest,
        totalAmountInvested: initialInvestment + annualInvestment * year,
      });
    }

    // الطباعة للتأكد
    console.log(annualData);
  }
}
```

---

## ✅ التحقق (Testing)

1. افتح المتصفح وأدخل قيماً في الحقول (مثلاً: 1000, 100, 5, 10).
    
2. اضغط **Calculate**.
    
3. افتح الكونسول: يجب أن ترى مصفوفة (Array) تحتوي على 10 عناصر (إذا كانت المدة 10 سنوات). كل عنصر يمثل بيانات سنة معينة.
    

**الخطوة التالية:** البيانات تم حسابها بنجاح ووصلت للأب! لكنها لا تزال "محبوسة" في الكونسول. الخطوة القادمة هي إنشاء المكون الثالث والأخير **`InvestmentResultsComponent`** لعرض هذه البيانات في جدول جميل للمستخدم. هل أنت مستعد لعرض النتائج؟


>[!Tip]
>End

---


# Step 7: Creating a Data Model (Interface)

## 🎯 الغرض من هذه الخطوة (The Goal)

لاحظنا أننا قمنا بنسخ ولصق تعريف نوع البيانات `{ initialInvestment: number, ... }` في مكانين (`AppComponent` و `UserInputComponent`).

هذا تكرار للكود (Code Duplication) وهو ممارسة سيئة.

الهدف هو:

1. إنشاء ملف واحد يحتوي على تعريف هذا النوع (Interface).
    
2. استيراد هذا النوع في أي مكان نحتاجه.
    
3. جعل الكود أنظف وأكثر قابلية للصيانة.
    

---

## 💡 الشرح المفصل (The Logic)

### 1. العقد (The Contract)

الـ **Interface** في TypeScript يعمل بمثابة "عقد". عندما نقول أن البيانات من نوع `InvestmentInput`، فنحن نضمن أن هذا الكائن يحتوي بالضبط على الخصائص الأربعة المطلوبة، وكلها أرقام.

### 2. Interface vs Type

ذكر المدرب أنه يمكن استخدام `interface` أو `type`. في حالتنا هذه، النتيجة واحدة تماماً. المطورون في Angular يميلون غالباً لاستخدام `interface` لتعريف أشكال الكائنات (Object Shapes).

---

## 💻 التطبيق العملي (Step-by-Step)

### 1. إنشاء ملف النموذج (Create Model File)

ننشئ ملفاً جديداً في المجلد الرئيسي `app`.

**src/app/investment-input.model.ts:**



```TypeScript
// تعريف الواجهة (Interface) وتصديرها لتكون متاحة للجميع
export interface InvestmentInput {
  initialInvestment: number;
  duration: number;
  expectedReturn: number;
  annualInvestment: number;
}
```

### 2. تحديث المكون الأب (App Component)

نستورد الـ Interface ونستخدمها بدلاً من الكتابة الطويلة.

**src/app/app.component.ts:**



```TypeScript
import { Component } from '@angular/core';
import { HeaderComponent } from './header/header.component';
import { UserInputComponent } from './user-input/user-input.component';
// 1. استيراد النوع
import type { InvestmentInput } from './investment-input.model';

@Component({ ... }) // (نفس الكود السابق)
export class AppComponent {
  
  // 2. استخدام النوع المختصر والنظيف
  onCalculateInvestmentResults(data: InvestmentInput) {
    const { initialInvestment, duration, expectedReturn, annualInvestment } = data;
    // ... باقي منطق الحساب كما هو ...
  }
}
```

### 3. تحديث المكون الابن (User Input Component)

نقوم بنفس الشيء في الـ `EventEmitter`.

**src/app/user-input/user-input.component.ts:**



```TypeScript
import { Component, Output, EventEmitter } from '@angular/core';
import { FormsModule } from '@angular/forms';
// 1. استيراد النوع
import type { InvestmentInput } from '../investment-input.model';

@Component({ ... }) // (نفس الكود السابق)
export class UserInputComponent {
  
  // 2. استخدام النوع داخل الأقواس الزاوية <>
  @Output() calculate = new EventEmitter<InvestmentInput>();

  // ... باقي المتغيرات والدوال كما هي ...
}
```

---

## ✅ الفائدة (The Benefit)

الآن، لو قررنا مستقبلاً تغيير اسم خاصية (مثلاً تغيير `duration` إلى `years`)، سنقوم بتغييرها في ملف واحد فقط (`investment-input.model.ts`)، وسيقوم TypeScript بتنبيهنا فوراً في جميع الملفات الأخرى التي تحتاج تعديل.

**الخطوة التالية:** الآن الكود نظيف وجاهز. حان الوقت للخطوة المنتظرة: إنشاء المكون الثالث **`InvestmentResultsComponent`** واستقبال البيانات المحسوبة لعرضها في الجدول الجميل. هل أنت مستعد لرؤية النتائج؟


>[!Tip]
>End

---

# Step 8: Displaying Results & The Input Decorator

سنقوم بتطبيق نمط تصميم مهم جداً في Angular يسمى **Data Down, Actions Up**:

- المكون `UserInput` يرسل الأحداث للأعلى (**Actions Up**).
    
- المكون `AppComponent` يعالج البيانات.
    
- المكون `InvestmentResults` يستقبل البيانات للأسفل (**Data Down**).
    

## 🎯 الغرض من هذه الخطوة (The Goal)

1. إنشاء مكون النتائج.
    
2. استقبال البيانات من الأب (`AppComponent`) باستخدام `@Input`.
    
3. تخزين النتائج في الأب بدلاً من طباعتها في الكونسول.
    

---

## 💻 التطبيق العملي (Step-by-Step)

### 1. إنشاء المكون (Generate Component)

نستخدم CLI لإنشاء المكون.



```Bash
ng g c investment-results --skip-tests
```

_(لا تنسَ نسخ كود CSS المرفق ولصقه في `investment-results.component.css` لتظهر الجداول بشكل جميل)._

### 2. استقبال البيانات (Investment Results TS)

سنستخدم `@Input` لاستقبال مصفوفة من النتائج. بما أن البيانات قد تكون غير موجودة في البداية (قبل الضغط على زر الحساب)، سنجعل المتغير اختيارياً باستخدام `?`.

**src/app/investment-results/investment-results.component.ts:**



```TypeScript
import { Component, Input } from '@angular/core';
import { CurrencyPipe } from '@angular/common'; // سنحتاجه لاحقاً للتنسيق

@Component({
  selector: 'app-investment-results',
  standalone: true,
  imports: [CurrencyPipe], 
  templateUrl: './investment-results.component.html',
  styleUrl: './investment-results.component.css'
})
export class InvestmentResultsComponent {
  // تعريف المدخلات: مصفوفة من الكائنات
  // علامة الاستفهام تعني أن القيمة قد تكون undefined في البداية
  @Input() results?: {
    year: number;
    interest: number;
    valueEndOfYear: number;
    annualInvestment: number;
    totalInterest: number;
    totalAmountInvested: number;
  }[];
}
```

### 3. تحديث الأب (App Component TS)

الآن `AppComponent` يحتاج لمكان لتخزين النتائج (`resultsData`) ليمررها لابنه الجديد.

**src/app/app.component.ts:**



```TypeScript
import { Component } from '@angular/core';
import { HeaderComponent } from './header/header.component';
import { UserInputComponent } from './user-input/user-input.component';
import { InvestmentResultsComponent } from './investment-results/investment-results.component'; // 1. استيراد المكون
import type { InvestmentInput } from './investment-input.model';

@Component({
  selector: 'app-root',
  standalone: true,
  templateUrl: './app.component.html',
  styleUrl: './app.component.css',
  imports: [HeaderComponent, UserInputComponent, InvestmentResultsComponent], // 2. إضافته للقائمة
})
export class AppComponent {
  // 3. متغير لتخزين النتائج (نفس نوع البيانات في الابن)
  resultsData?: {
    year: number;
    interest: number;
    valueEndOfYear: number;
    annualInvestment: number;
    totalInterest: number;
    totalAmountInvested: number;
  }[];

  onCalculateInvestmentResults(data: InvestmentInput) {
    const { initialInvestment, duration, expectedReturn, annualInvestment } = data;
    const annualData = [];
    let investmentValue = initialInvestment;

    for (let i = 0; i < duration; i++) {
      const year = i + 1;
      const interestEarnedInYear = investmentValue * (expectedReturn / 100);
      investmentValue += interestEarnedInYear + annualInvestment;
      const totalInterest =
        investmentValue - annualInvestment * year - initialInvestment;
      annualData.push({
        year: year,
        interest: interestEarnedInYear,
        valueEndOfYear: investmentValue,
        annualInvestment: annualInvestment,
        totalInterest: totalInterest,
        totalAmountInvested: initialInvestment + annualInvestment * year,
      });
    }

    // 4. تحديث المتغير بدلاً من الطباعة في الكونسول
    this.resultsData = annualData;
  }
}
```

### 4. الربط في القالب (App Component HTML)

نمرر البيانات من `resultsData` (في الأب) إلى `results` (في الابن).

**src/app/app.component.html:**



```HTML
<app-header />
<app-user-input (calculate)="onCalculateInvestmentResults($event)" />

<app-investment-results [results]="resultsData" />
```

---

## ⚠️ ملاحظة حول الأنواع (Type Definition)

لاحظ أننا قمنا بتعريف نوع البيانات `{ year: number, ... }[]` بشكل يدوي مرتين (مرة في الأب ومرة في الابن).

**أفضل ممارسة (Best Practice):** من الأفضل نقل هذا التعريف إلى ملف `investment-results.model.ts` واستيراده، تماماً كما فعلنا مع `Input Model` سابقاً، لكننا التزمنا بالشرح المبسط هنا.

**الخطوة التالية:** البيانات وصلت بنجاح إلى `InvestmentResultsComponent`. الآن بقي علينا تعديل ملف HTML الخاص بهذا المكون لعرض الجدول (Table) أو رسالة "أدخل البيانات" (Fallback Text) بناءً على وجود البيانات. هل أنت مستعد للخطوة الأخيرة في العرض؟

>[!Tip]
>End

---


# Step 9: Rendering the Table with `@if` & `@for`

## 🎯 الغرض من هذه الخطوة (The Goal)

1. **العرض الشرطي:** إذا لم تكن هناك نتائج (`!results`)، اعرض رسالة "Please enter some values". وإلا، اعرض الجدول.
    
2. **التكرار:** استخدام حلقة تكرار لإنشاء صف `<tr>` لكل سنة في مصفوفة النتائج.
    

---

## 💡 الشرح المفصل (The Logic)

### 1. الصيغة الحديثة (Modern Syntax)

بدلاً من التوجيهات الهيكلية القديمة (`*ngIf` و `*ngFor`)، Angular 17 قدمت صيغة أنظف وأسرع مدمجة في القالب:

- `@if (condition) { ... } @else { ... }`
    
- `@for (item of list; track item.id) { ... }`
    

### 2. خاصية `track`

في حلقة `@for`، يطلب Angular تحديد خاصية فريدة (`track result.year`) لتمييز العناصر. هذا يحسن الأداء بشكل هائل، حيث لا يضطر Angular لإعادة رسم القائمة بالكامل عند تغيير عنصر واحد فقط.

---

## 💻 التطبيق العملي (Step-by-Step)

نذهب إلى ملف القالب الخاص بمكون النتائج ونكتب الكود التالي.

**src/app/investment-results/investment-results.component.html:**



```HTML
@if (!results) {
  <p class="center">Please enter some values and press "Calculate".</p>
} @else {
  <table>
    <thead>
      <tr>
        <th>Year</th>
        <th>Investment Value</th>
        <th>Interest (Year)</th>
        <th>Total Interest</th>
        <th>Invested Capital</th>
      </tr>
    </thead>
    <tbody>
      @for (result of results; track result.year) {
        <tr>
          <td>{{ result.year }}</td>
          <td>{{ result.valueEndOfYear }}</td>
          <td>{{ result.interest }}</td>
          <td>{{ result.totalInterest }}</td>
          <td>{{ result.totalAmountInvested }}</td>
        </tr>
      }
    </tbody>
  </table>
}
```

---

## ✅ التحقق (Verification)

1. افتح التطبيق: ستظهر رسالة "Please enter some values..." (لأن `results` غير موجودة بعد).
    
2. أدخل قيماً واضغط **Calculate**.
    
3. يجب أن تختفي الرسالة ويظهر الجدول مليئاً بالأرقام.
    

**ملاحظة:** الأرقام ستظهر بشكل خام (مثلاً `1050.5555555`) وبدون تنسيق العملة. هذا طبيعي جداً في هذه المرحلة.

**الخطوة التالية:** الأرقام صحيحة لكنها قبيحة وصعبة القراءة. سنستخدم **Angular Pipes** لتنسيق هذه الأرقام كعملات (Currency) وجعل الجدول احترافياً. هل أنت مستعد للتجميل؟


>[!Tip]
>End


---

# Step 10: Formatting Data with `CurrencyPipe`

## 🎯 الغرض من هذه الخطوة (The Goal)

تحويل الأرقام الخام في الجدول إلى صيغة عملة مقروءة، مع إضافة الفواصل العشرية ورمز العملة.

---

## 💡 الشرح المفصل (The Logic)

### 1. ما هو Pipe (`|`)؟

هو أداة في Angular تأخذ بيانات كمدخل، وتقوم بتحويل شكلها للعرض فقط، دون تغيير القيمة الأصلية في البيانات.

- الصيغة: `{{ value | pipeName }}`
    
- الرمز `|` يسمى "Pipe Operator".
    

### 2. استيراد `CurrencyPipe`

هذا Pipe ليس متاحاً افتراضياً في القالب (في Standalone Components)، بل يجب استيراده من `@angular/common` وإضافته لمصفوفة `imports` في المكون.

---

## 💻 التطبيق العملي (Step-by-Step)

### 1. التأكد من الاستيراد (Investment Results TS)

_(قد نكون أضفناه في خطوة سابقة، لكن لنتأكد)_.

**src/app/investment-results/investment-results.component.ts:**



```TypeScript
import { Component, Input } from '@angular/core';
import { CurrencyPipe } from '@angular/common'; // 1. استيراد

@Component({
  selector: 'app-investment-results',
  standalone: true,
  imports: [CurrencyPipe], // 2. إضافته للمصفوفة
  // ...
})
export class InvestmentResultsComponent {
  // ...
}
```

### 2. تطبيق التنسيق (Investment Results HTML)

نضيف `| currency` لكل القيم المالية (ما عدا السنة).

**src/app/investment-results/investment-results.component.html:**



```HTML
<tr>
  <td>{{ result.year }}</td>
  <td>{{ result.valueEndOfYear | currency }}</td>
  <td>{{ result.interest | currency }}</td>
  <td>{{ result.totalInterest | currency }}</td>
  <td>{{ result.totalAmountInvested | currency }}</td>
</tr>
```

---

## 🎨 تخصيص العملة (Optional Configuration)

افتراضياً، يعرض Angular الدولار ($). إذا أردت تغييره لليورو أو أي عملة أخرى، يمكنك تمرير إعدادات للـ Pipe:



```HTML
<td>{{ result.valueEndOfYear | currency:'EUR' }}</td>
```

لكننا سنلتزم بالافتراضي كما فعل المدرب.

---

## ✅ النتيجة النهائية (Final Verification)

1. أدخل القيم: 1000, 100, 5, 10.
    
2. اضغط **Calculate**.
    
3. **النتيجة:** ستظهر الأرقام منسقة بشكل جميل (مثلاً `$1,050.00`).
    

**مبروك!** 🎉

لقد أنهيت التطبيق بنجاح باستخدام **النهج الأول** (Logic in App Component + Standalone Components).

التطبيق الآن يعمل بكفاءة، البيانات تتدفق من الابن للأب، ثم من الأب لابن آخر، ويتم عرضها وتنسيقها بشكل سليم.

**ماذا الآن؟**

المدرب ذكر أنه سيعيد بناء نفس التطبيق باستخدام **Service** (وهو الحل الأفضل والأكثر احترافية). هذا سينقلنا لمستوى أعلى في هندسة البرمجيات (Architecture). هل أنت مستعد للبدء في الـ **Refactoring** واستخدام الخدمات؟


>[!Tip]
>End

---


# Step 11: Refactoring to Signals

## 🎯 الغرض من هذه الخطوة (The Goal)

استبدال المتغيرات العادية (Variables) و `@Input/@Output` القديمة بنظام **Signals** الحديث.

- **الفائدة:** Signals تمنح Angular القدرة على معرفة "ما الذي تغير بالضبط" وتحديث ذلك الجزء فقط من الصفحة، بدلاً من فحص الصفحة بالكامل (Change Detection optimization).
    

---

## 💡 الشرح المفصل (The Logic)

### 1. `signal()` vs `variable`

- **المتغير العادي:** `data = 0;` (Angular لا يعرف متى يتغير إلا بفحص دوري).
    
- **الإشارة:** `data = signal(0);` (أنت تخبر Angular صراحة عند التغيير عبر `set` أو `update`).
    

### 2. `ngModel` و Signals

هذه نقطة سحرية في Angular:

- عند استخدام `[(ngModel)]="mySignal"`, **لا تحتاج** لاستدعاء الإشارة بالأقواس `mySignal()`.
    
- Angular ذكي بما يكفي ليعرف كيف يقرأ القيمة ويكتبها داخل الإشارة تلقائياً في النماذج.
    

---

## 💻 التطبيق العملي (Step-by-Step)

### 1. تحديث الحالة في الأب (App Component)

تحويل `resultsData` إلى إشارة.

**src/app/app.component.ts:**



```TypeScript
import { Component, signal } from '@angular/core'; // 1. استيراد signal
// ... imports

@Component({ ... })
export class AppComponent {
  // 2. تعريف الإشارة مع تحديد النوع (Array or undefined)
  resultsData = signal<{ year: number; ... }[] | undefined>(undefined);

  onCalculateInvestmentResults(data: InvestmentInput) {
    // ... حساب annualData ...

    // 3. تحديث القيمة باستخدام .set()
    this.resultsData.set(annualData);
  }
}
```

**src/app/app.component.html:**



```HTML
<app-investment-results [results]="resultsData()" />
```

---

### 2. تحديث المدخلات (User Input Component)

تحويل حقول الإدخال والـ `@Output` إلى النظام الحديث.

**src/app/user-input/user-input.component.ts:**



```TypeScript
import { Component, output, signal } from '@angular/core'; // لاحظ: output بدالة صغيرة (v17.3+)

@Component({ ... })
export class UserInputComponent {
  // 1. تحويل المخرجات للدالة الحديثة output()
  calculate = output<InvestmentInput>();

  // 2. تحويل الخصائص إلى Signals (يستنتج النوع string تلقائياً)
  enteredInitialInvestment = signal('0');
  enteredAnnualInvestment = signal('0');
  enteredExpectedReturn = signal('5');
  enteredDuration = signal('10');

  onSubmit() {
    // 3. قراءة القيم بالأقواس () عند الإرسال
    this.calculate.emit({
      initialInvestment: +this.enteredInitialInvestment(),
      annualInvestment: +this.enteredAnnualInvestment(),
      expectedReturn: +this.enteredExpectedReturn(),
      duration: +this.enteredDuration(),
    });

    // 4. إعادة تعيين النموذج (Reset Form)
    this.enteredInitialInvestment.set('0');
    this.enteredAnnualInvestment.set('0');
    this.enteredExpectedReturn.set('5');
    this.enteredDuration.set('10');
  }
}
```

**تنبيه هام:** في ملف `user-input.component.html`، اترك `[(ngModel)]="enteredInitialInvestment"` كما هي **بدون أقواس**.

---

### 3. تحديث المستقبل (Investment Results Component)

تحويل `@Input` إلى `input()`.

**src/app/investment-results/investment-results.component.ts:**



```TypeScript
import { Component, input } from '@angular/core'; // لاحظ: input بدالة صغيرة
// ...

@Component({ ... })
export class InvestmentResultsComponent {
  // تعريف المدخل كإشارة (Signal Input)
  // Angular يجعلها Read-only تلقائياً
  results = input<{ year: number; ... }[]>(); 
}
```

**src/app/investment-results/investment-results.component.html:**



```HTML
@if (!results()) { 
  <p>Please enter some values...</p>
} @else {
  @for (result of results(); track result.year) { ... }
}
```

---

## 🔍 الخلاصة (Recap)

بهذا التحويل، أصبح تطبيقك **Reactive** بالكامل:

1. **State:** تدار بواسطة `signal()`.
    
2. **Inputs:** تدار بواسطة `input()`.
    
3. **Outputs:** تدار بواسطة `output()`.
    

هذا هو المعيار الذهبي (Gold Standard) لتطبيقات Angular الحديثة في 2024 وما بعدها.

**الخطوة التالية:** المدرب ذكر خياراً آخر وهو استخدام **Service**. هذا سينقل منطق الحساب من `AppComponent` (الذي لا يجب أن يهتم بالحسابات) إلى خدمة متخصصة، مما يجعل الكود أنظف وأكثر احترافية. هل أنت مستعد لفصل المنطق؟


>[!Tip]
>End

---


# Step 12: Implementing a Service (Centralized State)

## 🎯 الغرض من هذه الخطوة (The Goal)

حالياً، البيانات تسافر في رحلة طويلة: `UserInput` -> `App` -> `InvestmentResults`.

هذا يجعل `AppComponent` مزدحماً بكود لا علاقة له به.

الهدف هو:

1. نقل منطق الحساب والبيانات إلى `InvestmentService`.
    
2. جعل `UserInput` يرسل البيانات للخدمة مباشرة.
    
3. جعل `InvestmentResults` يقرأ البيانات من الخدمة مباشرة.
    
4. تنظيف `AppComponent` ليصبح مجرد حاوية.
    

---

## 💻 التطبيق العملي (Step-by-Step)

### 1. إنشاء الخدمة (Create Service)

أنشئ ملفاً جديداً `investment.service.ts` (أو استخدم CLI: `ng g s investment`).

**src/app/investment.service.ts:**



```TypeScript
import { Injectable } from '@angular/core';
import type { InvestmentInput } from './investment-input.model';

@Injectable({ providedIn: 'root' }) // يجعل الخدمة متاحة في التطبيق كله (Singleton)
export class InvestmentService {
  // 1. خاصية لتخزين النتائج (State)
  resultData?: {
    year: number;
    interest: number;
    valueEndOfYear: number;
    annualInvestment: number;
    totalInterest: number;
    totalAmountInvested: number;
  }[];

  // 2. نقل دالة الحساب هنا
  calculateInvestmentResults(data: InvestmentInput) {
    const { initialInvestment, duration, expectedReturn, annualInvestment } = data;
    const annualData = [];
    let investmentValue = initialInvestment;

    for (let i = 0; i < duration; i++) {
      const year = i + 1;
      const interestEarnedInYear = investmentValue * (expectedReturn / 100);
      investmentValue += interestEarnedInYear + annualInvestment;
      const totalInterest =
        investmentValue - annualInvestment * year - initialInvestment;
      annualData.push({
        year: year,
        interest: interestEarnedInYear,
        valueEndOfYear: investmentValue,
        annualInvestment: annualInvestment,
        totalInterest: totalInterest,
        totalAmountInvested: initialInvestment + annualInvestment * year,
      });
    }

    // تخزين النتائج في الخاصية
    this.resultData = annualData;
  }
}
```

---

### 2. تحديث المكون المُدخل (User Input Component)

سنستخدم **Constructor Injection** هنا (كأحد أساليب الحقن).

**src/app/user-input/user-input.component.ts:**



```TypeScript
import { Component, signal } from '@angular/core';
import { InvestmentService } from '../investment.service'; // استيراد الخدمة

@Component({ ... })
export class UserInputComponent {
  // تم حذف @Output لأننا لن نرسل للأب بعد الآن

  // 1. حقن الخدمة عبر الـ Constructor
  constructor(private investmentService: InvestmentService) {}

  enteredInitialInvestment = signal('0');
  enteredAnnualInvestment = signal('0');
  enteredExpectedReturn = signal('5');
  enteredDuration = signal('10');

  onSubmit() {
    // 2. الاتصال بالخدمة مباشرة
    this.investmentService.calculateInvestmentResults({
      initialInvestment: +this.enteredInitialInvestment(),
      annualInvestment: +this.enteredAnnualInvestment(),
      expectedReturn: +this.enteredExpectedReturn(),
      duration: +this.enteredDuration(),
    });
    
    // ... Reset signals logic
  }
}
```

---

### 3. تحديث مكون النتائج (Investment Results Component)

سنستخدم دالة **`inject()`** هنا (كالأسلوب الثاني للحقن). وسنستخدم **Getter** لقراءة البيانات.

**src/app/investment-results/investment-results.component.ts:**



```TypeScript
import { Component, inject } from '@angular/core'; // استيراد inject
import { InvestmentService } from '../investment.service';

@Component({ ... })
export class InvestmentResultsComponent {
  // تم حذف @Input لأننا سنقرأ من الخدمة

  // 1. حقن الخدمة
  private investmentService = inject(InvestmentService);

  // 2. Getter لقراءة البيانات وعرضها في القالب
  // هذا يسمح للقالب بالوصول لبيانات الخدمة الخاصة
  get results() {
    return this.investmentService.resultData;
  }
}
```

**تعديل القالب (HTML):**

تأكد من إزالة الأقواس `()` من `results` لأننا عدنا لاستخدام خاصية عادية (Getter) وليست Signal في الوقت الحالي.



````HTML
@if (!results) { ... } @for (result of results; track result.year) { ... } ```

---

### 4. تنظيف المكون الأب (App Component)
الآن `AppComponent` أصبح نظيفاً تماماً.

**src/app/app.component.ts:**
احذف دالة الحساب ومتغير `resultsData`.

**src/app/app.component.html:**
```html
<app-header />
<app-user-input />
<app-investment-results />
````

---

## ✅ النتيجة (The Result)

لقد قمت بفصل **المنطق (Logic)** عن **العرض (UI)**.

- المكونات (`UserInput`, `InvestmentResults`) تهتم فقط بالشكل.
    
- الخدمة (`InvestmentService`) تهتم بالحسابات وتخزين البيانات.
    
    هذا هو الهيكل المثالي لأي تطبيق Angular قابل للتوسع.
    

**الخطوة التالية:** التطبيق الآن يعمل بشكل ممتاز باستخدام Service ومتغيرات عادية. هل تذكر كيف حولنا المتغيرات لـ **Signals** سابقاً؟ يمكننا فعل نفس الشيء داخل الـ Service لجعلها Reactive بالكامل. هل تريد أن نرى كيف ندمج **Services + Signals**؟



>[!Tip]
>End


---

# Step 13: Signal-Based Service & Read-Only Protection

## 🎯 الغرض من هذه الخطوة (The Goal)

1. تحويل خاصية `resultData` داخل الخدمة من متغير عادي إلى `signal`.
    
2. تحديث منطق الحساب لاستخدام `.set()`.
    
3. **الحماية (Encapsulation):** منع المكونات الخارجية من تعديل الإشارة مباشرة باستخدام `computed` أو `asReadonly`.
    

---

## 💡 الشرح المفصل (The Logic)

### 1. لماذا Read-Only؟

عندما نكشف `signal` من الخدمة للمكونات:

- **الخطر:** المكون قد يقوم باستدعاء `.set()` ويغير البيانات في الخدمة، وهذا يكسر قاعدة "الخدمة هي المصدر الوحيد للحقيقة" (Single Source of Truth).
    
- **الحل:** نكشف للمكونات نسخة "للقراءة فقط".
    
    - الطريقة 1: `computed(() => signal())`
        
    - الطريقة 2: `signal.asReadonly()`
        

---

## 💻 التطبيق العملي (Step-by-Step)

### 1. تحديث الخدمة (Investment Service)

نحول الخاصية إلى إشارة ونحدث دالة الحساب.

**src/app/investment.service.ts:**



```TypeScript
import { Injectable, signal } from '@angular/core'; // 1. استيراد signal
import type { InvestmentInput } from './investment-input.model';

@Injectable({ providedIn: 'root' })
export class InvestmentService {
  // 2. تعريف الإشارة مع تحديد النوع (Array or undefined)
  // لاحظ حذفنا علامة الاستفهام (?) لأن الإشارة موجودة دائماً وقيمتها هي undefined
  resultData = signal<{
    year: number;
    interest: number;
    valueEndOfYear: number;
    annualInvestment: number;
    totalInterest: number;
    totalAmountInvested: number;
  }[] | undefined>(undefined);

  calculateInvestmentResults(data: InvestmentInput) {
    const { initialInvestment, duration, expectedReturn, annualInvestment } = data;
    const annualData = [];
    let investmentValue = initialInvestment;

    for (let i = 0; i < duration; i++) {
      // ... (نفس منطق الحساب السابق تماماً) ...
      // ...
      // annualData.push({...});
    }

    // 3. تحديث الإشارة
    // بدلاً من this.resultData = annualData
    this.resultData.set(annualData);
  }
}
```

---

### 2. تحديث مكون النتائج (Investment Results Component)

هنا سنطبق الحماية. نريد قراءة الإشارة من الخدمة ولكن دون السماح للمكون بتعديلها.

**src/app/investment-results/investment-results.component.ts:**



```TypeScript
import { Component, inject, computed } from '@angular/core'; // 1. استيراد computed
import { CurrencyPipe } from '@angular/common';
import { InvestmentService } from '../investment.service';

@Component({ ... })
export class InvestmentResultsComponent {
  private investmentService = inject(InvestmentService);

  // 2. استخدام computed لإنشاء إشارة مشتقة (Read-Only)
  // أي تغيير في الخدمة سيؤدي لتحديث هذه الإشارة تلقائياً
  results = computed(() => this.investmentService.resultData());
  
  /* 3. بديل آخر ذكره المدرب (أقصر وأنظف):
     results = this.investmentService.resultData.asReadonly();
  */
}
```

### 3. تحديث القالب (HTML Template)

بما أن `results` أصبحت الآن `Signal` (سواء عبر computed أو asReadonly)، يجب استدعاؤها كدالة.

**src/app/investment-results/investment-results.component.html:**



```HTML
@if (!results()) {
  <p class="center">Please enter some values...</p>
} @else {
  <table>
    <thead>...</thead>
    <tbody>
      @for (result of results(); track result.year) {
        <tr>
          <td>{{ result.year }}</td>
          <td>{{ result.valueEndOfYear | currency }}</td>
          </tr>
      }
    </tbody>
  </table>
}
```

>[!Tip]
>End


---

# Step 14: Migrating from Standalone to NgModules

## 🎯 الغرض من هذه الخطوة (The Goal)

تحويل هيكلية التطبيق بالكامل:

1. تجريد المكونات من استقلاليتها (إزالة `standalone: true`).
    
2. إنشاء `AppModule` ليكون "المدير المركزي" للتطبيق.
    
3. تغيير طريقة تشغيل التطبيق في `main.ts`.
    
4. (إضافي) إنشاء `UserInputModule` لتعلم كيفية تقسيم الموديولات.
    

---

## 💻 المرحلة الأولى: التنظيف (Cleanup Phase)

يجب الذهاب لجميع المكونات الأربعة (`App`, `Header`, `UserInput`, `InvestmentResults`) وإزالة كل ما يتعلق بـ Standalone.

### 1. تنظيف المكونات (All Components)

قم بتطبيق هذا التغيير على الملفات الأربعة:

- احذف `standalone: true`.
    
- احذف مصفوفة `imports: [...]`.
    
- (تنبيه: إذا كنت تستخدم Angular 19+، تذكر إضافة `standalone: false`).
    

**مثال على `app.component.ts` بعد التنظيف:**



```TypeScript
import { Component } from '@angular/core';
// لاحظ: لا توجد imports للمكونات هنا

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrl: './app.component.css',
  // standalone: false // (فقط لنسخ Angular 19+)
})
export class AppComponent {}
```

_(كرر نفس العملية لـ Header, UserInput, InvestmentResults)_.

---

## 💻 المرحلة الثانية: المدير المركزي (AppModule)

### 2. إنشاء `app.module.ts`

أنشئ الملف في مجلد `src/app`. هذا الملف سيجمع شتات المكونات التي فقدت استقلاليتها.

**src/app/app.module.ts:**



```TypeScript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser'; // أساسي لأي تطبيق ويب
import { FormsModule } from '@angular/forms'; // نحتاجه لـ UserInput

import { AppComponent } from './app.component';
import { HeaderComponent } from './header/header.component';
import { UserInputComponent } from './user-input/user-input.component';
import { InvestmentResultsComponent } from './investment-results/investment-results.component';

@NgModule({
  declarations: [
    // تسجيل جميع المكونات هنا
    AppComponent,
    HeaderComponent,
    UserInputComponent,
    InvestmentResultsComponent
  ],
  imports: [
    // استيراد الموديولات الخارجية
    BrowserModule, // يوفر CommonModule (Pipes, ngIf, ngFor)
    FormsModule    // يوفر ngModel, ngSubmit
  ],
  bootstrap: [AppComponent] // نقطة الانطلاق
})
export class AppModule {}
```

---

## 💻 المرحلة الثالثة: التشغيل (Bootstrapping)

### 3. تعديل `main.ts`

يجب تغيير المحرك ليقوم بتشغيل الموديول بدلاً من المكون.

**src/main.ts:**



```TypeScript
import { platformBrowserDynamic } from '@angular/platform-browser-dynamic';
import { AppModule } from './app/app.module';

platformBrowserDynamic().bootstrapModule(AppModule)
  .catch((err) => console.error(err));
```

---

## 💻 المرحلة الرابعة: التجزئة (Modularization)

لنتخيل أن `UserInput` ميزة كبيرة ونريد عزلها في موديول خاص (Feature Module).

### 4. إنشاء `user-input.module.ts`

في المجلد `src/app/user-input`.



```TypeScript
import { NgModule } from '@angular/core';
import { FormsModule } from '@angular/forms'; // انتقل هنا لأنه مستخدم هنا فقط
import { UserInputComponent } from './user-input.component';

@NgModule({
  declarations: [UserInputComponent], // المكون "ملك" لهذا الموديول
  imports: [FormsModule],
  exports: [UserInputComponent] // ⚠️ هام جداً: لتتمكن الموديولات الخارجية من استخدامه
})
export class UserInputModule {}
```

### 5. تحديث `app.module.ts`

الآن نحذف `UserInput` و `FormsModule` من الموديول الرئيسي ونستورد الموديول الفرعي بدلاً منهم.

**src/app/app.module.ts (المحدث):**



```TypeScript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { UserInputModule } from './user-input/user-input.module'; // استيراد الموديول الفرعي

import { AppComponent } from './app.component';
import { HeaderComponent } from './header/header.component';
import { InvestmentResultsComponent } from './investment-results/investment-results.component';

@NgModule({
  declarations: [
    AppComponent,
    HeaderComponent,
    InvestmentResultsComponent
    // تم حذف UserInputComponent من هنا
  ],
  imports: [
    BrowserModule,
    UserInputModule // استيراد الموديول بدلاً من المكون
    // تم حذف FormsModule من هنا
  ],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

>[!Tip]
>End

---
