# 🚗 DVLD - Driver & Vehicle License Department
## 📚 Project Architecture & Technical Reference Manual (المرجع الشامل للنظام)

> **Repository Remote**: `https://github.com/mohamed68909/DriverLicenseManagement.git`  
> **Target Framework**: .NET Framework 4.8 / Windows Forms  
> **Architecture Style**: 3-Tier Architecture (Presentation Layer, Business Logic Layer, Data Access Layer)  
> **Database Engine**: Microsoft SQL Server (ADO.NET)  
> **Solution File**: `DVLD/DVLD.sln`

---

## 📌 Maintenance Protocol (بروتوكول التحديث الإلزامي)

> [!IMPORTANT]
> **قاعدة عمل دائمة لجميع المطورين والذكاء الاصطناعي (Antigravity / AI Agents):**
> 1. هذا الملف هو **المصدر الوحيد والمباشر للحقيقة (Single Source of Truth)** لفهم هيكل المشروع، جداول البيانات، وقواعد العمل دون الحاجة لإعادة قراءة ملفات الكود من الصفر في كل مهمة.
> 2. **مع كل تعديل مستقبلي في الكود** (إضافة شاشة، تعديل إجراء مخزن أو استعلام، إضافة فئة Business أو Data Access، تعديل Enums أو قواعد التحقق):
>    - **يجب فوراً تحديث هذا الملف (`PROJECT_REFERENCE.md`)** ليعكس التغييرات الجديدة بدقة.
>    - يمنع إنهاء أي مهمة تطويرية دون مزامنة هذا الملف.

---

## 1. 🏗️ Solution Architecture (هيكلية الحل والطبقات)

المشروع مبني بأسلوب **3-Tier Architecture** مفصول في مشاريع مستقلة ضمن Solution واحد:

```
DVLD (Root)
│
├── 📁 DVLD/                     --> [Presentation Layer] تطبيق واجهات ويندوز (Windows Forms)
├── 📁 DVLD_Buisness/            --> [Business Logic Layer - BLL] منطق الأعمال والتحقق والقواعد
├── 📁 DVLD_DataAccess/          --> [Data Access Layer - DAL] الوصول لقاعدة البيانات عبر ADO.NET
└── 📁 bin/                      --> المخرجات المجمعة والملفات التنفيذية
```

### العلاقات بين الطبقات:
- **`DVLD` (UI)** يشير إلى `DVLD_Buisness` فقط (ولا يتعامل مع `DVLD_DataAccess` مباشرة).
- **`DVLD_Buisness` (BLL)** يشير إلى `DVLD_DataAccess`، ويمثل وسيط الأعمال والتحقق.
- **`DVLD_DataAccess` (DAL)** يتعامل مباشرة مع SQL Server باستخدام `System.Data.SqlClient`.

---

## 2. 🗄️ Database Architecture & Data Dictionary (قاموس وقواعد البيانات)

### 2.1 إعدادات الاتصال (Connection String)
- **الملف**: `DVLD_DataAccess/clsConnectionSetting.cs`
- **القيمة الافتراضية**:
  ```csharp
  public static string ConnectionString = "Server=.;Database=DVLD;User Id=sa;Password=sa123456;";
  ```
  *(يمكن تعديلها لاستخدام Windows Authentication عبر `Server=.;Database=DVLD;Integrated Security=True;` حسب بيئة التشغيل).*

### 2.2 دليل الجداول الرئيسية والحقول (Database Tables)

