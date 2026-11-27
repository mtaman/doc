# دليل استخدام السكربتات — مع أمثلة بسيطة

هذا الدليل يشرح **كل أمر وكل خيار** في سكربتات القالب، مع مثال بسيط ومخرجات متوقعة. الشرح بالعربية، والأوامر والمخرجات بالإنجليزية كما هي في الملفات.

> **ملاحظة مهمة عن الدقة:** الأوامر أدناه مطابقة لما هو **مُنفَّذ فعلياً** في نسخة القالب التي سلّمتها لك في هذه الجلسة. بعض السكربتات في القالب مبسّطة مقارنة بالمستودع المرجعي `peldrun-provider-cloudflare-ai`، وقد وضعت جدولاً في النهاية (§12) يوضح الفرق حتى لا تتوقع سلوكاً غير موجود.

---

## 0. تجهيز البيئة والمثال الموحّد

كل الأمثلة تستخدم مزوداً وهمياً واحداً لتسهيل المتابعة:

| العنصر | القيمة |
|--------|--------|
| Provider ID | `demo-ai` |
| Snake case | `demo_ai` |
| اسم الحزمة | `demo_ai_peldrun` |
| اسم الكلاس | `DemoAiAdapter` |
| الاسم المعروض | `Demo AI` |
| اللون | `#3B82F6` |
| الأيقونة | `demo` |

### تجهيز بيئة العمل

```bash
python -m venv .venv
```

```bash
# Linux / macOS
source .venv/bin/activate
```

```powershell
# Windows PowerShell
.venv\Scripts\Activate.ps1
```

```bash
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
python -m pip install "pytest>=8.0,<9.0"
```

**كل الأوامر تُنفَّذ من جذر المستودع** (وليس من داخل `scripts/`)، لأن السكربتات تحسب الجذر بـ `Path(__file__).resolve().parent.parent`.

---

## 1. جدول الأوامر السريع (Cheat Sheet)

| الأمر | الوظيفة |
|-------|---------|
| `python scripts/bootstrap_provider.py --id demo-ai` | تهيئة القالب واستبدال كل المتغيرات (مرة واحدة) |
| `python scripts/validate_manifest.py` | فحص صحة `manifest.json` |
| `python scripts/update_version.py patch` | رفع رقم الإصدار |
| `python scripts/update_version.py --check` | فحص الإصدار (نسخة القالب: stub) |
| `python scripts/changelog.py 0.1.1` | توليد مقطع CHANGELOG منコミتات Git |
| `python scripts/build_package.py` | بناء ملف ZIP |
| `python scripts/verify_package.py` | فحص أمان الـ ZIP وطباعة SHA-256 |
| `python scripts/generate_catalog_entry.py --download-url "..."` | توليد `catalog-entry.json` |
| `python -m pytest tests -v` | تشغيل الاختبارات |

**الأوامر التي تدعم `--help` فعلاً:** `bootstrap_provider.py` و `generate_catalog_entry.py` فقط (لأنهما يستخدمان `argparse`). باقي السكربتات لا تدعمه.

---

## 2. `bootstrap_provider.py` — التهيئة التلقائية

### الصيغة

```bash
python scripts/bootstrap_provider.py --id <provider-id> [options]
```

### كل الخيارات

| الخيار | إلزامي | الافتراضي | الوظيفة |
|--------|--------|-----------|---------|
| `--id` | ✅ | — | معرّف المزود (أحرف صغيرة) |
| `--display-name` | ❌ | مشتق من `--id` | الاسم المعروض في المتجر |
| `--author` | ❌ | `PELDRUN Community` | اسم المؤلف |
| `--author-url` | ❌ | `""` | رابط المؤلف |
| `--homepage-url` | ❌ | `""` | صفحة المزود |
| `--icon` | ❌ | snake_case من الـ id | اسم الأيقونة |
| `--brand-color` | ❌ | `#000000` | لون الهوية بصيغة hex |
| `--dry-run` | ❌ | مغلق | معاينة بدون كتابة |

### مثال 1: أبسط استخدام ممكن

```bash
python scripts/bootstrap_provider.py --id demo-ai
```

يشتق تلقائياً: `demo_ai_peldrun` / `DemoAiAdapter` / `Demo AI` / أيقونة `demo_ai` / لون `#000000`.

### مثال 2: معاينة قبل التنفيذ (مستحسن دائماً)

```bash
python scripts/bootstrap_provider.py --id demo-ai --dry-run
```

**المخرجات:**

