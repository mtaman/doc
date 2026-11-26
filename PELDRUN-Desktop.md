# PELDRUN Desktop — دليل التركيب والتشغيل والإصدار

تطبيق سطح مكتب لـ PELDRUN: **Electron** يغلّف واجهة **Next.js** (وضع standalone) وباك إند **FastAPI** يعمل بـ **Python مدمج**.
يعمل على Windows وmacOS وLinux، ومعه **Docker** للخوادم.

> **اقرأ قسم «ما اختُبر وما لم يُختبر» قبل الاعتماد على أي شيء.** كل الاختبارات أُجريت على **Linux x64 فقط** داخل حاوية.
> Windows وmacOS وDocker وتشغيل GitHub Actions نفسها **لم تُجرَّب بعد**، وأول تشغيل لـ CI هو أول اختبار حقيقي لها.

---

## 1. محتويات الحزمة

```
desktop/                      الغلاف (Electron) وسكربتات البناء
  src/                        main.js, env.js (طبقة البيئة), update.js (التحديث), ports.json, splash.html
  scripts/                    prepare-env / prepare-code / prepare-frontend / after-pack / lock-env / smoke / backend-smoke
  locks/                      أقفال الحزم بإصدارات ثابتة وبصمات sha256، لكل منصة
  env-config.json             وسم Python المثبَّت وإعدادات البيئة
  electron-builder.config.cjs إعداد التحزيم (نسختان: lite وfull)
  build-resources/            entitlements لـ macOS
docker/                       backend.Dockerfile و frontend.Dockerfile
docker-compose.yml            تشغيل كل شيء بأمر واحد
.dockerignore
.github/workflows/            ci.yml (فحص كل طلب دمج) و desktop-release.yml (بناء الإصدارات)
patches/peldrun-repo-fixes.patch   إصلاحات لازمة في المستودع نفسه (انظر القسم 2)
```

## 2. التركيب على المستودع (مرة واحدة)

```bash
# من جذر المستودع، بعد فك الحزمة فوقه
git apply patches/peldrun-repo-fixes.patch
git add -A && git commit -m "Desktop packaging + cross-platform fixes"
```

ماذا يفعل الـ patch (ثلاثة تغييرات، وتحقّقتُ أنه يُطبَّق على نسخة نظيفة):

| الملف | التغيير | السبب |
|---|---|---|
| `backend/omweb/routers/status.py` | `import winreg` صار داخل `if sys.platform == "win32"` | كان يُسقط الباك إند عند الإقلاع على macOS وLinux |
| `frontend/next.config.mjs` | إضافة `output: "standalone"` **اختيارياً** عبر `PELDRUN_STANDALONE=1` | لا يتأثر `next dev` ولا `next start` |
| `frontend/src/app/api/memory-proxy/[...path]/route.ts` | حذف (كان ملفاً فارغاً في git) | كان يُفشل `next build` في فحص الأنواع، أي أن بناء الإنتاج لم يكن يعمل أصلاً |

**أمور يجب أن تفعلها بنفسك:**
1. في `config/config.toml` قيمتان حقيقيتا الشكل لـ `api_key` (السطران 6 و115). **أزلهما من المستودع وجدّد المفاتيح**، فهي موجودة في تاريخ git. سكربتات التحزيم وDocker تفرّغها تلقائياً من النسخة المشحونة، لكن هذا لا يمحوها من المستودع.
2. في `desktop/electron-builder.config.cjs` المالك والمستودع مضبوطان على `peldrun/peldrun`. غيّرهما إن اختلف.
3. **المستودع يجب أن يكون عاماً** (أو تستضيف الملفات في مكان آخر)، لأن التطبيق ينزّل بيئته وتحديثاته من روابط GitHub Releases دون مصادقة.

## 3. التطوير اليومي

**لا شيء يتغير.** استمر بالتطوير كما تفعل (`start-backend.ps1` و`next dev`). سكربتات `desktop/` أدوات بناء تقرأ الكود وقت البناء ولا تُشحن مع التطبيق.

