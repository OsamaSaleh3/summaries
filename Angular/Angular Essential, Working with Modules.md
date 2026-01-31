

**لقد تعلمت في القسم السابق الطريقة "الحديثة" (Modern Way) باستخدام Standalone Components، وهي الطريقة الموصى بها حالياً. لكن، لكي تكون مطور Angular محترفاً، يجب أن تفهم الطريقة "التقليدية" (Legacy Way) التي بنيت عليها Angular منذ عام 2016، وهي Angular Modules.**


# Angular Modules (NgModules) vs. Standalone Components

## 🎯 الغرض من هذا القسم (The Goal)

في العالم الواقعي، ستعمل في شركات لديها مشاريع ضخمة بنيت قبل سنوات (قبل ظهور Standalone في Angular 14/15). هذه المشاريع تعتمد كلياً على **Modules**.

الهدف هو أن تفهم:

1. ما هي الـ Modules؟
    
2. كيف نجمع المكونات (Components) داخل "صناديق" تنظيمية.
    
3. كيف ننتقل بين الطريقتين (Standalone vs NgModule).
    

---

## 💡 الشرح النظري (The Concept)

### 1. الفرق الجوهري (The Core Difference)

- **Standalone Components (ما تعلمته):**
    
    - كل مكون هو "سيد نفسه".
        
    - يحمل حقيبته الخاصة (`imports: []`) ويستورد ما يحتاجه مباشرة.
        
    - سهل وسريع، مثل بناء الليجو (Lego) قطعة بقطعة.
        
- **NgModules (ما ستتعلمه):**
    
    - المكونات "خجولة" ولا تعرف كيف تستورد الأشياء بنفسها.
        
    - يجب وضع المكونات داخل "صندوق كبير" يسمى **Module**.
        
    - هذا الصندوق هو المسؤول عن استيراد الأدوات وتوزيعها على المكونات التي بداخله.
        

### 2. لماذا نتعلمها الآن؟ (Why Now?)

- **التاريخ:** من 2016 حتى 2022، كانت الـ Modules هي الطريقة _الوحيدة_ لبناء تطبيقات Angular.
    
- **سوق العمل:** 90% من المشاريع الحالية في السوق تستخدم Modules.
    
- **الهيكلة:** في التطبيقات العملاقة جداً، الـ Modules تساعد في تقسيم التطبيق إلى "مكعبات" كبيرة (مثل UserModule, AdminModule).
    

---

## 🗺️ خارطة الطريق لهذا القسم

سنتعلم المفاهيم التالية التي تحل محل ما تعلمناه سابقاً:

1. **Declarations:** بدلاً من `imports` في المكون، سنعلن عن المكونات في الموديول.
    
2. **Exports:** كيف نجعل المكونات مرئية خارج الصندوق (Module).
    
3. **Bootstrap:** كيف يختلف ملف تشغيل التطبيق (`main.ts`).
    
4. **Refactoring:** سنقوم بتحويل تطبيق "إدارة المهام" الذي بنيناه من Standalone إلى Modules كتدريب عملي.


>[!tip]
>END



---



# Introduction to Angular Modules (NgModules)

## 🎯 الغرض من هذا الدرس (The Goal)

فهم الفلسفة المعمارية القديمة (والتي لا تزال مستخدمة بكثرة) في Angular.

- **الوضع الحالي (Standalone):** كل مكون مسؤول عن نفسه، ويستورد ما يحتاجه في مصفوفة `imports`.
    
- **الوضع الجديد (Modules):** المكونات يتم تجميعها في "وحدات" (Modules). الوحدة هي المسؤولة عن تعريف المكونات وجلب الاعتمادات لها.
    

---

## 💡 الفرق الجوهري (Standalone vs. Modules)

### 1. نهج المكون المستقل (Standalone Approach)

- **المكان:** الاعتمادات تُعرف داخل الـ `@Component` decorator.
    
- **الشفافية:** عندما تفتح ملف المكون، ترى بوضوح ما يعتمد عليه (مثل `CardComponent`, `DatePipe`).
    
- **الكود:**
    
    
    
    ```TypeScript
    @Component({
      standalone: true,
      imports: [CardComponent, DatePipe, ...], // الاستيراد هنا
      ...
    })
    export class TaskComponent {}
    ```
    

### 2. نهج الوحدات (NgModule Approach)

- **المكان:** الاعتمادات تُعرف داخل ملف منفصل يسمى `Module` (مثل `app.module.ts`).
    
- **التجريد:** المكون يصبح "أخف" (Leaner)، حيث تختفي مصفوفة `imports` منه، لكنك تفقد الوضوح المباشر (لا تعرف ماذا يستخدم المكون إلا بالرجوع للموديول).
    