```text
PELDRUN provider bootstrap
  id:           demo-ai
  package:      demo_ai_peldrun
  adapter:      DemoAiAdapter
  display name: Demo AI
  mode:         dry-run (no writes)

  [dry] manifest.json (9 replacements)
  [dry] pyproject.toml (3 replacements)
  [dry] renamed src/{{PROVIDER_ID_SNAKE}}_peldrun -> src/demo_ai_peldrun

Done: 9 file(s) updated, 24 replacement(s), 1 directory rename(s).
```

لا يُكتب أي ملف ولا يُعاد تسمية أي مجلد في هذا الوضع.

### مثال 3: تخصيص الاسم المعروض

```bash
python scripts/bootstrap_provider.py --id demo-ai --display-name "Demo AI Platform"
```

الفرق: `--id` يبقى `demo-ai` (تقني)، والاسم المعروض يصبح `Demo AI Platform` (تسويقي).

### مثال 4: المؤلف والروابط

```bash
python scripts/bootstrap_provider.py \
  --id demo-ai \
  --author "Demo Team" \
  --author-url "https://github.com/demoteam" \
  --homepage-url "https://demo.example.com/docs"
```

يملأ `{{AUTHOR_NAME}}` و `{{AUTHOR_URL}}` و `{{HOMEPAGE_URL}}` في `manifest.json` و `LICENSE` و `pyproject.toml`.

### مثال 5: اللون والأيقونة

```bash
python scripts/bootstrap_provider.py --id demo-ai --brand-color "#3B82F6" --icon demo
```

بفضل خطوة **Legacy Literal Normalization** يتحول السطران في `release.yml` تلقائياً:

```yaml
--brand-color "#3B82F6" \
--icon demo
```

 Without أي تعديل يدوي منك.

### مثال 6: الأمر الكامل

```bash
python scripts/bootstrap_provider.py \
  --id demo-ai \
  --display-name "Demo AI" \
  --author "Demo Team" \
  --author-url "https://github.com/demoteam" \
  --homepage-url "https://demo.example.com/docs" \
  --brand-color "#3B82F6" \
  --icon demo
```

### ما ينفّذه تلقائياً

1. يستبدل كل التوكنات في كل الملفات النصية.
2. يعيد تسمية `src/{{PROVIDER_ID_SNAKE}}_peldrun/` → `src/demo_ai_peldrun/`.
3. يصلح المسار الثابت `CHANGE_ME_peldrun` في `update_version.py`.
4. يهرّب القيم الخاصة داخل ملفات `.json` (يحافظ على صحة JSON).
5. يضيف نفسه إلى `.gitignore`.
6. يفحص وجود توكنات متبقية ويبلغ عنها.

### أخطاء يرفضها السكربت (أمثلة)

```bash
python scripts/bootstrap_provider.py --id "Demo AI"
```
```text
[err] Invalid provider ID 'Demo AI'. Must match ^[a-z0-9][a-z0-9._-]*$
```

```bash
python scripts/bootstrap_provider.py --id ollama
```
```text
[err] Provider ID 'ollama' collides with a built-in adapter.
```

```bash
python scripts/bootstrap_provider.py --id demo-ai --brand-color blue
```
```text
[err] --brand-color must be a hex color like '#F38020', got 'blue'
```

### رموز الخروج

| الرمز | المعنى |
|-------|--------|
| `0` | نجاح |
| `1` | معاملات غير صالحة أو توكنات متبقية |
| `2` | خطأ في القراءة/الكتابة |

### تنبيهان عمليان

- **التوكنات في الوثائق تُستبدل أيضاً.** إذا كان `README.md` يحتوي جدولاً يشرح `{{PROVIDER_ID}}` فسيتم استبداله بالقيمة الفعلية. الحل: احفظ هذا الدليل خارج المستودع، أو أضف `docs` إلى `SKIP_DIRS` في السكربت قبل التشغيل.
- **إعادة التشغيل آمنة على المحتوى**، لكن مع القيم الافتراضية (مثل `#000000`) قد يطبع سطر استبدال في `release.yml` لأن خطوة التطبيع تكتب ثم تُعاد القيمة نفسها؛ الملف النهائي لا يتغير.

---

## 3. `validate_manifest.py` — فحص المنفيست

### الصيغة

```bash
python scripts/validate_manifest.py
```

**لا يقبل أي خيارات أو مسار ملف** في نسخة القالب؛ يقرأ دائماً `manifest.json` من الجذر.

### مثال ناجح

```bash
python scripts/validate_manifest.py
```
```text
[ok] manifest.json is valid.
```