| اسم الجدول | المفتاح الأساسي (PK) | العلاقات الخارجية (FK) | الوصف والأعمدة الهامة |
| :--- | :--- | :--- | :--- |
| **`People`** | `PersonID` (Identity) | `NationalityCountryID` -> `Countries` | `NationalNo` (فريد), `FirstName`, `SecondName`, `ThirdName`, `LastName`, `DateOfBirth`, `Gendor` (0: Male, 1: Female), `Address`, `Phone`, `Email`, `ImagePath` |
| **`Users`** | `UserID` (Identity) | `PersonID` -> `People` (1-to-1) | `UserName` (فريد), `Password`, `IsActive` (bit) |
| **`Countries`** | `CountryID` (Identity) | لا يوجد | `CountryName` (قائمة الدول للجنسيات) |
| **`ApplicationTypes`** | `ApplicationTypeID` (Identity) | لا يوجد | `ApplicationTypeTitle`, `ApplicationFees` (رسوم كل نوع معاملة) |
| **`Applications`** | `ApplicationID` (Identity) | `ApplicantPersonID` -> `People`<br>`ApplicationTypeID` -> `ApplicationTypes`<br>`CreatedByUserID` -> `Users` | `ApplicationDate`, `ApplicationStatus` (1: New, 2: Cancelled, 3: Completed), `LastStatusDate`, `PaidFees` |
| **`LocalDrivingLicenseApplications`** | `LocalDrivingLicenseApplicationID` | `ApplicationID` -> `Applications`<br>`LicenseClassID` -> `LicenseClasses` | طلبات رخص القيادة المحلية وربطها بفئة الرخصة واختباراتها |
| **`LicenseClasses`** | `LicenseClassID` (Identity) | لا يوجد | `ClassName`, `ClassDescription`, `MinimumAllowedAge`, `DefaultValidityLength`, `ClassFees` |
| **`Drivers`** | `DriverID` (Identity) | `PersonID` -> `People`<br>`CreatedByUserID` -> `Users` | `CreatedDate` (يسجل تلقائياً عند إصدار أول رخصة قيادة للشخص) |
| **`Licenses`** | `LicenseID` (Identity) | `ApplicationID` -> `Applications`<br>`DriverID` -> `Drivers`<br>`LicenseClass` -> `LicenseClasses`<br>`CreatedByUserID` -> `Users` | `IssueDate`, `ExpirationDate`, `Notes`, `PaidFees`, `IsActive`, `IssueReason` (1: First, 2: Renew, 3: Damaged, 4: Lost) |
| **`InternationalLicenses`** | `InternationalLicenseID` | `ApplicationID` -> `Applications`<br>`DriverID` -> `Drivers`<br>`IssuedUsingLocalLicenseID` -> `Licenses` | `IssueDate`, `ExpirationDate`, `IsActive` (مدتها سنة واحدة، تعتمد على رخصة محلية سارية من الفئة 3) |
| **`DetainedLicenses`** | `DetainID` (Identity) | `LicenseID` -> `Licenses`<br>`CreatedByUserID` -> `Users`<br>`ReleasedByUserID` -> `Users`<br>`ReleaseApplicationID` -> `Applications` | `DetainDate`, `FineFees`, `IsReleased`, `ReleaseDate` |
| **`TestTypes`** | `TestTypeID` (Identity) | لا يوجد | `TestTypeTitle`, `TestTypeDescription`, `TestTypeFees` (1: Vision, 2: Written, 3: Street) |
| **`TestAppointments`** | `TestAppointmentID` | `TestTypeID` -> `TestTypes`<br>`LocalDrivingLicenseApplicationID` -> `LocalDrivingLicenseApplications`<br>`CreatedByUserID` -> `Users`<br>`RetakeTestApplicationID` -> `Applications` | `AppointmentDate`, `PaidFees`, `IsLocked` (يغلق الموعد بعد تقديم الاختبار) |
| **`Tests`** | `TestID` (Identity) | `TestAppointmentID` -> `TestAppointments`<br>`CreatedByUserID` -> `Users` | `TestResult` (bit: 1 Pass, 0 Fail), `Notes` |

### 2.3 الـ Views المستخدمة في الاستعلامات وعرض الجداول:
- `ApplicationsList_View`
- `LocalDrivingLicenseApplications_View`
- `TestAppointments_View`
- `Drivers_View`
- `People_View`
- `InternationalLicenses_View`
- `DetainedLicenses_View`

---

## 3. ⚙️ Business Logic Layer (BLL - طبقة منطق الأعمال)

### 3.1 النمط المتبع في فئات الأعمال (BLL Pattern)
تعتمد جميع كلاسات BLL على نمط موحد:
1. **نمط الوضع (`enMode`)**:
   ```csharp
   public enum enMode { AddNew = 0, Update = 1 };
   public enMode Mode = enMode.AddNew;
   ```
2. **دالة الحفظ الموحدة (`Save()`)**:
   تقوم بفحص `Mode`، فإذا كان `AddNew` تستدعي `_AddNew()` في DAL وتغير النمط إلى `Update` بعد النجاح، وإذا كان `Update` تستدعي `_Update()`.
3. **الدوال الثابتة للبحث (`Find`)**:
   - `clsPerson.Find(int PersonID)` / `clsPerson.Find(string NationalNo)`
   - `clsUser.FindByUserID(int UserID)` / `clsUser.FindByPersonID(int PersonID)` / `clsUser.FindByUsernameAndPassword(...)`
   - `clsLicense.Find(int LicenseID)`
   - إلخ.