- **الكود:**
    
    
    
    ```TypeScript
    // في ملف المكون (يصبح فارغاً من الاستيرادات)
    @Component({
      // لا يوجد standalone: true
      // لا يوجد imports: []
      ...
    })
    export class TaskComponent {}
    
    // في ملف الموديول (يجمع الكل)
    @NgModule({
      declarations: [TaskComponent, ...], // تعريف المكونات
      imports: [ ... ] // استيراد الموديولات الأخرى
    })
    export class AppModule {}
    ```
    

---

## ⚖️ المميزات والعيوب (Trade-offs)

|**الميزة/العيب**|**Standalone Components (Modern)**|**NgModules (Classic)**|
|---|---|---|
|**حجم الكود (Boilerplate)**|أقل (لا توجد ملفات modules إضافية).|أكثر (تحتاج إنشاء ملفات `.module.ts`).|
|**وضوح الاعتمادات**|عالي جداً (Explicit).|منخفض (Implicit).|
|**تنظيم المكونات**|كل مكون لوحده.|تجميع المكونات المترابطة في حزمة واحدة.|
|**ديكور المكون (Decorator)**|مزدحم قليلاً بـ `imports`.|نظيف وخفيف (Lean).|

---

## 🚀 خطة العمل (Migration Plan)

في الدروس القادمة، سنقوم بالخطوات التالية لتحويل تطبيق "إدارة المهام":

1. **إزالة `standalone: true`:** من كل المكونات.
    
2. **إنشاء `AppModule`:** أول موديول رئيسي للتطبيق.
    
3. **التعريف (`declarations`):** تسجيل المكونات داخل الموديول.
    
4. **تغيير نقطة الانطلاق (`Bootstrap`):** تعديل `main.ts` ليقوم بتشغيل الموديول بدلاً من المكون.



>[!tip]
>END

 

---



# Critical Update: Angular 19+ & `standalone: false`

## ⚠️ تنبيه هام لمستخدمي Angular 19+

في السابق (Angular 18 وما قبلها)، إذا حذفت سطر `standalone: true`، كان Angular يفترض تلقائياً أن المكون يتبع نظام الـ Modules.
**في Angular 19، انقلبت الآية!** إذا لم تكتب شيئاً، سيفترض Angular أن المكون **Standalone**.

### ماذا يعني هذا بالنسبة لك في هذا القسم؟

عندما نقوم بتحويل المكونات في الدروس القادمة لاستخدام `AppModule`:

1. **إذا كنت تستخدم Angular < 19:**
* يكفي أن **تحذف** سطر `standalone: true`.


2. **إذا كنت تستخدم Angular 19+ (وهذا المرجح):**
* يجب أن **تكتب صراحة**: `standalone: false`.



---

## 💻 مقارنة الكود (Code Comparison)

إليك كيف سيبدو كود المكون (Component Decorator) في الحالتين عند استخدام Modules:

### Angular 18 (وأسفل)

```typescript
@Component({
  selector: 'app-user',
  templateUrl: './user.component.html',
  styleUrl: './user.component.css',
  // حذفنا standalone: true، والوضع الافتراضي هو Module-based
})
export class UserComponent {}

```

### Angular 19+ (الوضع الجديد)

```typescript
@Component({
  selector: 'app-user',
  templateUrl: './user.component.html',
  styleUrl: './user.component.css',
  standalone: false // ⚠️ إجباري لإلغاء الوضع الافتراضي الجديد
})
export class UserComponent {}

```

---

## ✅ الخطوة العملية (Check your version)

قبل أن نبدأ التعديل في الدرس القادم، ألقِ نظرة سريعة على ملف `package.json` في مشروعك:

* ابحث عن `"@angular/core"`.
* إذا كان الرقم يبدأ بـ `^19.0.0`، فتذكر استخدام `standalone: false` دائماً في هذا القسم.
  
  
>[!tip]
>END


---

إ


# Creating the Root Module: `app.module.ts`

## 🎯 الغرض من هذا الدرس (The Goal)

في نظام Standalone، كان الملف `main.ts` يقوم بتشغيل `AppComponent` مباشرة.

في نظام Modules، نحتاج لوسيط. يجب إنشاء **Root Module** (عادة يسمى `AppModule`) ليقوم بتجميع كل أجزاء التطبيق، ثم يقوم Angular بتشغيل هذا الموديول.

---

## 💡 الشرح المفصل (The Logic)

### 1. ما هو `AppModule`؟

هو كلاس TypeScript عادي، ولكنه مزين بـ Decorator خاص اسمه `@NgModule`.

- **وظيفته:** يخبر Angular عن "مكونات" التطبيق وكيف ترتبط ببعضها.
    
- **المكان:** ينشأ بجانب `app.component.ts`.
    

### 2. مصفوفة `declarations`

هذا هو الجزء الأهم في هذا الدرس.