ما يلتقطه البناء تلقائياً: أي ملف جديد داخل `backend/` أو `frontend/` أو `config/`.
ما يحتاج تدخلاً:

| إذا فعلت هذا | افعل هذا |
|---|---|
| أضفت مكتبة Python | عدّل `backend/requirements.txt` ثم `pip install uv` ثم `cd desktop && npm run lock` وارفع الأقفال. (ينتج معرّف بيئة جديد، فينزّل المستخدمون بيئة جديدة عند التحديث. هذا مقصود.) |
| أضفت ملفات أو مجلدات خارج `backend/frontend/config` | أضفها في `prepare-code.mjs` و`electron-builder.config.cjs` |
| كتبت ملفات بجانب الكود (مثل `config/` أو مساراً نسبياً) | تفشل داخل التثبيت. استخدم `PELDRUN_STORAGE_ROOT` أو مجلد بيانات المستخدم |
| استوردت وحدة خاصة بويندوز | تكسر macOS وLinux. فحص CI (`backend-smoke`) يلتقطه على الأنظمة الثلاثة |
| غيّرت المنفذين 8088/3088 | سطح المكتب يستخدم **18088/13088** (في `desktop/src/ports.json`) كي لا يتعارض مع نسخة التطوير. **لا تغيّرهما بعد النشر**: المنفذ جزء من الـ origin وتغييره يمحو localStorage والكوكيز |

## 4. بناء محلي (للتجربة قبل CI)

المتطلبات: Node 22، ومنصتك نفسها (البيئة خاصة بكل نظام)، واتصال بالإنترنت (تنزيل Python وخطوط Google وChromium).

```bash
cd desktop
npm ci

# أسرع للتجربة: بدون Chromium
export PELDRUN_SKIP_BROWSERS=1            # PowerShell: $env:PELDRUN_SKIP_BROWSERS="1"

node scripts/prepare-env.mjs              # Python مثبّت + الحزم من القفل (+ Chromium) + أرشيف البيئة
node scripts/prepare-code.mjs             # الباك إند والإعدادات (بعد تفريغ المفاتيح)
node scripts/prepare-frontend.mjs         # بناء الواجهة standalone

npm run dev                               # يشغّل التطبيق من مجلد build/ بدون مثبّت
```

لصنع المثبّتات ثم اختبار **الحزمة المبنية نفسها** (وليس وضع التطوير):

```bash
npm run dist                              # lite: المثبّتات في desktop/dist
export PELDRUN_ENV_ARCHIVE="$PWD/build/env-archive/$(ls build/env-archive | grep 'tar.gz$')"
npm run smoke                             # Linux: xvfb-run -a node scripts/smoke.mjs  (+ PELDRUN_SMOKE_NO_SANDBOX=1 كجذر)

# نسخة بدون إنترنت (تضمّ Python وChromium، حجمها أضعاف):
PELDRUN_FLAVOR=full npm run dist          # المخرجات في desktop/dist-full
```

## 5. الإصدار عبر GitHub Actions

**إعداد لمرة واحدة:**
- `Settings → Actions → General → Workflow permissions`: **Read and write**.
- الأسرار (كلها اختيارية؛ بدونها تُبنى حزم غير موقّعة):

| السر | الغرض |
|---|---|
| `MAC_CSC_LINK`, `MAC_CSC_KEY_PASSWORD` | شهادة Developer ID Application (ملف p12 بصيغة base64) |
| `APPLE_ID`, `APPLE_APP_SPECIFIC_PASSWORD`, `APPLE_TEAM_ID` | notarization لـ macOS |
| `WIN_CSC_LINK`, `WIN_CSC_KEY_PASSWORD` | توقيع Windows (شهادة pfx). الشهادات الحديثة غالباً لا تُصدَّر كملفات، فقد تحتاج خدمة توقيع سحابية وتعديل الإعداد |