### مثال فاشل

إذا كان `transport: "native"` بدون `entry_point`:

```text
[err] Native transport requires 'entry_point' (module:ClassName)
```

### ما يفحصه

- وجود الحقول: `schema_version`, `kind`, `id`, `name`, `version`, `transport`, `permissions`
- نمط الـ `id` بأحرف صغيرة
- الإصدار بصيغة SemVer
- وجود `entry_point` عند `native` / `custom`
- `permissions.workspace_access` و `shell_execute` يجب أن تكون `false` صراحةً

---

## 4. `update_version.py` — إدارة الإصدار

### الصيغة

```bash
python scripts/update_version.py <major|minor|patch|X.Y.Z|--check>
```

### مثال 1: رفع patch

```bash
python scripts/update_version.py patch
```
```text
Version updated: 0.1.0 -> 0.1.1
```

### مثال 2: رفع minor

```bash
python scripts/update_version.py minor
```
```text
Version updated: 0.1.1 -> 0.2.0
```

### مثال 3: رفع major

```bash
python scripts/update_version.py major
```
```text
Version updated: 0.2.0 -> 1.0.0
```

### مثال 4: إصدار محدد

```bash
python scripts/update_version.py 1.4.2
```
```text
Version updated: 1.0.0 -> 1.4.2
```

### مثال 5: الفحص

```bash
python scripts/update_version.py --check
```
```text
Version check passed (stub).
```

### مثال 6: معامل خاطئ

```bash
python scripts/update_version.py beta
```
```text
Usage: update_version.py <major|minor|patch|X.Y.Z>
```

### ⚠️ حدود نسخة القالب (مهم)

| Behavior | الحالة في القالب |
|----------|------------------|
| تحديث `manifest.json` | ✅ منفَّذ |
| تحديث `pyproject.toml` | ✅ منفَّذ |
| تحديث `__init__.py` | ❌ غير منفَّذ (المتغير `INIT_PATH` معرّف لكن غير مستخدم) |
| إضافة مقطع لـ `CHANGELOG.md` | ❌ غير منفَّذ |
| `--check` يفحص التطابق فعلاً | ❌ stub يطبع نجاحاً دائماً |

**لذلك بعد كل رفع إصدار، عدّل يدوياً:**

```python
# src/demo_ai_peldrun/__init__.py
__version__ = "0.1.1"
```

---

## 5. `changelog.py` — توليد ملاحظات الإصدار

### الصيغة

```bash
python scripts/changelog.py [version]
```

إذا حذفت الرقم يستخدم `Unreleased`.

### مثال 1

```bash
python scripts/changelog.py 0.1.1
```

**المخرجات:**

```text
Generated CHANGELOG section:
## [0.1.1] - YYYY-MM-DD

### Added
- add streaming support

### Fixed
- handle empty choices chunk
```

### مثال 2: حفظ المخرجات في ملف

السكربت **يطبع فقط** ولا يكتب في `CHANGELOG.md`:

```bash
python scripts/changelog.py 0.1.1 > section.md
```

ثم انسخ المقطع يدوياً إلى أعلى `CHANGELOG.md` واستبدل `YYYY-MM-DD` بالتاريخ الحقيقي.

### تصنيف الـ Commits

| بادئة | التصنيف |
|-------|---------|
| `feat:` | Added |
| `fix:` | Fixed |
| `refactor:` / `perf:` / `chore:` | Changed |
| غير ذلك | مُتجاهَل |

### أمثلة رسائل تصلح للتصنيف

```bash
git commit -m "feat: add model discovery"
git commit -m "fix: handle 429 rate limit"
git commit -m "perf: cache model list per connection"
```

### أوامر Git التي يعتمد عليها

```bash
git describe --tags --abbrev=0
```
```bash
git log v0.1.0..HEAD --pretty=format:%s
```

إذا لم توجد tags أو لم يكن المستودع git، يعيد قائمة فارغة ويخرج مقطعاً بلا بنود.

---

## 6. `build_package.py` — بناء الحزمة

### الصيغة

```bash
python scripts/build_package.py
```

**لا خيارات** في نسخة القالب (`--verify` غير مدعوم؛ شغّل سكربت التحقق منفصلاً).

### مثال

```bash
python scripts/build_package.py
```
```text
Built /path/to/repo/dist/demo-ai-0.1.0.zip
```

### التحقق من محتوى الحزمة

```bash
python -m zipfile -l dist/demo-ai-0.1.0.zip
```

**المخرجات المتوقعة:**