### 3.2 الـ Enums الأساسية في المشروع

#### أ. أنواع المعاملات (`clsApplication.enApplicationType`)
| القيمة (ID) | الاسم البرمجي | الوصف |
| :---: | :--- | :--- |
| **1** | `NewDrivingLicense` | إصدار رخصة قيادة جديدة لأول مرة |
| **2** | `RenewDrivingLicense` | تجديد رخصة قيادة منتهية |
| **3** | `ReplaceLostDrivingLicense` | إصدار بدل فاقد |
| **4** | `ReplaceDamagedDrivingLicense` | إصدار بدل تالف |
| **5** | `ReleaseDetainedDrivingLicsense` | فك حجز رخصة محجوزة |
| **6** | `NewInternationalLicense` | إصدار رخصة دولية |
| **7** | `RetakeTest` | إعادة اختبار بعد الرسوب |

#### ب. حالات المعاملة (`clsApplication.enApplicationStatus`)
- `New = 1`: قيد الإجراء / جديدة
- `Cancelled = 2`: ملغاة
- `Completed = 3`: مكتملة ومنتهية

#### ج. أنواع الاختبارات (`clsTestType.enTestType`)
- `VisionTest = 1`: فحص النظر
- `WrittenTest = 2`: الاختبار النظري (الكتابي)
- `StreetTest = 3`: الاختبار العملي (الشارع)

#### د. أسباب إصدار الرخصة (`clsLicense.enIssueReason`)
- `FirstTime = 1`: إصدار لأول مرة
- `Renew = 2`: تجديد
- `DamagedReplacement = 3`: بدل تالف
- `LostReplacement = 4`: بدل فاقد

### 3.3 الوراثة في كلاسات الطلبات (Inheritance Hierarchy)
- **`clsApplication`** (Base Class): يحتوي البيانات الأساسية (`ApplicationID`, `ApplicantPersonID`, `ApplicationDate`, `ApplicationTypeID`, `ApplicationStatus`, `LastStatusDate`, `PaidFees`, `CreatedByUserID`).
  - **`clsLocalDrivingLicenseApplication`** يرث من `clsApplication`: يضيف `LocalDrivingLicenseApplicationID`, `LicenseClassID`, وإدارة الاختبارات وإصدار الرخصة الأولى.
  - **`clsInternationalLicense`** يرث من `clsApplication`: يضيف `InternationalLicenseID`, `IssuedUsingLocalLicenseID`, `DriverID`, وتاريخ الانتهاء السنوي.

---

## 4. 🔄 Core Business Workflows & Rules (مسارات وقواعد العمل)

### 4.1 مسار طلب رخصة جديدة واجتياز الاختبارات (Local Driving License Workflow)
1. إنشاء سجل الطلب (`LocalDrivingLicenseApplication`):
   - التحقق من عدم وجود رخصة نشطة أو طلب مفتوح لنفس الشخص ونفس فئة الرخصة.
   - التحقق من السن القانوني للشخص مقارنة بـ `MinimumAllowedAge` لفئة الرخصة.
2. الاختبارات المتسلسلة (Strict Order):
   - **الخطوة 1: Vision Test**: لا يمكن حجز موعد إلا إذا لم يكن هناك موعد نشط.
   - **الخطوة 2: Written Test**: لا يمكن حجز موعد إلا بعد **اجتياز فحص النظر**.
   - **الخطوة 3: Street Test**: لا يمكن حجز موعد إلا بعد **اجتياز الاختبار النظري**.
3. الرسوب وإعادة الاختبار (`Retake Test`):
   - عند رسوب الطالب في أي اختبار، يتطلب حجز موعد جديد دفع رسوم إعادة الاختبار عبر إنشاء طلب فرعي من نوع `RetakeTest (ID: 7)`.
4. إصدار الرخصة لأول مرة (`IssueLicenseForTheFirstTime`):
   - بعد اجتياز الاختبارات الثلاثة (Passed Count = 3).
   - يتم التحقق من وجود الشخص كـ `Driver`؛ فإذا لم يكن مسجلاً يُنشأ له سجل `Driver` جديد.
   - يتم إنشاء رخصة `clsLicense` جديدة بفترة صلاحية مأخوذة من `DefaultValidityLength` لفئتها، وتصبح `IsActive = true`.
   - يتم تحويل حالة الطلب إلى `Completed (3)`.