**الإصدار:**
```bash
git tag v1.0.0 && git push origin v1.0.0
```
ماذا يحدث:
1. `prepare` يستخرج الرقم من التاج وينشئ **مسودة** إصدار.
2. `build` يعمل بالتوازي على: Linux x64 وarm64، وmacOS arm64 وIntel، وWindows x64. كل مشغّل يبني البيئة والكود والواجهة والمثبّتات، ثم **يشغّل التطبيق المحزَّم** في وضع الاختبار. فشله يوقف المهمة ولا يُرفع شيء من مشغّلها.
3. تُرفع البيئة إلى إصدار `env-<id>` (مرة واحدة لكل معرّف) والمثبّتات وملفات `latest*.yml` إلى المسودة.
4. `docker` ينشر صورتين إلى `ghcr.io/<owner>/peldrun-backend` و`peldrun-frontend`.
5. **أنت** تراجع المسودة وتضغط **Publish release**. لا يظهر شيء للمستخدمين قبلها.

تشغيل يدوي (تبويب Actions → Run workflow): يبني الرقم الذي تعطيه، ويحفظ المثبّتات كـ artifacts فقط ما لم تفعّل `publish`. خيار `build_full` يبني المثبّتات التي تعمل بدون إنترنت.

## 6. التركيب والتشغيل عند المستخدم

| النظام | الخطوات |
|---|---|
| **Windows** | شغّل `PELDRUN-x.y.z-win-x64.exe`. التثبيت لكل مستخدم دون صلاحيات إدارية. بدون توقيع تظهر شاشة SmartScreen («مزيد من المعلومات ← تشغيل على أي حال») |
| **macOS** | افتح `.dmg` واسحب التطبيق إلى Applications. غير الموقّع: بزر يمين ← فتح، أو `xattr -dr com.apple.quarantine /Applications/PELDRUN.app` |
| **Linux** | `chmod +x PELDRUN-*.AppImage && ./PELDRUN-*.AppImage` أو `sudo apt install ./PELDRUN-*.deb` |
| **Docker** | `docker compose up --build` ثم افتح http://127.0.0.1:3088 |

أول تشغيل لنسخة lite: يظهر شريط تقدم وهو ينزّل **بيئة Python** (حجم أرشيف Linux بدون Chromium: 259MB؛ مع Chromium أكبر، ولم أقس بقدر). تُحفظ في مجلد بيانات المستخدم وتُعاد استخدامها في كل التحديثات التي لا تغيّر معرّف البيئة. إن فشل التنزيل تظهر نافذة «إعادة المحاولة».

**أين تُحفظ البيانات** (`<userData>`): Windows `%APPDATA%\PELDRUN` · macOS `~/Library/Application Support/PELDRUN` · Linux `~/.config/PELDRUN`
- `peldrun-home/storage` المحادثات والذاكرة وقواعد البيانات · `peldrun-home/config` الإعدادات (تُزرع مرة ولا تُستبدل عند التحديث)
- `runtimes/<id>` بيئة Python · `logs/` (`desktop.log` و`backend.log` و`frontend.log`) · `backups/` نسخ قواعد البيانات قبل كل ترقية (آخر 3)

## 7. كيف يعمل (ثلاث طبقات)

| الطبقة | المحتوى | يتحدث متى | كيف |
|---|---|---|---|
| قشرة + كود | Electron والباك إند والواجهة | كل إصدار | `electron-updater` من GitHub Releases |
| **بيئة** | Python وكل المكتبات (+ Chromium) | عند تغيّر القفل فقط | أرشيف يُنزَّل مرة، والتحقق بـ sha256 المكتوبة داخل المثبّت نفسه |

