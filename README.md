# Sehaty Automation (n8n + GitHub)

هذا المستودع يوفّر آلية نشر تلقائي لوركفلو n8n من GitHub إلى n8n Public API.

## المتطلبات
- n8n مُفعّل فيه Public API.
- Personal Access Token (API Key) من n8n.
- إمّا:
  - خادم n8n له عنوان عام يمكن لـ GitHub Actions الوصول إليه، أو
  - تشغيل **Self-hosted Runner** على نفس الجهاز الذي يعمل عليه n8n (بحيث يتصل على `http://127.0.0.1:5678`).

---

## 1) تفعيل n8n Public API
أوقف n8n ثم فعّل المتغيّرات التالية (أمثلة Windows):
```
setx N8N_PUBLIC_API_DISABLED "false"
setx N8N_HOST "127.0.0.1"
setx N8N_PORT "5678"
setx N8N_PROTOCOL "http"
```
أعد تشغيل n8n.

> إن كان n8n على خادم بعيد مع HTTPS، اضبط `N8N_HOST/N8N_PROTOCOL/N8N_PORT` بما يناسب.

## 2) إنشاء API Key (Personal Access Token)
- من واجهة n8n: **Settings → API (beta) → Create API Key**
  - **Label:** مثلاً: `GitHub Deploy`
  - **Expiration:** اختر `No Expiration` (أو حدّد مدة مناسبة لك).
  - **Scopes:** في النسخة المجانية قد لا يمكنك تعديلها، اتركها كما هي ثم **Create**.
- **انسخ التوكن** واحفظه في مكان آمن (لن يظهر مرة أخرى).

### اختبار سريع
جرّب من جهازك:
```
curl -H "X-N8N-API-KEY: <YOUR_TOKEN>" http://127.0.0.1:5678/api/v1/workflows
```
لو ظهر JSON هذا يعني الربط سليم.

## 3) ملف الوركفلو
الملف هنا: `workflows/sehaty.json`  
هذا هو JSON للوركفلو. عند أي تعديل و Push، سيتم نشره تلقائيًا لـ n8n.

> إن رغبت بإخفاء الحساسيات داخل الوركفلو (Set - Config)، استبدلها بتعابير بيئة n8n (مثال: `{{$env.TELEGRAM_BOT_TOKEN}}`) وعَرّف المتغيرات في تشغيل n8n (`.env` أو خدمة التشغيل).

## 4) أسرار GitHub (Secrets)
في المستودع: **Settings → Secrets and variables → Actions → New repository secret**
- `N8N_URL` = رابط خادمك (مثال: `http://127.0.0.1:5678` أو `https://your-n8n.example.com`)
- `N8N_API_KEY` = التوكن الذي أنشأته من n8n

## 5) تشغيل GitHub Actions
- أي Push على `workflows/sehaty.json` يشغّل نشرًا تلقائيًا.
- أو شغّل يدويًا من **Actions → Deploy to n8n → Run workflow**.

## 6) سيناريو الوصول
### أ) خادم n8n بعناون عام
يكفي ضبط `N8N_URL` للعنوان العام.

### ب) جهازك الشخصي (بدون عنوان عام)
شغّل **Self-hosted Runner** في GitHub:
- بالمستودع: **Settings → Actions → Runners → New self-hosted runner**
- اختر نظامك واتبع الخطوات (تحميل، تثبيت، run).
- بعد تشغيله، الـ Action سيتصل بـ n8n عبر `http://127.0.0.1:5678` بنجاح.

## 7) هيكل المستودع
```
.
├─ workflows/
│  └─ sehaty.json        # الوركفلو
└─ .github/
   └─ workflows/
      └─ deploy.yml      # GitHub Action للنشر
```

> احتاج أي تعديل إضافي؟ أخبرني وسأجهز لك التغييرات فورًا.