### 4.2 مسار تجديد الرخصة (`Renew Driving License`)
- متاح فقط إذا كانت الرخصة منتهية الصلاحية (`IsLicenseExpired()`) ونشطة وغير محجوزة.
- يتم إنشاء طلب تجديد (`RenewDrivingLicense (ID: 2)`).
- يتم إصدار رخصة جديدة بنفس الفئة والسائق، وتعطيل الرخصة القديمة (`DeactivateCurrentLicense()`).

### 4.3 مسار الاستبدال لبدل فاقد / تالف (`Replace Lost/Damaged`)
- الرخصة يجب أن تكون نشطة.
- يتم إنشاء طلب (`ReplaceDamagedDrivingLicense (ID: 4)` أو `ReplaceLostDrivingLicense (ID: 3)`).
- يتم إصدار رخصة جديدة بنفس تاريخ انتهاء الرخصة القديمة (بدون رسوم رخصة إضافية، فقط رسوم الطلب).
- يتم إلغاء تنشيط الرخصة الأصلية.

### 4.4 مسار حجز وفك حجز الرخصة (`Detain & Release License`)
- **الحجز (Detain)**:
  - التحقق من أن الرخصة ليست محجوزة بالفعل.
  - إدخال الغرامة المالية (`FineFees`) ومستخدم النظام القائم بالحجز.
- **فك الحجز (Release)**:
  - إنشاء طلب فك حجز (`ReleaseDetainedDrivingLicsense (ID: 5)`).
  - دفع رسوم فك الحجز + قيمة الغرامة (`FineFees`).
  - تحديث سجل الحجز (`IsReleased = true`, `ReleaseDate`, `ReleasedByUserID`, `ReleaseApplicationID`).

### 4.5 مسار الرخصة الدولية (`International License`)
- يشترط أن يمتلك المتقدم رخصة محلية **سارية ونشطة ومن الفئة رقم 3 (Class 3 - Ordinary driving license)**.
- لا يمكن إصدار رخصة دولية إذا كان لديه بالفعل رخصة دولية نشطة غير منتهية.
- مدة صلاحية الرخصة الدولية **سنة واحدة فقط** من تاريخ الإصدار.

---

## 5. 🖥️ Presentation Layer (UI & User Controls)

### 5.1 تنظيم الواجهات (`DVLD/`):
- **`frmMain.cs`**: الشاشة الرئيسية التي تضم الـ MenuStrip والمربوطة بجميع وظائف النظام:
  - **Applications**: Driving Licenses Services (Local, International, Renew, Replacement, Detain/Release, Retake), Manage Applications, Manage Application Types, Manage Test Types.
  - **People**: شاشة إدارة وإضافة الأشخاص والفلترة والبحث.
  - **Drivers**: شاشة استعراض السائقين وتاريخ رخصهم.
  - **Users**: إدارة مستخدمي النظام وصلاحياتهم وتغيير كلمات السر.
  - **Account Settings**: عرض بيانات المستخدم الحالي وتسجيل الخروج.
- **`frmLogin.cs`**: شاشة الدخول الداعمة لميزة "Remember Me".

### 5.2 عناصر التحكم المخصصة القابلة لإعادة الاستخدام (Reusable User Controls)
| اسم الـ Control | المسار | الوظيفة |
| :--- | :--- | :--- |
| **`ctrlPersonCard`** | `DVLD/People/Controls` | عرض بطاقة بيانات الشخص الكاملة مع صورته وجنسيته |
| **`ctrlPersonCardWithFilter`** | `DVLD/People/Controls` | بطاقة الشخص مع شريط بحث (بالرقم القومي أو PersonID) وزر إضافة شخص جديد |
| **`ctrlUserCard`** | `DVLD/User` | كارت يدمج بطاقة الشخص مع بيانات حساب المستخدم (`UserID`, `UserName`, `IsActive`) |
| **`ctrlDriverLicenseInfo`** | `DVLD/Licenses/Local Licenses/Controls` | عرض بطاقة تفاصيل الرخصة المحلية بالكامل |
| **`ctrlDriverLicenseInfoWithFilter`**| `DVLD/Licenses/Local Licenses/Controls` | عرض بيانات الرخصة مع شريط بحث برقم الرخصة |
| **`ctrlDriverLicenses`** | `DVLD/Licenses/Controls` | عرض تاريخ رخص السائق (رخص محلية ورخص دولية في TabControl) |
| **`ctrlDriverInternationalLicenseInfo`** | `DVLD/Licenses/International Licenses/Controls` | عرض بطاقة تفاصيل الرخصة الدولية |
| **`ctrlApplicationBasicInfo`** | `DVLD/Applications/Controls` | عرض البيانات العامة للطلب الأساسي (`clsApplication`) |
| **`ctrlDrivingLicenseApplicationInfo`**| `DVLD/Applications/Local Driving License` | دمج بيانات الطلب الأساسي مع بيانات رخصة القيادة المحلية والاختبارات المجتازة |
| **`crlScheduleTest` / `ctrlSecheduledTest`**| `DVLD/Tests/Controls` | جدولة وعرض تفاصيل مواعيد الاختبارات الثلاثة مع دعم Retake Test |