- **معرّف البيئة** = بصمة (القفل + إصدار Python + المراجعة + الأعلام). نفس المعرّف يعني نفس البايتات، فلا إعادة تنزيل.
- **الإصدارات مثبّتة بالكامل:** `desktop/locks/*.txt` (إصدارات وبصمات لكل منصة)، ووسم Python في `env-config.json`، وElectron وelectron-builder بأرقام ثابتة.
- يُثبَّت من القفل بـ `pip install --require-hashes --no-deps`، فلا يُقبل أي ملف خارجه.
- **التحديث:** يفحص عند الإقلاع وكل 6 ساعات، ينزّل في الخلفية، ثم يسأل «أعد التشغيل الآن؟»، ويوقف العمليات الفرعية قبل التثبيت (وإلا يفشل مثبّت ويندوز).
  Windows وAppImage ينزّلان **الأجزاء المتغيرة فقط** (blockmap)، وmacOS ينزّل zip كاملاً، وحزمة `.deb` لا تتحدث ذاتياً.
- يُنقل الباك إند إلى `peldrun-home/backend` عند كل إصدار جديد، لأن الكود يكتب بجانب نفسه ولا يصلح مجلد تثبيت للقراءة فقط.

## 8. ما اختُبر وما لم يُختبر

**اختُبر فعلاً (Linux x64، داخل حاوية):**
- الباك إند يقلع بقفل الحزم المثبّت بالبصمات، و`/api/health` يعمل.
- تحضير البيئة والكود، وبناء الواجهة standalone وتشغيلها.
- Electron في وضع التطوير: يقلع الاثنين، البروكسي `/api` يعمل، لقطة شاشة تُظهر الواجهة الحقيقية، الإغلاق نظيف بلا عمليات يتيمة.
- **الحزمة المبنية lite:** التنزيل الفعلي عبر HTTP، عدم إعادة التنزيل في التشغيل الثاني، رفض أرشيف بصمته خاطئة.
- **الحزمة full** (بيئة مضمَّنة، بدون تنزيل) و**ملف AppImage نفسه**.
- أحجام فعلية (lite): AppImage 158MB، deb 125MB.
- فحص `actionlint` للـ workflows (كشف خطأً فعلياً أصلحته)، وصحة ملف compose، وأمر `sed` الذي يفرّغ المفاتيح في صورة Docker.

**لم يُختبر إطلاقاً:**
- **Windows وmacOS** بأي شكل، ولا ARM (Linux/macOS arm64).
- **تشغيل GitHub Actions فعلياً**، ولا رفع الإصدارات، ولا ghcr.
- **بناء صور Docker** (لا Docker في بيئتي). مهمة `docker-build` في `ci.yml` ستفحصها.
- **Chromium داخل البيئة:** تنزيله محجوب من بيئتي. أول CI سيبنيه (Chrome for Testing 159 مع playwright 1.65.2).
- **التوقيع وnotarization**، وتوقيع ad-hoc لملفات Mach-O في البيئة (كتبته ولم أجرّبه).
- **التحديث التلقائي** (`electron-updater`)، وقناة arm64 لـ macOS.
- **بناء الواجهة بخطوط Google الحقيقية** (استخدمت بديلاً في الاختبار فقط ثم أعدت الملف الأصلي).
- **أدوات الوكيل** (تنفيذ الكود، الصدفة، MCP، sandbox)، وتدفقات SSE الطويلة. اختبرتُ الإقلاع و`/api/health` فقط.
- تثبيت حزمة `.deb` نفسها.

## 9. مؤجَّل ومخاطر معروفة

