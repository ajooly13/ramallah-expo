# Ramallah Expo — بوابة الشركات (تعليمات لـ Claude)

هذا الملف موجّه لأي جلسة Claude Code تكمل العمل على هذا المشروع من أي جهاز.
صاحب المشروع: علي (Ali) — مستخدم غير تقني، احكي معه بالعربي العامي (لهجة فلسطينية)، خطوة بخطوة، بدون مصطلحات.

## المشروع
- تطبيق محاسبة لشركتين: **Ramallah Expo** (قبة معارض حقيقية برام الله) + **TheMob** (تأجير معدات إيفنتات).
- ملف واحد `index.html` (HTML+CSS+JS، عربي RTL، بدون أدوات بناء). الرابط المباشر:
  **https://ajooly13.github.io/ramallah-expo/** (GitHub Pages من فرع main).
- التصميم: هوية Ramallah Expo كحلية، واجهة داخلية (hero) فيها قبة SVG مضيئة أصفر + أكشاك RA1–RA11 + كشّافات + نجوم، أيقونات تنقّل، مشهد سينمائي عند الضغط على القبة (حجوزات 14 يوم)، سكرول لتفاصيل المناسبات.

## الحسابات والأدوار (Firebase Auth بالإيميل)
- مشروع Firebase: `ramallah--mob` (شرطتين!). الإعدادات (config) موجودة داخل index.html.
- **علي: ali8ittqanlaw@gmail.com** — أدمن (نافذة إنشاء الحسابات) + بياناته خاصة فيه (`users/{uid}`) — ما بيشوف بيانات جواد (خصوصية).
- **جواد: jmaher2023@icloud.com** — أدمن + صاحب دفتر اكسبو المُراقَب: بياناته بمستند مشترك `workspaces/main` (هو owner).
- **البلدية: ramallahmunicipalit@gmail.com** — اطلاع فقط على دفتر جواد: كل تبويبات اكسبو **ما عدا الأرباح/الشركاء**، بدون أي تعديل، بدون TheMob وبدون hero. (تحقق من إملاء الإيميل مع علي — ناقصه y).
- إنشاء الحسابات ممنوع من شاشة الدخول — فقط من قسم "👤 إدارة الحسابات" بتبويب النسخ (يظهر للأدمن، ينشئ عبر Firebase app ثانوي).
- "نسيت كلمة السر" موجودة (sendPasswordResetEmail).

## ⏳ خطوة معلّقة إلزامية: نشر قواعد Firestore
بدون هالخطوة حساب البلدية بيطلعله "الصلاحيات لسا ما تفعّلت". علي بيفتح **shell.cloud.google.com** (بحساب Google تبعه، مشروع ramallah--mob) ويلصق:

```bash
PROJECT=$(gcloud config get-value project); TOKEN=$(gcloud auth print-access-token)
RULES='rules_version = "2";
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{uid} {
      allow read, write: if request.auth != null && request.auth.uid == uid;
    }
    match /workspaces/{wid} {
      allow get: if request.auth != null && (resource == null || resource.data.owner == request.auth.uid || request.auth.token.email in resource.data.get("viewers", []));
      allow list: if false;
      allow create: if request.auth != null && request.resource.data.owner == request.auth.uid && request.auth.token.email == "jmaher2023@icloud.com";
      allow update, delete: if request.auth != null && resource.data.owner == request.auth.uid;
    }
  }
}'
RSNAME=$(jq -n --arg c "$RULES" '{source:{files:[{name:"firestore.rules",content:$c}]}}' | curl -s -X POST "https://firebaserules.googleapis.com/v1/projects/$PROJECT/rulesets" -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" -d @- | jq -r '.name'); echo "Ruleset: $RSNAME"
curl -s -X PATCH "https://firebaserules.googleapis.com/v1/projects/$PROJECT/releases/cloud.firestore" -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" -d "{\"release\":{\"name\":\"projects/$PROJECT/releases/cloud.firestore\",\"rulesetName\":\"$RSNAME\"}}" | jq -r '.rulesetName // .error.message'
echo "=== DONE ==="
```

بعدها بالترتيب: (1) حساب جواد يفتح التطبيق أول مرة (بينشئ workspaces/main ببياناته) — حسابه ينُنشأ من نافذة إدارة الحسابات عند علي. (2) حساب البلدية يفتح بعده.

## طريقة النشر (مهم!)
جهاز علي ما عليه Node/Python/gh. النشر عبر **محرر GitHub بالمتصفح** (امتداد Claude in Chrome):
1. عدّل النسخة المحلية `C:\Users\areej\Desktop\ramallah-expo.html` (على جهاز علي الأساسي) أو نزّل index.html من الريبو.
2. PowerShell: اقرأ الملف بـ `[IO.File]::ReadAllText(path,[Text.Encoding]::UTF8)` ثم `Set-Clipboard` (لا تستخدم Get-Content -Raw — بيخرب العربي!).
3. افتح `github.com/ajooly13/ramallah-expo/edit/main/index.html` → انقر على سطر كود → Ctrl+A → Ctrl+V → تأكد إن زر Commit صار **أخضر** وعدد الأسطر منطقي (اللصقة أول مرة كثير مرات ما بتمسك — أعد النقر واللصق) → Commit.
4. تحقق: نزّل `https://raw.githubusercontent.com/ajooly13/ramallah-expo/main/index.html` بـ Invoke-WebRequest وقارن الطول/التطابق مع المحلي (WebFetch بيقص الملفات الكبيرة).

## قواعد مالية مهمة
- اكسبو: بلدية رام الله بتاخد 10% من إجمالي الدخل قبل المصاريف؛ الصافي يتوزع على الشركاء (محمود 5 / جواد 62 / عقبة 33 افتراضياً، قابلة للتعديل والإضافة/الحذف).
- إيجار الأكشاك يدوي كل شهر (زر "سجّل إيجار هذا الشهر")، مش تلقائي.
- القبة (اسمها بالتطبيق "Ramallah Expo"): إيجار من تاريخ–إلى تاريخ + فك وتركيب اختياري + أجندة تقويم.
- TheMob: جواد 45 / عقبة 45 / TheMob 10؛ شيكات بمواعيد استحقاق؛ بوكيت موني منفصل.
- عملتين منفصلتين ₪ و $ بكل شي.