- **المعنى:** "التصريحات" أو "السجل المدني".
    
- **القاعدة:** أي Component (أو Directive أو Pipe) ليس `standalone`، **يجب** أن يتم تسجيل اسمه في هذه المصفوفة. إذا لم تسجله، فكأنك لم تنشئه، وسيظهر لك خطأ: `'app-user' is not a known element`.
    

---

## 💻 التطبيق العملي (Step-by-Step)

### 1. إنشاء الملف

قم بإنشاء ملف جديد في المسار: `src/app/app.module.ts`.

### 2. كتابة الكود الأساسي

سنقوم بإنشاء الكلاس وإعداد المصفوفة (سنملؤها في الدروس القادمة).

**src/app/app.module.ts:**



```TypeScript
import { NgModule } from '@angular/core'; // 1. استيراد Decorator

// 2. تزيين الكلاس
@NgModule({
  // 3. مصفوفة التصريحات: هنا سنسجل كل مكوناتنا
  declarations: [
    // AppComponent,
    // HeaderComponent,
    // UserComponent,
    // ...
  ],
  // سنحتاج لاحقاً لمصفوفة imports ومصفوفة bootstrap
})
export class AppModule {} // 4. تصدير الكلاس
```

---

## ⚠️ ملاحظات للمستقبل (Avoid These Mistakes)

> [!WARNING] قاعدة الـ Declarations
> 
> المكون يمكن أن ينتمي لموديول **واحد فقط**. لا يمكنك وضع `UserComponent` في `AppModule` وفي موديول آخر في نفس الوقت.

> [!TIP] الترتيب
> 
> رغم أننا أنشأنا الملف، إلا أن التطبيق لن يعمل بعد لأننا لم نقم بتعديل المكونات لتكون `standalone: false`، ولم نقم بربط الموديول بملف `main.ts`. هذه هي الخطوات القادمة.

**الخطوة التالية:** الآن الهيكل جاهز (`AppModule`). الخطوة القادمة هي الذهاب لجميع مكوناتنا القديمة (`AppComponent`, `Header`, `User`...) وتحويلها إلى **Non-Standalone** وتسجيلها داخل مصفوفة `declarations`.

---


# Bootstrapping: Wiring Up the Root Module

## 🎯 الغرض من هذا الدرس (The Goal)

لدينا `AppModule`، لكن التطبيق لا يزال يحاول العمل بالنظام القديم (Standalone) وسيفشل.

الهدف هو:

1. تحويل `AppComponent` من Standalone إلى Module-based.
    
2. تغيير ملف الإقلاع `main.ts` ليشغل الموديول بدلاً من المكون.
    
3. تحديد نقطة البداية (Entry Component) داخل الموديول.
    

---

## 💡 الشرح المفصل (The Logic)

### 1. تحرير المكون (The Component Transformation)

المكون لا يمكنه أن يكون Standalone ومسجلاً في Module في نفس الوقت.

- **الحذف:** نزيل `standalone: true` (أو نضعها `false` في Angular 19+).
    
- **التنظيف:** نزيل مصفوفة `imports` تماماً من المكون، لأن الموديول هو من سيتولى هذه المهمة الآن.
    

### 2. تغيير عملية الإقلاع (Main.ts Refactoring)

- **سابقاً:** `bootstrapApplication(AppComponent)` -> تشغيل مباشر للمكون.
    
- **الآن:** `platformBrowserDynamic().bootstrapModule(AppModule)` -> تشغيل الموديول، والموديول بدوره يشغل المكون.
    
- **لماذا `platformBrowserDynamic`؟** لأن الموديولات تحتاج عادةً إلى تجميع (Compilation) داخل المتصفح (JIT) في بيئة التطوير.
    

### 3. مصفوفة `bootstrap` في الموديول

عندما يشتغل `AppModule`، كيف يعرف أي مكون يجب أن يضعه في الصفحة أولاً؟

- نحتاج لإضافة خاصية `bootstrap: [AppComponent]` في الموديول. هذا يخبر Angular: "عندما يعمل هذا الموديول، ابدأ فوراً بعرض هذا المكون".
    

---

## 💻 التطبيق العملي (Step-by-Step)

### 1. تعديل المكون الجذري (App Component)

نذهب إلى `app.component.ts` ونزيل خصائص الـ Standalone.

**src/app/app.component.ts:**



```TypeScript
import { Component } from '@angular/core';
// لاحظ: لا توجد استيرادات للمكونات الأخرى هنا

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrl: './app.component.css',
  standalone: false // ⚠️ ضروري لـ Angular 19+
  // imports: [] <-- تم حذفها بالكامل
})
export class AppComponent {
  // Logic...
}
```

### 2. تعديل الموديول (App Module)

نضيف المكون في `declarations` ونحدده في `bootstrap`.