```text
File Name                                             Modified             Size
manifest.json                                    2020-01-01 00:00:00         1024
requirements.txt                                 2020-01-01 00:00:00          128
demo_ai_peldrun/__init__.py                      2020-01-01 00:00:00          256
demo_ai_peldrun/adapter.py                       2020-01-01 00:00:00         4096
demo_ai_peldrun/py.typed                         2020-01-01 00:00:00            0
```

لاحظ: التاريخ ثابت `2020-01-01` (بناء قابل للتكرار)، والحزمة **مُسطَّحة** بدون `src/`.

### ⚠️ تنبيه المعرّف المنقوط

السكربت يشتق اسم المجلد بـ `id.replace('-', '_')` فقط، بينما `bootstrap` يستبدل `-` و `.` معاً. لذلك معرّف مثل `acme.ai` يجعل السكربت يبحث عن `src/acme.ai_peldrun` غير الموجود، فيبني ZIP بدون كود.

**الحل الأسهل:** استخدم شرطات فقط (`demo-ai`) وتجنّب النقاط. أو عدّل السطر في `build_package.py`:

```python
src_pkg = BASE_DIR / "src" / f"{data['id'].replace('-', '_').replace('.', '_')}_peldrun"
```

---

## 7. `verify_package.py` — فحص الأمان والهاش

### الصيغة

```bash
python scripts/verify_package.py
```

**لا يقبل مسار ملف ولا `--expected-sha256`** في نسخة القالب؛ يختار تلقائياً آخر `.zip` يجده في `dist/`.

### مثال

```bash
python scripts/verify_package.py
```
```text
Verifying demo-ai-0.1.0.zip...
  sha256: 3f2a9c1b7d4e5a6f8091b2c3d4e5f60718293a4b5c6d7e8f9012a3b4c5d6e7f8
[ok] ZIP verification passed. Safe for PELDRUN.
```

### ما يفحصه

- وجود `manifest.json` في جذر الأرشيف
- عدم وجود `..` أو مسار يبدأ بـ `/`
- عدم وجود symlinks
- طباعة SHA-256

### نصيحة عملية

نظّف `dist/` قبل البناء حتى لا يفحص ملفاً قديماً:

```bash
rm dist/*.zip
python scripts/build_package.py
python scripts/verify_package.py
```

### استخدام الهاش

انسخ قيمة `sha256` واستخدمها في `catalog-entry.json` (انظر §8). والتنسيق `  sha256: <hex>` مطابق لما يلتقطه `release.yml` بـ `grep`.

---

## 8. `generate_catalog_entry.py` — بيانات المتجر

### الصيغة

```bash
python scripts/generate_catalog_entry.py --download-url "<url>" [--brand-color "#RRGGBB"] [--icon <slug>]
```

| الخيار | إلزامي | الافتراضي |
|--------|--------|-----------|
| `--download-url` | ✅ | — |
| `--brand-color` | ❌ | `#000000` |
| `--icon` | ❌ | `default` |

### مثال 1: الحد الأدنى

```bash
python scripts/generate_catalog_entry.py \
  --download-url "https://github.com/demoteam/peldrun-provider-demo-ai/releases/download/v0.1.0/demo-ai-0.1.0.zip"
```

### مثال 2: مع اللون والأيقونة

```bash
python scripts/generate_catalog_entry.py \
  --download-url "https://github.com/demoteam/peldrun-provider-demo-ai/releases/download/v0.1.0/demo-ai-0.1.0.zip" \
  --brand-color "#3B82F6" \
  --icon demo
```
```text
Generated /path/to/repo/dist/catalog-entry.json
```

### مثال 3: نسيان الخيار الإلزامي

```bash
python scripts/generate_catalog_entry.py --icon demo
```
```text
error: the following arguments are required: --download-url
```

### ⚠️ حقن الهاش الحقيقي (خطوة ضرورية)

نسخة القالب تكتب قيمة ثابتة:

```json
"checksum_sha256": "PLACEHOLDER_SHA256_FROM_VERIFY_SCRIPT"
```

استبدلها بالهاش الفعلي بهذا الأمر (bash):

```bash
python - <<'PY'
import hashlib, json, pathlib
dist = pathlib.Path("dist")
zip_path = sorted(dist.glob("*.zip"))[-1]
sha = hashlib.sha256(zip_path.read_bytes()).hexdigest()
entry_path = dist / "catalog-entry.json"
entry = json.loads(entry_path.read_text(encoding="utf-8"))
entry["checksum_sha256"] = sha
entry_path.write_text(json.dumps(entry, indent=2) + "\n", encoding="utf-8")
print("checksum_sha256 =", sha)
PY
```

