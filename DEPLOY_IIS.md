# نشر التطبيق على IIS | Deploy to IIS

## المتطلبات | Requirements

- **Windows Server** أو **Windows** مع IIS
- **.NET 10.0 Runtime** (أو SDK) على السيرفر  
  تحميل: https://dotnet.microsoft.com/download
- **ASP.NET Core Hosting Bundle** (يضم Runtime + Module الخاص بـ IIS)  
  تحميل: https://dotnet.microsoft.com/download/dotnet/10.0

---

## 1. نشر التطبيق | Publish the App

من مجلد المشروع في الطرفية:

```powershell
dotnet publish -c Release
```

المخرجات ستكون في: `bin\Release\net10.0\publish\`

أو باستخدام ملف النشر:

```powershell
dotnet publish /p:PublishProfile=FolderProfile
```

---

## 2. إعداد IIS | Configure IIS

### تثبيت ميزات IIS (إن لم تكن مثبتة)

1. **Server Manager** → Add Roles and Features
2. اختر **Web Server (IIS)** ثم:
   - Application Development → **ASP.NET 4.8** (إن احتجت تطبيقات قديمة)
   - Application Development → **.NET Extensibility 4.8**
   - أو من PowerShell (كمسؤول):

```powershell
Install-WindowsFeature -Name Web-Server -IncludeManagementTools
Install-WindowsFeature -Name Web-Asp-Net45
```

3. ثبّت **ASP.NET Core Hosting Bundle** من الرابط أعلاه (هذا يضيف دعم تطبيقات ASP.NET Core لـ IIS).

### إنشاء الموقع أو التطبيق في IIS

1. افتح **IIS Manager** (inetmgr).
2. انقر يمين على **Sites** → **Add Website** (أو Add Application تحت موقع موجود).
3. **Site name:** مثلاً `IisTestApp`
4. **Physical path:** المسار إلى مجلد النشر، مثلاً:
   - `C:\sdv\iistest\bin\Release\net10.0\publish`
5. **Binding:** اختر البورت (مثلاً 80 أو 8080) أو ربط على اسم نطاق.
6. **Application Pool:**
   - انقر **Select** بجانب Application pool واختر أو أنشئ Pool جديد.
   - انقر يمين على الـ Pool → **Basic Settings**:
     - **.NET CLR version:** **No Managed Code**
     - **Managed pipeline mode:** Integrated
   - تأكد أن الهوية (Identity) لديها صلاحية قراءة مجلد النشر.

7. اضغط **OK** ثم شغّل الموقع (Start).

---

## 3. التحقق | Verify

افتح المتصفح على:

- `http://localhost` (أو البورت الذي اخترته)
- أو `http://اسم-السيرفر`

يجب أن تظهر الصفحة الرئيسية لتطبيق ASP.NET MVC.

---

## استكشاف الأخطاء | Troubleshooting

| المشكلة | الحل |
|--------|------|
| 500.19 أو خطأ في web.config | تأكد من تثبيت **ASP.NET Core Hosting Bundle** وإعادة تشغيل IIS. |
| 500.30 In-Process Start Failure | تأكد أن الـ Application Pool = **No Managed Code** وأن مجلد النشر صحيح. |
| 404 | تأكد أن **Physical path** يشير إلى مجلد **publish** وليس مجلد المشروع. |
| صلاحيات | تأكد أن هوية الـ Application Pool (مثلاً ApplicationPoolIdentity) لها صلاحية قراءة على مجلد النشر. |

---

## ملخص الأوامر | Quick Commands

```powershell
# نشر
dotnet publish -c Release

# مسار المخرجات (انسخه إلى Physical path في IIS)
# bin\Release\net10.0\publish
```

بعد النسخ إلى السيرفر، استخدم نفس المسار كـ **Physical path** للموقع أو التطبيق في IIS.