**src/app/app.module.ts:**



```TypeScript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser'; // ضروري للمتصفح
import { AppComponent } from './app.component';

@NgModule({
  declarations: [
    AppComponent // 1. تعريف المكون
  ],
  imports: [
    BrowserModule // 2. وحدة المتصفح الأساسية (ضرورية لعمل ngIf, ngFor في البداية)
  ],
  bootstrap: [AppComponent] // 3. نقطة انطلاق التطبيق
})
export class AppModule {}
```

### 3. تعديل ملف التشغيل (Main.ts)

نغير طريقة الإقلاع بالكامل.

**src/main.ts:**



```TypeScript
import { platformBrowserDynamic } from '@angular/platform-browser-dynamic';
import { AppModule } from './app/app.module';

// تشغيل الموديول بدلاً من المكون
platformBrowserDynamic().bootstrapModule(AppModule)
  .catch(err => console.error(err));
```

---

## 🔍 لماذا لا يزال هناك أخطاء؟ (The "Unknown Element" Error)

حتى بعد هذه الخطوات، ستظهر أخطاء في الكونسول مثل:

`'app-header' is not a known element`

`'app-user' is not a known element`

**السبب:**

لقد قمنا بتعريف `AppComponent` فقط داخل الموديول.

ولكن `AppComponent` يستخدم في القالب الخاص به: `<app-header>`, `<app-user>`, `<app-tasks>`.

الموديول لا يعرف ما هي هذه العناصر لأننا لم نقم بتحويلها وتسجيلها في `declarations` بعد. الموديول "أعمى" لا يرى إلا ما تسجله بداخله.

**الخطوة التالية:** سنقوم بحل هذه الأخطاء عن طريق تحويل باقي المكونات (`Header`, `User`, `Tasks`) إلى نظام Modules وتسجيلها جميعاً في `AppModule`.

>[!tip]
>END



---


# Converting Components: From Standalone to Module-Based

## 🎯 الغرض من هذا الدرس (The Goal)

حالياً، `AppModule` يستورد المكونات (`Header`, `User`, `Tasks`) كما لو كانت موديولات خارجية لأنها لا تزال `standalone: true`.

الهدف هو:

1. تحويل `HeaderComponent` إلى مكون كلاسيكي (Non-standalone).
    
2. نقله من مصفوفة `imports` إلى مصفوفة `declarations` في الموديول.
    
3. فهم الفرق بين المصفوفتين بدقة.
    

---

## 💡 الشرح المفصل (The Logic)

### 1. مصفوفة `imports` مقابل `declarations`

هذه هي القاعدة الذهبية في عالم NgModules:

- **Declarations (التصريحات):** مخصصة للمكونات (Components)، التوجيهات (Directives)، والأنابيب (Pipes) التي **تنتمي** لهذا الموديول وليست Standalone. (أنت تملكها).
    
- **Imports (الواردات):** مخصصة للموديولات الأخرى (مثل `BrowserModule`, `FormsModule`) أو للمكونات التي هي **Standalone**. (أنت تستخدمها فقط).
    

### 2. لماذا نحولها؟

لتحقيق هيكلية "Module-based" نقية (Pure)، يجب أن تكون المكونات جزءاً من الموديول (Declared) وليست مستوردة (Imported). هذا يسمح للموديول بالتحكم الكامل في الاعتمادات المشتركة.

---

## 💻 التطبيق العملي (Step-by-Step)

سنبدأ بالمكون الأبسط: `HeaderComponent`.

### 1. تعديل المكون (Header Component)

نذهب للملف ونلغي خاصية الاستقلالية.

**src/app/header/header.component.ts:**



```TypeScript
import { Component } from '@angular/core';

@Component({
  selector: 'app-header',
  templateUrl: './header.component.html',
  styleUrl: './header.component.css',
  standalone: false // ⚠️ ضروري لـ Angular 19+ (أو احذف السطر في النسخ القديمة)
})
export class HeaderComponent {}
```

### 2. تحديث الموديول (App Module)

بمجرد حفظ الخطوة السابقة، سيظهر خطأ في `AppModule` لأنك تحاول استيراد مكون "غير مستقل" داخل `imports`. يجب نقله فوراً.

**src/app/app.module.ts:**