> على Windows: شغّل نفس الكود داخل `python` تفاعلياً، أو احفظه في ملف مؤقت ونفّذه.

---

## 9. سير العمل الكامل — مثال متسلسل

```bash
# 1) Initialize
python scripts/bootstrap_provider.py --id demo-ai --display-name "Demo AI" --brand-color "#3B82F6" --icon demo

# 2) Implement manifest.json + src/demo_ai_peldrun/adapter.py + requirements.txt

# 3) Validate + test
python scripts/validate_manifest.py
python -m pytest tests -v

# 4) Bump version (then update __init__.py manually)
python scripts/update_version.py patch

# 5) Changelog (printed only)
python scripts/changelog.py 0.1.1

# 6) Build + verify
rm -f dist/*.zip
python scripts/build_package.py
python scripts/verify_package.py

# 7) Catalog entry + real checksum
python scripts/generate_catalog_entry.py \
  --download-url "https://github.com/demoteam/peldrun-provider-demo-ai/releases/download/v0.1.1/demo-ai-0.1.1.zip" \
  --brand-color "#3B82F6" --icon demo

# 8) Commit + tag + push
git add -A
git commit -m "feat: initial demo-ai provider"
git tag -a v0.1.1 -m "Release v0.1.1"
git push origin main
git push origin v0.1.1
```

---

## 10. أوامر Git و CI

### إنشاء الوسم الذي يشغّل النشر

```bash
git tag -a v0.1.1 -m "Release v0.1.1"
git push origin v0.1.1
```

**شرط أساسي:** الرقم في الوسم بدون `v` يجب أن يساوي `manifest.json["version"]` وإلا فشل `release.yml` قبل إنشاء أي Release.

### التحقق قبل الوسم

```bash
python -c "import json; print(json.load(open('manifest.json'))['version'])"
```

### تشغيل الـ workflows يدوياً

`validate.yml` و `release.yml` يدعمان `workflow_dispatch` من تبويب **Actions** في GitHub. عند تشغيل `release.yml` يدوياً يجب إدخال خانة `tag` بوسم موجود مسبقاً.

---

## 11. أخطاء شائعة وحلولها

| الخطأ | السبب | الحل |
|-------|-------|------|
| `manifest.json not found` | تشغيل الأمر من داخل `scripts/` | نفّذ من جذر المستودع |
| `No ZIP files found in dist/` | لم تبنِ الحزمة | `python scripts/build_package.py` |
| ZIP بدون كود المزود | اسم المجلد لا يطابق المشتق من الـ id | استخدم شرطات فقط، أو صحّح سطر `src_pkg` |
| `Tag version does not match manifest` | نسيت رفع الإصدار | `python scripts/update_version.py 0.1.1` ثم commit و tag |
| `checksum_sha256` بقي PLACEHOLDER | لم تحقن الهاش | نفّذ أمر §8 |
| الإصدار مختلف بين الملفات | `--check` في القالب stub | قارن يدوياً أو طوّر السكربت |
| توكنات متبقية بعد bootstrap | خطأ كتابي في التوكن | صحّح التوكن وأعد التشغيل |

---

## 12. جدول الدقة: القالب مقابل المستودع المرجعي

| الميزة | القالب (هذه الجلسة) | المرجعي (cloudflare-ai) |
|--------|---------------------|------------------------|
| `validate_manifest.py` يقبل مسار ملف | ❌ | ✅ |
| `update_version.py --check` فحص حقيقي | ❌ stub | ✅ |
| تحديث `__init__.py` + `CHANGELOG` عند الرفع | ❌ | ✅ |
| `build_package.py --verify` | ❌ | ✅ |
| `verify_package.py` يقبل مسار + `--expected-sha256` | ❌ | ✅ |
| حدود الحجم/النسبة/عدد الملفات في التحقق | ❌ | ✅ |
| `generate_catalog_entry.py` يكتب الهاش تلقائياً | ❌ | ✅ (حسب الإعداد) |
| `merge_catalog_entry.py` | ❌ غير موجود | ✅ |
| `bootstrap_provider.py` + التطبيع التلقائي | ✅ | ❌ (يستخدم `scaffold.ps1`) |

إذا أردت، أرقّي أي سكربت من عمود «القالب» ليطابق سلوك المرجعي — مع فحص الكود المصدري للمرجعي أولاً لإثبات العقد قبل الكتابة.