**قرارات مؤجلة:**
1. **تحديث «الكود فقط» بنظام خاص** (طبقة كود منفصلة عن القشرة بتوقيعك ومفتاح عام مدمج وتراجع تلقائي). الحالي: `electron-updater`، وهو تفاضلي على Windows وAppImage فقط. على macOS كل تحديث zip كامل (القشرة مع الكود، دون البيئة).
2. **تعتيم/تشفير كود التطبيق** (Nuitka أو PyArmor للباك إند). الكود يُشحن مقروءاً. لا حماية حقيقية ممكنة، والمفاتيح لا توضع في الحزمة أبداً.
3. ترحيل (migrations) الإعدادات والقواعد بين الإصدارات: الموجود فقط **نسخ احتياطي** لملفات `*.db` قبل أول تشغيل لإصدار جديد.
4. استئناف التنزيل المتقطع (الموجود: 3 محاولات)، وخادم تنزيل بديل لشبكات الشركات التي تحجب GitHub. التنزيل يمر عبر شبكة Chromium (يدعم بروكسي النظام) لكن لم يُجرَّب خلف بروكسي.
5. توحيد رقم الإصدار: الواجهة تعرض `v1.0.0` من `VERSION.json` في المستودع بينما التطبيق يأخذ رقمه من التاج.
6. تثبيت actions بـ **commit SHA** (الآن بأرقام رئيسية مثل `@v6`)، وإشعارات تراخيص الطرف الثالث (THIRD-PARTY).
7. ويندوز ARM (`win32-arm64`) غير مدعوم حالياً. صور Docker لـ amd64 فقط (arm64 يحتاج تعديلاً في workflow، وهو بطيء تحت QEMU).

**مخاطر متوقعة عند أول تجربة:**
- **macOS:** الحزم المنزَّلة لا تمر بتوقيعك. Apple Silicon يرفض ملفات arm64 غير الموقّعة، وخطوة التوقيع المؤقت قد تحتاج ضبطاً. إن فشلت فالحل البديل تنزيل Chromium والبيئة وتوقيعها أو تضمينها في الحزمة الموقّعة.
- **Linux:** على Ubuntu 24.04 قد يحتاج AppImage خيار `--no-sandbox` (قيد AppArmor)، وحزمة `.deb` تتجنبه. Chromium قد يحتاج مكتبات نظام إضافية (`playwright install-deps` يتطلب root).
- **Windows:** SmartScreen بلا توقيع، ومضادات الفيروسات قد تعترض `python.exe` المنزَّل، والمسارات الطويلة (>260 حرفاً).
- **الحجم:** مع Chromium يكبر أرشيف البيئة كثيراً. المتوقع مئات الميجابايتات، لكنني لم أقسه.
- **الأمان في Docker:** الباك إند بلا مصادقة وCORS مفتوح. الـ compose ينشر الواجهة على `127.0.0.1` فقط. لا تفتحه للشبكة دون مصادقة. وربط `docker.sock` بالحاوية يعادل صلاحيات root على المضيف.
- **ذاكرة بناء الواجهة:** ذروتها 2.6GB (جهازي 4GB). مشغّلات GitHub أكبر من ذلك.
- مجلدات `backend/storage/store/...` الفارغة تظهر عند تشغيل الباك إند بمسار نسبي (أثر تشغيل، وليست من git). لا تُشحن، وتُنشأ وقت التشغيل.

## 10. استكشاف الأخطاء

| العَرَض | ما يُفحص |
|---|---|
| نافذة «Port 18088/13088 in use» | نسخة PELDRUN أخرى تعمل، أو برنامج يستخدم المنفذ |
| «runtime could not be downloaded or verified» | الإنترنت أو حجب GitHub، أو أن المستودع خاص. السجل: `logs/desktop.log`. لإعادة المحاولة من الصفر احذف `runtimes/` |
| يقلع ثم يتوقف | `logs/backend.log` و`logs/frontend.log` |
| اختبار CI فشل على الحزمة | شغّل `npm run smoke` محلياً. الأخطاء المتوقعة: `Cannot find module 'next'` تعني فشل `after-pack.cjs` |
| فحص `lock-check` فشل | غيّرت `requirements.txt` دون `npm run lock` |

متغيرات مفيدة: `PELDRUN_ENV_ARCHIVE` (أرشيف بيئة محلي، للاختبار أو لأجهزة بلا إنترنت)، `PELDRUN_DISABLE_UPDATES=1`، `PELDRUN_SKIP_BROWSERS=1` (وقت البناء)، `PELDRUN_NO_TRIM=1`.