```TypeScript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { AppComponent } from './app.component';
import { HeaderComponent } from './header/header.component'; // استيراد الكلاس
import { UserComponent } from './user/user.component';
import { TasksComponent } from './tasks/tasks.component';

@NgModule({
  declarations: [
    AppComponent,
    HeaderComponent // 1. تم نقله هنا (لأنه أصبح جزءاً من العائلة)
  ],
  imports: [
    BrowserModule,
    UserComponent, // لا يزال Standalone (مؤقتاً)
    TasksComponent // لا يزال Standalone (مؤقتاً)
  ],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

---

## ⚠️ ماذا حدث؟ (Analysis)

1. **HeaderComponent:** أصبح الآن "مواطناً" داخل `AppModule`.
    
2. **User & Tasks:** لا يزالان "ضيوفاً" (Standalone) يتم استيرادهم.
    
3. **التطبيق:** لا يزال يعمل بنجاح لأن Angular يدعم هذا الخليط (Interoperability).
    


>[!tip]
>END


---



# Migrating User & Card Components: Handling Nested Dependencies

## 🎯 الغرض من هذا الدرس (The Goal)

نريد تحويل `UserComponent` إلى نظام Modules.

- **المشكلة:** `UserComponent` يستخدم `<app-card>` في قالبه.
    
- **التحدي:** بمجرد أن نزيل `imports` من `UserComponent`، سيفقد قدرته على التعرف على `CardComponent`.
    
- **الحل:** يجب أن يتولى `AppModule` مسؤولية تعريف الاثنين معاً لكي يرى أحدهما الآخر.
    

---

## 💡 الشرح المفصل (The Logic)

### 1. فقدان الذاكرة (Amnesia)

عندما تحول مكوناً من Standalone إلى Module-based:

1. تقوم بحذف مصفوفة `imports: [CardComponent]`.
    
2. المكون "ينسى" ما هي `CardComponent`.
    
3. يجب أن يقوم الموديول (`AppModule`) بتعريف `CardComponent` في مصفوفة `declarations` لكي يصبح متاحاً لجميع المكونات الأخرى داخل نفس الموديول.
    

### 2. قاعدة الأخوة (Sibling Visibility)

في الـ Module الواحد، جميع المكونات المسجلة في `declarations` ترى بعضها البعض تلقائياً. لا داعي لاستيراد شيء. بمجرد أن يكون `User` و `Card` في نفس الموديول، سيعملان معاً.

---

## 💻 التطبيق العملي (Step-by-Step)

### 1. تحويل UserComponent

نذهب للملف ونلغي خاصية الاستقلالية ونحذف الاستيرادات.

**src/app/user/user.component.ts:**



```TypeScript
import { Component, Input, Output, EventEmitter } from '@angular/core';
import { type User } from './user.model';
// لاحظ: حذفنا استيراد CardComponent

@Component({
  selector: 'app-user',
  templateUrl: './user.component.html',
  styleUrl: './user.component.css',
  standalone: false // ⚠️ ضروري لـ Angular 19+
  // imports: [CardComponent] <-- تم حذفها
})
export class UserComponent {
  // ... Logic
}
```

### 2. تحويل CardComponent (الاعتمادية)

بما أننا سنستخدمه في الموديول، يجب تحويله هو أيضاً.

**src/app/shared/card/card.component.ts:**



```TypeScript
import { Component } from '@angular/core';

@Component({
  selector: 'app-card',
  templateUrl: './card.component.html',
  styleUrl: './card.component.css',
  standalone: false // ⚠️ ضروري لـ Angular 19+
})
export class CardComponent {}
```

### 3. تحديث الموديول (App Module)

الآن نسجل الاثنين في `declarations`.

**src/app/app.module.ts:**



```TypeScript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { AppComponent } from './app.component';
import { HeaderComponent } from './header/header.component';
import { UserComponent } from './user/user.component'; // Import
import { CardComponent } from './shared/card/card.component'; // Import
import { TasksComponent } from './tasks/tasks.component';

