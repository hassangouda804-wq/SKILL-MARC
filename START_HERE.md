# SKILL MARC — النسخة الكاملة (الواجهة + الـ Backend)

```
public/index.html      ← ملفك الأصلي كما هو (نفس الـ checksum، لم يُعدَّل)
bridge/sm-bridge.js    ← الجسر الذي يربط الصفحة بالخادم
src/ db/ scripts/      ← الـ Backend (API + قاعدة البيانات)
docker-compose.local.yml + .env.local  ← تشغيل محلي بأمر واحد
deploy/ docker-compose.yml             ← للنشر على سيرفر حقيقي (nginx + HTTPS)
ARCHITECTURE.md        ← شرح المعمارية والأمان وحدود النظام
```

## تشغيل على جهازك (الأسهل)
1. ثبّت **Docker Desktop**.
2. افتح terminal داخل هذا الفولدر ونفّذ:
   ```
   docker compose --env-file .env.local -f docker-compose.local.yml up --build
   ```
3. افتح **http://localhost:8088** — هذه صفحتك الأصلية متصلة بالخادم وقاعدة البيانات.
4. سجّل حساباً جديداً من الصفحة (يُنشأ في PostgreSQL حقيقي بكلمة مرور مشفّرة).

### إنشاء حساب أدمن (مرة واحدة)
```
docker compose --env-file .env.local -f docker-compose.local.yml exec \
  -e ADMIN_DATABASE_URL=postgres://skillmarc_owner:<OWNER_DB_PASSWORD من .env.local>@db:5432/skillmarc \
  api node dist/scripts/create-admin.js you@example.com "Your Name"
```
يطبع كلمة مرور مؤقتة ورابط TOTP (امسحه بتطبيق Authenticator). **لا توجد واجهة أدمن بعد**؛ العمليات (تأكيد الشحن…) عبر `/api/admin/*` (انظر ARCHITECTURE.md §4).

### بدون Docker
تحتاج Node 22 وPostgreSQL 16: أنشئ قاعدة `skillmarc`، ثم  
`psql -v app_pw=... -f db/roles.sql -f db/migrations/001_init.sql -f db/seed.sql`  
ثم اضبط متغيرات `.env.local` (DATABASE_URL إلخ) و`npm ci && npm run local`.

## قبل أي استخدام حقيقي
- **أرقام الاستلام** (Vodafone Cash/InstaPay…) في `public/index.html` ما زالت `01XXXXXXXXX`؛ عدّلها بأرقامك. بعدها أعد توليد البيانات: `node scripts/extract-catalog.mjs public/index.html https://نطاقك`.
- `ADMIN_PASS` داخل الملف لم يعد يفعل شيئاً (لوحة الأدمن المضمّنة معطّلة)، يُفضّل حذفه.
- للنشر على نطاق حقيقي استخدم `docker-compose.yml` + `deploy/nginx.conf` + HTTPS، وولّد أسراراً جديدة من `.env.example`.
- ما اختُبر وما لم يُختبر (متصفح حقيقي، Docker) مذكور بصراحة في ARCHITECTURE.md §6.