---

## 6. 🛠️ Utilities & Global Helpers (الأدوات العامة)

1. **`clsGlobal`** (`DVLD/Global Classes/clsGlobal.cs`):
   - `clsGlobal.CurrentUser`: تخزين كائن المستخدم المسجل دخوله حالياً في الذاكرة.
   - `RememberUsernameAndPassword(username, password)`: حفظ بيانات الدخول في ملف نصي `data.txt` مشفر بالفاصل `#//#`.
   - `GetStoredCredential(ref username, ref password)`: استرجاع بيانات الدخول المحفوظة تلقائياً عند فتح شاشة الدخول.
2. **`clsUtil`** (`DVLD/Global Classes/util.cs`):
   - `CopyImageToProjectImagesFolder(ref sourceFile)`: نسخ الصورة المختارة للشخص إلى مسار النظام المركزي `C:\DVLD-People-Images\` بعد توليد اسم فريد لها عبر `Guid`.
3. **`clsFormat`** (`DVLD/Global Classes/clsFormat.cs`):
   - تنسيق التواريخ إلى صيغة `dd/MMM/yyyy`.
4. **`clsValidatoin`** (`DVLD/Global Classes/clsValidatoin.cs`):
   - التحقق من صحة البريد الإلكتروني، الأرقام الصحيحة والأرقام العشرية باستخدام Regex.

---

## 7. 🚀 Build & Compilation Guide (دليل البناء والتشغيل)

- **MSBuild Path**:
  ```powershell
  & "C:\Program Files\Microsoft Visual Studio\18\Community\MSBuild\Current\Bin\MSBuild.exe" DVLD\DVLD.sln /p:Configuration=Debug
  ```
- **النتيجة الحالية**:
  `Build succeeded. 0 Error(s).`
- **تشغيل البرنامج**:
  الملف التنفيذي الناتج: `DVLD/bin/Debug/DVLD.exe`.
- **مجلد الصور**:
  يجب التأكد من وجود المجلد `C:\DVLD-People-Images\` (يتم إنشاؤه برمجياً إن لم يكن موجوداً).

---

## 8. 📝 Changelog & Modification Log (سجل التعديلات المحدث)

| التاريخ | المطور / الأداة | ملخص التعديلات المنفذة | الملفات المتأثرة |
| :---: | :---: | :--- | :--- |
| **2026-09-20** | Antigravity AI | - استعادة ملفات مجلد `Applications` بالكامل.<br>- التحقق من اكتمال البناء مع حل مشاكل المسارات وتأكيد نجاح البناء بـ 0 أخطاء.<br>- إنشاء المرجع الهندسي الشامل للنظام وتوثيق كافة الطبقات وقواعد البيانات. | `DVLD/Applications/**`, `PROJECT_REFERENCE.md`, `GEMINI.md` |
| **2026-09-25** | Copilot | - تحسين التصميم المرئي لشاشة تسجيل الدخول: ألوان النظام الموحدة، الخطوط، المسافات، زر الدخول، وتخطيط النموذج.<br>- الحفاظ على منطق تسجيل الدخول كما هو دون تغيير. | `DVLD/Login/frmLogin.Designer.cs`, `PROJECT_REFERENCE.md` |

> ⚠️ **ملاحظة للمستقبل**: أي تعديل يطرأ على كلاسات BLL، أو دوال DAL، أو جداول قاعدة البيانات، أو إضافة شاشات جديدة، يجب تسجيله في هذا الجدول وتحديث البنود المقابلة له أعلاه فوراً.