@NgModule({
  declarations: [
    AppComponent,
    HeaderComponent,
    UserComponent, // 1. تم النقل هنا
    CardComponent  // 2. تم النقل هنا (ليكون متاحاً للـ User)
  ],
  imports: [
    BrowserModule,
    TasksComponent // ما زال Standalone (مؤقتاً)
  ],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

---

## 🔍 تحليل ما حدث

1. **UserComponent:** لم يعد يعرف `CardComponent` بشكل مباشر.
    
2. **AppModule:** جمع الاثنين في غرفة واحدة (`declarations`).
    
3. **النتيجة:** عندما يطلب `UserComponent` العنصر `<app-card>`، يبحث Angular في الموديول، يجده مسجلاً، فيقوم بربطهما بنجاح.
    

**الخطوة التالية:** بقي لدينا المكون الأكبر والأكثر تعقيداً: `TasksComponent`. هذا المكون يحتوي على `TaskComponent` (المفرد)، `NewTaskComponent`، ويستخدم `FormsModule` و `DatePipe`. تحويله سيتطلب تركيزاً عالياً لأننا سنتعامل مع استيراد موديولات أخرى (`FormsModule`) داخل موديولنا. هل أنت مستعد للخطوة النهائية؟

>[!tip]
>END



---





# Finalizing Migration: Pipes, Forms & The Tasks Feature

## 🎯 الغرض من هذا الدرس (The Goal)

نريد تحويل `TasksComponent` وأبنائه (`Task`, `NewTask`).

- **التحدي 1:** `TaskComponent` يستخدم `DatePipe` (أداة تنسيق).
    
- **التحدي 2:** `NewTaskComponent` يستخدم `FormsModule` (موديول كامل).
    
- **الهدف:** دمج كل هذه القطع في `AppModule` ليعمل التطبيق كوحدة واحدة متناغمة.
    

---

## 💡 الشرح المفصل (The Logic)

### 1. أين ذهب الـ `DatePipe`؟

في Standalone، كنا نستورد `DatePipe` يدوياً. في نظام Modules، القصة مختلفة:

- الـ `BrowserModule` الذي استوردناه في البداية يحتوي داخله على موديول يسمى `CommonModule`.
    
- **CommonModule:** هو "حقيبة أدوات" Angular الأساسية (تحتوي على `ngIf`, `ngFor`, `DatePipe`, `CurrencyPipe`, etc...).
    
- **النتيجة:** بمجرد وجود `BrowserModule` في `AppModule`، تصبح كل الـ Pipes متاحة تلقائياً لجميع المكونات دون الحاجة لاستيرادها.
    

### 2. التعامل مع `FormsModule`

بدلاً من استيراد `FormsModule` في كل مكون يحتاجه (مثل `NewTaskComponent`)، نقوم باستيراده **مرة واحدة** في `AppModule`.

- هذا يجعله متاحاً "Global" لجميع المكونات المسجلة في `declarations`.
    

---

## 💻 التطبيق العملي (Step-by-Step)

### 1. تحويل المكون الفرعي (Task Component)

هذا المكون يستخدم `Card` و `DatePipe`.

**src/app/tasks/task/task.component.ts:**



```TypeScript
import { Component, Input, inject } from '@angular/core';
// لاحظ: حذفنا استيراد DatePipe و CardComponent

@Component({
  selector: 'app-task',
  templateUrl: './task.component.html',
  styleUrl: './task.component.css',
  standalone: false // ⚠️ ضروري لـ Angular 19+
  // imports: [DatePipe, CardComponent] <-- تم حذفها
})
export class TaskComponent {
  // ... Logic
}
```

### 2. تحويل مكون الإضافة (New Task Component)

هذا المكون يستخدم `FormsModule`.

**src/app/tasks/new-task/new-task.component.ts:**



```TypeScript
import { Component, Output, EventEmitter, inject, Input } from '@angular/core';
import { TasksService } from '../tasks.service';
// لاحظ: حذفنا استيراد FormsModule من هنا

@Component({
  selector: 'app-new-task',
  templateUrl: './new-task.component.html',
  styleUrl: './new-task.component.css',
  standalone: false // ⚠️ ضروري لـ Angular 19+
  // imports: [FormsModule] <-- تم حذفها
})
export class NewTaskComponent {
  // ... Logic
}
```

### 3. تحويل المكون الأب (Tasks Component)

هذا المكون يجمعهم.

**src/app/tasks/tasks.component.ts:**



```TypeScript
import { Component, Input } from '@angular/core';
// لاحظ: حذفنا استيراد المكونات الفرعية

@Component({
  selector: 'app-tasks',
  templateUrl: './tasks.component.html',
  styleUrl: './tasks.component.css',
  standalone: false // ⚠️ ضروري لـ Angular 19+
  // imports: [TaskComponent, NewTaskComponent] <-- تم حذفها
})
export class TasksComponent {
  // ... Logic
}
```

### 4. التجميع النهائي (App Module Update)

الآن نجمع كل القطع في "لوحة التحكم".

**src/app/app.module.ts:**



```TypeScript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { FormsModule } from '@angular/forms'; // 1. استيراد موديول النماذج

import { AppComponent } from './app.component';
import { HeaderComponent } from './header/header.component';
import { UserComponent } from './user/user.component';
import { CardComponent } from './shared/card/card.component';
import { TasksComponent } from './tasks/tasks.component';
import { TaskComponent } from './tasks/task/task.component';
import { NewTaskComponent } from './tasks/new-task/new-task.component';

@NgModule({
  declarations: [
    AppComponent,
    HeaderComponent,
    UserComponent,
    CardComponent,
    TasksComponent,    // 2. تسجيل المكون الأب
    TaskComponent,     // 3. تسجيل المكون الابن
    NewTaskComponent   // 4. تسجيل مكون الإضافة
  ],
  imports: [
    BrowserModule,
    FormsModule // 5. تفعيل النماذج لكل المكونات أعلاه
  ],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

---

## 🔍 الخلاصة (Recap)

بهذا نكون قد أتممنا الهجرة بنجاح! 🚀

التطبيق الآن يعمل بنسبة 100% باستخدام **NgModules** التقليدية.

- **Declarations:** تحتوي على كل مكوناتنا (`App`, `Header`, `User`, `Card`, `Tasks`, `Task`, `NewTask`).
    
- **Imports:** تحتوي على الموديولات الخارجية (`BrowserModule`, `FormsModule`).
    
- **Standalone:** تم إزالته (أو تعطيله بـ `false`) من جميع الملفات.
    

>[!tip]
>END
>

---




# Creating a Shared Module: The `exports` Array

## 🎯 الغرض من هذا الدرس (The Goal)

حالياً، `AppModule` مزدحم جداً. في التطبيقات الكبيرة، لا نضع كل شيء في `AppModule`.

الهدف هو:

1. إنشاء موديول خاص (`SharedModule`) يجمع المكونات التي تُستخدم في أماكن متعددة (مثل `CardComponent`).
    
2. فهم مفهوم **الخصوصية (Encapsulation)** في الموديولات: المكونات داخل الموديول "خاصة" افتراضياً، ويجب "تصديرها" لتصبح مرئية للآخرين.
    

---

## 💡 الشرح المفصل (The Logic)

### 1. الموديول "حصن مغلق" (Fortress)

تخيل الموديول كأنه منزل.

- **Declarations:** هم سكان المنزل (المكونات). يعرفون بعضهم البعض جيداً.
    
- **Imports:** هم الضيوف الذين يدخلون المنزل.
    
- **Exports (الجديد):** هي "الشرفة" أو "النافذة". سكان المنزل لا يمكن رؤيتهم من الخارج إلا إذا وقفوا في الشرفة (تم وضعهم في مصفوفة Exports).
    

### 2. لماذا SharedModule؟

المكون `CardComponent` يُستخدم من قبل `UserComponent` ويستخدم أيضاً من قبل `TasksComponent`. بدلاً من تركه عائماً في `AppModule`، نضعه في موديول مشترك يمكن استيراده في أي مكان نحتاج فيه للبطاقات.

---

## 💻 التطبيق العملي (Step-by-Step)

### 1. إنشاء الموديول المشترك

أنشئ ملفاً جديداً في مجلد `shared`.

**Masar:** `src/app/shared/shared.module.ts`



```TypeScript
import { NgModule } from '@angular/core';
import { CardComponent } from './card/card.component'; // استيراد المكون

@NgModule({
  // 1. التعريف: نسجل المكون ليعرف الموديول بوجوده
  declarations: [CardComponent],
  
  // 2. التصدير: هذه الخطوة الحاسمة!
  // بدونها، لن يستطيع أي موديول آخر استخدام <app-card>
  exports: [CardComponent]
})
export class SharedModule {}
```

### 2. تنظيف الموديول الرئيسي (AppModule)

الآن يجب أن نزيل `CardComponent` من `AppModule` (لأنه لا يجوز تسجيل المكون في موديولين)، ونستورد `SharedModule` بدلاً منه.

**src/app/app.module.ts:**



```TypeScript
import { NgModule } from '@angular/core';
// ... imports
import { SharedModule } from './shared/shared.module'; // 1. استيراد الموديول الجديد

@NgModule({
  declarations: [
    AppComponent,
    HeaderComponent,
    UserComponent,
    // CardComponent, <-- تم حذفه من هنا (نقلناه)
    TasksComponent,
    TaskComponent,
    NewTaskComponent
  ],
  imports: [
    BrowserModule,
    FormsModule,
    SharedModule // 2. إضافة الموديول المشترك
  ],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

---

## ⚠️ ملاحظات هامة (Analysis)

1. **خطأ شائع:** إذا وضعت `CardComponent` في `declarations` داخل `SharedModule` ونسيت وضعه في `exports`، سيظهر لك خطأ في `AppModule` يقول: `'app-card' is not a known element`. السبب هو أن المكون موجود داخل الموديول المشترك لكنه "محبوس" بالداخل.
    
2. **التكرار ممنوع:** لا يمكنك ترك `CardComponent` في `declarations` الخاصة بـ `AppModule` وأيضاً في `SharedModule`. المكون يجب أن ينتمي لمنزل واحد فقط.
    

**الخطوة التالية:** التطبيق الآن أنظف قليلاً. لكن `TasksComponent` و `UserComponent` لا يزالون يزحمون `AppModule`. في الدرس القادم، سنقوم بإنشاء موديول خاص للمهام (`TasksModule`) لنقل كل ما يتعلق بالمهام داخله.

>[!tip]
>END

---


# Creating a Feature Module: `TasksModule` & `CommonModule`

## 🎯 الغرض من هذا الدرس (The Goal)

نريد تنظيف `AppModule` تماماً.

- **الوضع الحالي:** `AppModule` يحتوي على تفاصيل كثيرة عن المهام (`Tasks`, `Task`, `NewTask`).
    
- **الهدف:** نقل كل هذه التفاصيل إلى موديول خاص (`TasksModule`)، وجعل `AppModule` يستورد هذا الموديول فقط كسطر واحد.
    

---

## 💡 الشرح المفصل (The Logic)

### 1. قاعدة الانعزال (Module Isolation Rule)

هذه أهم نقطة في الدرس: **الموديولات لا ترث الواردات (Imports) من الأب!**

- إذا كان `AppModule` يستورد `SharedModule` (للبطاقات)، هذا لا يعني أن `TasksModule` (الابن) يرى البطاقات تلقائياً.
    
- **كل موديول يجب أن يخدم نفسه بنفسه.** إذا احتاج `TasksModule` للبطاقات، يجب أن يستورد `SharedModule` بنفسه.
    

### 2. معضلة `BrowserModule` vs `CommonModule`

- **`BrowserModule`:** يحتوي على أدوات التشغيل الأساسية للمتصفح + أدوات مشتركة (`ngIf`, `ngFor`, `DatePipe`). **يُستورد مرة واحدة فقط** في الموديول الجذري (`AppModule`).
    
- **`CommonModule`:** يحتوي على الأدوات المشتركة فقط (`ngIf`, `DatePipe`...). **يُستورد في كل الموديولات الفرعية** (Feature Modules).
    
- **القاعدة:**
    
    - Root Module -> `BrowserModule`
        
    - Any Other Module -> `CommonModule`
        

---

## 💻 التطبيق العملي (Step-by-Step)

### 1. إنشاء موديول المهام

أنشئ ملفاً جديداً: `src/app/tasks/tasks.module.ts`.



```TypeScript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common'; // 1. بديل BrowserModule
import { FormsModule } from '@angular/forms'; // 2. نحتاجه للنموذج
import { SharedModule } from '../shared/shared.module'; // 3. نحتاجه للبطاقات

import { TasksComponent } from './tasks.component';
import { TaskComponent } from './task/task.component';
import { NewTaskComponent } from './new-task/new-task.component';

@NgModule({
  declarations: [
    TasksComponent,
    TaskComponent,
    NewTaskComponent
  ],
  exports: [
    TasksComponent // 4. نصدّر فقط ما سيستخدمه الأب (AppModule)
    // لا حاجة لتصدير Task أو NewTask لأنهم "شأن داخلي" لهذا الموديول
  ],
  imports: [
    CommonModule, // هام جداً: للوصول لـ DatePipe, ngIf, ngFor
    FormsModule,  // للوصول لـ ngModel, ngSubmit
    SharedModule  // للوصول لـ app-card
  ]
})
export class TasksModule {}
```

### 2. تنظيف الموديول الرئيسي (AppModule)

الآن `AppModule` سيصبح رشيقاً جداً. سنحذف كل مكونات المهام ونستورد الموديول بدلاً منها.

**src/app/app.module.ts:**



```TypeScript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

import { AppComponent } from './app.component';
import { HeaderComponent } from './header/header.component';
import { UserComponent } from './user/user.component';
import { SharedModule } from './shared/shared.module';
import { TasksModule } from './tasks/tasks.module'; // 1. استيراد الموديول الجديد

@NgModule({
  declarations: [
    AppComponent,
    HeaderComponent,
    UserComponent,
    // تم حذف TasksComponent و TaskComponent و NewTaskComponent
  ],
  imports: [
    BrowserModule,
    // تم حذف FormsModule (لأننا لم نعد نحتاجه هنا، انتقل للابن)
    SharedModule,
    TasksModule // 2. تسجيل الموديول
  ],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

---

## 🔍 الخلاصة المعمارية (Architectural Review)

أصبح تطبيقنا الآن منظماً بشكل احترافي جداً:

1. **`AppModule` (Root):** المدير العام. يعرف فقط المكونات الرئيسية (`Header`, `User`) والموديولات الفرعية (`Tasks`).
    
2. **`TasksModule` (Feature):** مدير قسم المهام. يدير كل تفاصيل المهام ويستورد أدواته الخاصة (`CommonModule`, `FormsModule`).
    
3. **`SharedModule` (Shared):** قسم الخدمات المشتركة. يوفر عناصر واجهة (`Card`) للجميع.
    

**النتيجة:**

- **قابلة للصيانة (Maintainable):** كل جزء من التطبيق معزول في مكانه.
    
- **قابلة لإعادة الاستخدام (Reusable):** يمكن نقل `TasksModule` لتطبيق آخر بسهولة.
    
- **واضحة (Clean):** ملف `app.module.ts` صغير وسهل القراءة.
    



>[!tip]
>END

---

