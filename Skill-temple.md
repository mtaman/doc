# 📘 دليل شامل لاستخدام سكربتات أتمتة مهارات PELDRUN

هذا الدليل يشرح كل سكربت بالتفصيل مع أمثلة عملية وسير العمل الكامل.

---

## 📋 نظرة عامة على السكربتات

| السكربت | الوظيفة | متى يستخدم |
|---------|---------|-----------|
| `validate_manifest.py` | التحقق من صحة manifest.json | قبل كل commit |
| `build_package.py` | بناء حزمة ZIP | قبل الإطلاق |
| `verify_package.py` | التحقق من حزمة ZIP | بعد البناء |
| `generate_catalog_entry.py` | توليد مدخل الكتالوج | عند الإطلاق |
| `update_version.py` | تحديث رقم الإصدار | قبل tag جديد |
| `changelog.py` | توليد سجل التغييرات | قبل الإطلاق |

---

## 🔧 التثبيت الأولي

```bash
# Clone the template repository
git clone https://github.com/peldrunai/peldrun-skill-template.git
cd peldrun-skill-template

# Create virtual environment
python -m venv .venv
source .venv/bin/activate  # Linux/Mac
# or
.venv\Scripts\activate     # Windows

# Install dependencies
pip install jsonschema==4.23.0 pyyaml==6.0.2
```

---

## 1️⃣ validate_manifest.py - التحقق من صحة Manifest

### الوصف
يتحقق من أن ملف `manifest.json` يطابق مخطط PELDRUN الرسمي. يكشف الأخطاء قبل الوصول إلى المُثبِّت.

### التحقق من
- بنية JSON الصحيحة
- تطابق مع JSON Schema
- وجود ملفات التعليمات (SKILL.md)
- صحة sub-skills
- عدم وجود ملفات محظورة (.exe, .pyc, إلخ)
- حدود الحجم وعدد الملفات

### الاستخدام

```bash
# التحقق من مهارة واحدة
python scripts/validate_manifest.py <skill_id>

# التحقق من جميع المهارات
python scripts/validate_manifest.py --all

# التحقق من مسار محدد
python scripts/validate_manifest.py path/to/manifest.json
```

### أمثلة عملية

#### مثال 1: التحقق من مهارة بسيطة
```bash
$ python scripts/validate_manifest.py pdf_document
[pdf_document] OK
```

#### مثال 2: التحقق من جميع المهارات
```bash
$ python scripts/validate_manifest.py --all
Validating 6 skill(s)...

[pdf_document] OK
[docx_document] OK
[xlsx_spreadsheet] OK
[pptx_presentation] OK
[csv_data] OK
[markdown_doc] OK

============================================================
Total: 6 skill(s) validated
```

#### مثال 3: خطأ في الـ manifest
```bash
$ python scripts/validate_manifest.py my_skill
[my_skill] ERROR: Schema: id: 'My-Skill' does not match '^[a-z0-9][a-z0-9_]{2,63}$'
[my_skill] ERROR: Schema: version: '1.0' does not match '^\d+\.\d+\.\d+(?:-[a-z0-9.\-]+)?$'
[my_skill] FAILED (2 error(s))
```

### متى تستخدم هذا السكربت؟
- ✅ قبل كل `git commit`
- ✅ في CI/CD (GitHub Actions)
- ✅ عند إضافة مهارة جديدة
- ✅ عند تعديل manifest.json

---

## 2️⃣ build_package.py - بناء حزمة ZIP

### الوصف
يبني حزمة ZIP قابلة للتثبيت. يضمن أن الملفات في جذر الـ ZIP (وليس في مجلد فرعي).

### المخرجات
```
dist/<skill_id>-<version>.zip
```

### الاستخدام

```bash
# بناء مهارة واحدة
python scripts/build_package.py <skill_id>

# بناء جميع المهارات
python scripts/build_package.py --all

# بناء مع التحقق
python scripts/build_package.py <skill_id> --verify
```

### أمثلة عملية

#### مثال 1: بناء مهارة واحدة
```bash
$ python scripts/build_package.py pdf_document
Building: pdf_document v1.0.1
Source:   skills/pdf_document
Output:   dist/pdf_document-1.0.1.zip
Built:    pdf_document-1.0.1.zip (15,234 bytes, 8 files)
```

#### مثال 2: بناء جميع المهارات
```bash
$ python scripts/build_package.py --all

============================================================
Building: pdf_document v1.0.1
Source:   skills/pdf_document
Output:   dist/pdf_document-1.0.1.zip
Built:    pdf_document-1.0.1.zip (15,234 bytes, 8 files)

============================================================
Building: docx_document v1.0.1
Source:   skills/docx_document
Output:   dist/docx_document-1.0.1.zip
Built:    docx_document-1.0.1.zip (12,456 bytes, 7 files)

============================================================
Successfully built 2 package(s)
```

#### مثال 3: بناء مع التحقق
```bash
$ python scripts/build_package.py pdf_document --verify
Building: pdf_document v1.0.1
Source:   skills/pdf_document
Output:   dist/pdf_document-1.0.1.zip
Built:    pdf_document-1.0.1.zip (15,234 bytes, 8 files)

Verifying package...
[pdf_document-1.0.1.zip] VERIFICATION PASSED
  Skill ID:  pdf_document
  Version:   1.0.1
```

### متى تستخدم هذا السكربت؟
- ✅ قبل إنشاء tag جديد
- ✅ في CI/CD عند push tag
- ✅ عند اختبار مهارة محلياً

---

## 3️⃣ verify_package.py - التحقق من حزمة ZIP

### الوصف
يتحقق من أن حزمة ZIP المبنية تطابق متطلبات PELDRUN عند التثبيت. يعكس تماماً ما يفعله المُثبِّت.

### التحقق من
- صلاحية ZIP
- وجود manifest.json في الجذر
- تطابق مع Schema
- وجود ملفات sub-skills
- عدم وجود ملفات محظورة
- عدم وجود path traversal
- حدود الحجم

### الاستخدام

```bash
# التحقق من حزمة
python scripts/verify_package.py dist/<skill_id>-<version>.zip
```

### أمثلة عملية

#### مثال 1: حزمة صحيحة
```bash
$ python scripts/verify_package.py dist/pdf_document-1.0.1.zip
[pdf_document-1.0.1.zip] VERIFICATION PASSED
  Skill ID:  pdf_document
  Version:   1.0.1
```

#### مثال 2: حزمة بها خطأ
```bash
$ python scripts/verify_package.py dist/broken-skill.zip
[broken-skill.zip] ERROR: manifest.json not found at ZIP root
[broken-skill.zip] ERROR: Instructions file not in ZIP: SKILL.md
[broken-skill.zip] VERIFICATION FAILED (2 error(s))
```

#### مثال 3: حزمة router مع sub-skills
```bash
$ python scripts/verify_package.py dist/code_review-2.0.0.zip
[code_review-2.0.0.zip] VERIFICATION PASSED
  Skill ID:  code_review
  Version:   2.0.0
  Sub-skills: 4
```

### متى تستخدم هذا السكربت؟
- ✅ بعد `build_package.py` (مع `--verify`)
- ✅ عند التحقق اليدوي قبل الإطلاق
- ✅ في CI/CD للتأكد من صحة الحزم

---

## 4️⃣ generate_catalog_entry.py - توليد مدخل الكتالوج

### الوصف
يولد ملف JSON متوافق مع كتالوج PELDRUN للمهارات. يتضمن SHA-256 للحزمة للتحقق من سلسلة التوريد.

### المخرجات
```
dist/<skill_id>-catalog-entry.json
```

### الاستخدام

```bash
# توليد مدخل مع URL يدوي
python scripts/generate_catalog_entry.py \
    --skill <skill_id> \
    --zip <path_to_zip> \
    --download-url <url> \
    --repo <repository_url>

# توليد مدخل من GitHub
python scripts/generate_catalog_entry.py \
    --skill <skill_id> \
    --github-owner <owner> \
    --github-repo <repo>
```

### أمثلة عملية

#### مثال 1: توليد مدخل مع URL يدوي
```bash
$ python scripts/generate_catalog_entry.py \
    --skill pdf_document \
    --zip dist/pdf_document-1.0.1.zip \
    --download-url "https://github.com/peldrunai/peldrun-office-skills/releases/download/pdf_document-v1.0.1/pdf_document-1.0.1.zip" \
    --repo "https://github.com/peldrunai/peldrun-office-skills"

Generating catalog entry for: pdf_document v1.0.1
ZIP: dist/pdf_document-1.0.1.zip
Download URL: https://github.com/peldrunai/peldrun-office-skills/releases/download/pdf_document-v1.0.1/pdf_document-1.0.1.zip

Generated: dist/pdf_document-catalog-entry.json
SHA-256: 8ae7a6b4bc94c39a6d8ab31e25a051fdc5f8b854de2d635fb1516a51fda1711e
```

#### مثال 2: توليد مدخل من GitHub (تلقائي)
```bash
$ python scripts/generate_catalog_entry.py \
    --skill pdf_document \
    --github-owner peldrunai \
    --github-repo peldrun-office-skills

Generating catalog entry for: pdf_document v1.0.1
ZIP: dist/pdf_document-1.0.1.zip
Download URL: https://github.com/peldrunai/peldrun-office-skills/releases/download/pdf_document-v1.0.1/pdf_document-1.0.1.zip

Generated: dist/pdf_document-catalog-entry.json
SHA-256: 8ae7a6b4bc94c39a6d8ab31e25a051fdc5f8b854de2d635fb1516a51fda1711e
```

#### مثال 3: توليد مدخل لمهارة router
```bash
$ python scripts/generate_catalog_entry.py \
    --skill code_review \
    --github-owner peldrunai \
    --github-repo peldrun-skill-code-review

Generating catalog entry for: code_review v2.0.0
ZIP: dist/code_review-2.0.0.zip
Download URL: https://github.com/peldrunai/peldrun-skill-code-review/releases/download/code_review-v2.0.0/code_review-2.0.0.zip

Generated: dist/code_review-catalog-entry.json
SHA-256: c4001b97ed4ebbbea09db9645359499180d83da3fa3aecda3fcc8a3f1450cf43
Sub-skills: 4
```

### محتوى الملف المولد
```json
{
  "id": "pdf_document",
  "name": "PDF Document Generator",
  "version": "1.0.1",
  "description": "Generate PDF reports with full Unicode support",
  "download_url": "https://github.com/.../pdf_document-1.0.1.zip",
  "checksum_sha256": "8ae7a6b4bc94c39a6d8ab31e25a051fdc5f8b854de2d635fb1516a51fda1711e",
  "has_sub_skills": false,
  "sub_skills_count": 0,
  "trust": "community",
  "stability": "stable",
  "release_date": "2026-11-28"
}
```

### متى تستخدم هذا السكربت؟
- ✅ عند إنشاء release جديد
- ✅ في GitHub Actions workflow
- ✅ قبل دمج المهارة في الكتالوج المركزي

---

## 5️⃣ update_version.py - تحديث رقم الإصدار

### الوصف
يحدث رقم الإصدار في `manifest.json` وفقاً لإصدار الدلالي (Semver).

### أنواع التحديث
- `patch` → 1.0.0 → 1.0.1 (إصلاحات صغيرة)
- `minor` → 1.0.0 → 1.1.0 (ميزات جديدة غير مُعطِّلة)
- `major` → 1.0.0 → 2.0.0 (تغييرات مُعطِّلة)
- إصدار محدد → 1.2.3

### الاستخدام

```bash
# تحديث مهارة واحدة
python scripts/update_version.py <skill_id> <bump_type>
python scripts/update_version.py <skill_id> <exact_version>

# تحديث جميع المهارات
python scripts/update_version.py --all <bump_type>
```

### أمثلة عملية

#### مثال 1: تحديث patch
```bash
$ python scripts/update_version.py pdf_document patch
Updated pdf_document: 1.0.0 -> 1.0.1

Next steps:
  1. Review changes: git diff
  2. Commit: git commit -m 'chore(release): pdf_document v1.0.1'
  3. Tag: git tag -a pdf_document-v1.0.1 -m 'Release v1.0.1'
  4. Push: git push origin main && git push origin pdf_document-v1.0.1
```

#### مثال 2: تحديث minor
```bash
$ python scripts/update_version.py pdf_document minor
Updated pdf_document: 1.0.1 -> 1.1.0

Next steps:
  1. Review changes: git diff
  2. Commit: git commit -m 'chore(release): pdf_document v1.1.0'
  3. Tag: git tag -a pdf_document-v1.1.0 -m 'Release v1.1.0'
  4. Push: git push origin main && git push origin pdf_document-v1.1.0
```

#### مثال 3: تحديث major
```bash
$ python scripts/update_version.py pdf_document major
Updated pdf_document: 1.1.0 -> 2.0.0

Next steps:
  1. Review changes: git diff
  2. Commit: git commit -m 'chore(release): pdf_document v2.0.0'
  3. Tag: git tag -a pdf_document-v2.0.0 -m 'Release v2.0.0'
  4. Push: git push origin main && git push origin pdf_document-v2.0.0
```

#### مثال 4: إصدار محدد
```bash
$ python scripts/update_version.py pdf_document 1.2.3
Updated pdf_document: 1.1.0 -> 1.2.3

Next steps:
  1. Review changes: git diff
  2. Commit: git commit -m 'chore(release): pdf_document v1.2.3'
  3. Tag: git tag -a pdf_document-v1.2.3 -m 'Release v1.2.3'
  4. Push: git push origin main && git push origin pdf_document-v1.2.3
```

#### مثال 5: تحديث جميع المهارات
```bash
$ python scripts/update_version.py --all patch
Bumping 6 skill(s) with 'patch'...

[pdf_document] 1.0.0 -> 1.0.1
[docx_document] 1.0.0 -> 1.0.1
[xlsx_spreadsheet] 1.0.0 -> 1.0.1
[pptx_presentation] 1.0.0 -> 1.0.1
[csv_data] 1.0.0 -> 1.0.1
[markdown_doc] 1.0.0 -> 1.0.1
```

### متى تستخدم هذا السكربت؟
- ✅ قبل إنشاء tag جديد
- ✅ بعد إصلاح bug (patch)
- ✅ بعد إضافة ميزة جديدة (minor)
- ✅ عند تغيير مُعطِّل (major)

---

## 6️⃣ changelog.py - توليد سجل التغييرات

### الوصف
يولد CHANGELOG من git commits باستخدام Conventional Commits.

### تصنيف Commits
| النوع | القسم |
|-------|-------|
| `feat` | Features |
| `fix` | Bug Fixes |
| `docs` | Documentation |
| `perf` | Performance |
| `refactor` | Refactoring |
| `chore`, `test`, `ci`, `style` | _(يُتخطى)_ |

### الاستخدام

```bash
# توليد changelog بين tagين
python scripts/changelog.py <skill_id> <from_tag> <to_tag>

# معاينة التغييرات غير الملتزمة
python scripts/changelog.py --preview <skill_id>

# حفظ في ملف
python scripts/changelog.py <skill_id> <from_tag> <to_tag> --output CHANGELOG-section.md
```

### أمثلة عملية

#### مثال 1: توليد changelog بين إصدارين
```bash
$ python scripts/changelog.py pdf_document pdf_document-v1.0.0 pdf_document-v1.0.1
## 1.0.1

### Features
- Add support for Arabic text rendering
- Add page numbering option

### Bug Fixes
- Fix font size calculation for CJK characters

### Documentation
- Update installation instructions
```

#### مثال 2: معاينة التغييرات غير الملتزمة
```bash
$ python scripts/changelog.py --preview pdf_document
## Unreleased

### Features
- Add watermark support
- Add header/footer templates

### Performance
- Optimize PDF generation speed
```

#### مثال 3: حفظ في ملف
```bash
$ python scripts/changelog.py pdf_document pdf_document-v1.0.0 pdf_document-v1.0.1 --output CHANGELOG-section.md
Written to: CHANGELOG-section.md

$ cat CHANGELOG-section.md
## 1.0.1

### Features
- Add support for Arabic text rendering
- Add page numbering option

### Bug Fixes
- Fix font size calculation for CJK characters
```

### تنسيق Conventional Commits
```
feat(scope): description
fix(scope): description
docs: description
perf(scope): description
refactor: description
```

### أمثلة على commits
```bash
git commit -m "feat(pdf): add watermark support"
git commit -m "fix(pdf): correct Arabic text alignment"
git commit -m "docs: update README with examples"
git commit -m "perf: optimize image compression"
git commit -m "chore: update dependencies"  # يُتخطى
```

### متى تستخدم هذا السكربت؟
- ✅ قبل إنشاء release
- ✅ عند كتابة release notes
- ✅ لتوثيق التغييرات تلقائياً

---

## 🔄 سير العمل الكامل (Full Workflow)

### من التطوير إلى الإطلاق

```bash
# 1. إنشاء مهارة جديدة
mkdir -p skills/my_skill
# ... إنشاء manifest.json و SKILL.md ...

# 2. التحقق من الصحة
python scripts/validate_manifest.py my_skill

# 3. البناء والتحقق
python scripts/build_package.py my_skill --verify

# 4. اختبار محلي
# ... قم بتثبيت الحزمة في PELDRUN ...

# 5. Commit التغييرات
git add skills/my_skill/
git commit -m "feat: add my_skill"

# 6. تحديث الإصدار
python scripts/update_version.py my_skill patch

# 7. Review و Commit
git diff
git add -A
git commit -m "chore(release): my_skill v1.0.0"

# 8. إنشاء Tag
git tag -a my_skill-v1.0.0 -m "Release v1.0.0"

# 9. Push (يُشغّل GitHub Actions تلقائياً)
git push origin main
git push origin my_skill-v1.0.0

# GitHub Actions سيقوم بـ:
# - بناء الحزمة
# - التحقق منها
# - توليد catalog entry
# - إنشاء GitHub Release
```

---

## 📊 جدول القرارات السريع

### "ماذا أفعل عندما..."

| الموقف | السكربت | الأمر |
|--------|---------|-------|
| أضفت مهارة جديدة | `validate_manifest.py` | `python scripts/validate_manifest.py <id>` |
| عدلت manifest.json | `validate_manifest.py` | `python scripts/validate_manifest.py <id>` |
| أريد اختبار الحزمة | `build_package.py` | `python scripts/build_package.py <id> --verify` |
| أصلحت bug صغير | `update_version.py` | `python scripts/update_version.py <id> patch` |
| أضفت ميزة جديدة | `update_version.py` | `python scripts/update_version.py <id> minor` |
| تغيير مُعطِّل | `update_version.py` | `python scripts/update_version.py <id> major` |
| أريد توليد release | `changelog.py` | `python scripts/changelog.py <id> v1.0.0 v1.1.0` |
| أحتاج catalog entry | `generate_catalog_entry.py` | `python scripts/generate_catalog_entry.py --skill <id> --github-owner <owner> --github-repo <repo>` |

---

## 🎯 أمثلة سيناريوهات كاملة

### السيناريو 1: إضافة مهارة جديدة من الصفر

```bash
# إنشاء المهارة
mkdir -p skills/data_analysis
cat > skills/data_analysis/manifest.json << 'EOF'
{
  "schema_version": "1.0.0",
  "id": "data_analysis",
  "version": "1.0.0",
  "name": "Data Analysis",
  "description": "Analyze datasets and generate insights",
  "execution": {
    "type": "instructions",
    "instructions_file": "SKILL.md"
  }
}
EOF

# إنشاء SKILL.md
cat > skills/data_analysis/SKILL.md << 'EOF'
---
name: data_analysis
description: Analyze datasets and generate insights
---

# Data Analysis Skill

## Instructions
1. Load the dataset
2. Perform exploratory analysis
3. Generate visualizations
4. Summarize findings
EOF

# التحقق
python scripts/validate_manifest.py data_analysis
# Output: [data_analysis] OK

# البناء
python scripts/build_package.py data_analysis --verify
# Output: Built: data_analysis-1.0.0.zip (5,234 bytes, 2 files)
# Output: VERIFICATION PASSED

# Commit
git add skills/data_analysis/
git commit -m "feat: add data_analysis skill"

# Tag
git tag -a data_analysis-v1.0.0 -m "Release v1.0.0"
git push origin main
git push origin data_analysis-v1.0.0
```

### السيناريو 2: إصلاح bug في مهارة موجودة

```bash
# عدل المهارة
# ... قم بالتعديلات في skills/pdf_document/SKILL.md ...

# Commit
git add -A
git commit -m "fix(pdf): correct Arabic text alignment"

# تحديث الإصدار
python scripts/update_version.py pdf_document patch
# Output: Updated pdf_document: 1.0.1 -> 1.0.2

# Review
git diff

# Commit version bump
git add -A
git commit -m "chore(release): pdf_document v1.0.2"

# Tag
git tag -a pdf_document-v1.0.2 -m "Release v1.0.2"
git push origin main
git push origin pdf_document-v1.0.2
```

### السيناريو 3: إضافة ميزة جديدة

```bash
# أضف الميزة
# ... عدل skills/docx_document/manifest.json و SKILL.md ...

# التحقق
python scripts/validate_manifest.py docx_document
# Output: [docx_document] OK

# Commit
git add -A
git commit -m "feat(docx): add table of contents support"

# تحديث الإصدار
python scripts/update_version.py docx_document minor
# Output: Updated docx_document: 1.0.2 -> 1.1.0

# Changelog
python scripts/changelog.py docx_document docx_document-v1.0.2 HEAD
# Output:
# ## 1.1.0
# ### Features
# - Add table of contents support

# Commit
git add -A
git commit -m "chore(release): docx_document v1.1.0"

# Tag
git tag -a docx_document-v1.1.0 -m "Release v1.1.0"
git push origin main
git push origin docx_document-v1.1.0
```

### السيناريو 4: تغيير مُعطِّل (Breaking Change)

```bash
# قم بالتغيير المُعطِّل
# ... عدل skills/code_review/manifest.json ...

# التحقق
python scripts/validate_manifest.py code_review

# Commit
git add -A
git commit -m "feat!: redesign code review output format"

# تحديث الإصدار
python scripts/update_version.py code_review major
# Output: Updated code_review: 1.3.2 -> 2.0.0

# Commit
git add -A
git commit -m "chore(release): code_review v2.0.0"

# Tag
git tag -a code_review-v2.0.0 -m "Release v2.0.0"
git push origin main
git push origin code_review-v2.0.0
```

### السيناريو 5: تحديث جميع المهارات

```bash
# تحديث جميع المهارات
python scripts/update_version.py --all patch
# Output:
# Bumping 6 skill(s) with 'patch'...
# [pdf_document] 1.0.1 -> 1.0.2
# [docx_document] 1.1.0 -> 1.1.1
# [xlsx_spreadsheet] 1.0.0 -> 1.0.1
# [pptx_presentation] 1.0.0 -> 1.0.1
# [csv_data] 1.0.0 -> 1.0.1
# [markdown_doc] 1.0.0 -> 1.0.1

# بناء جميع الحزم
python scripts/build_package.py --all --verify

# Commit
git add -A
git commit -m "chore: bump all skills to patch version"

# Tags (لكل مهارة)
git tag -a pdf_document-v1.0.2 -m "Release v1.0.2"
git tag -a docx_document-v1.1.1 -m "Release v1.1.1"
# ... إلخ

git push origin main
git push origin --tags
```

---

## 🔍 استكشاف الأخطاء وإصلاحها

### خطأ: "manifest.json is not valid JSON"
```bash
# الحل: تحقق من صيغة JSON
python -c "import json; json.load(open('skills/my_skill/manifest.json'))"

# استخدم linter
# https://jsonlint.com/
```

### خطأ: "Schema validation failed"
```bash
# تحقق من الرسالة المحددة
python scripts/validate_manifest.py my_skill

# أصلح الحقل المذكور في الخطأ
# مثال: id must match '^[a-z0-9][a-z0-9_]{2,63}$'
# الحل: استخدم حروف صغيرة وأرقام و underscores فقط
```

### خطأ: "Instructions file not found"
```bash
# تحقق من وجود SKILL.md
ls skills/my_skill/SKILL.md

# أو تحقق من execution.instructions_file في manifest
cat skills/my_skill/manifest.json | grep instructions_file
```

### خطأ: "File exceeds 5 MB limit"
```bash
# ابحث عن الملفات الكبيرة
find skills/my_skill -type f -size +5M

# احذف أو ضغط الملفات الكبيرة
```

### خطأ: "Checksum mismatch"
```bash
# أعد توليد catalog entry
python scripts/generate_catalog_entry.py \
    --skill my_skill \
    --zip dist/my_skill-1.0.0.zip \
    --download-url "..." \
    --repo "..."
```

---

## 📝 ملخص الأوامر الشائعة

```bash
# التحقق من جميع المهارات
python scripts/validate_manifest.py --all

# بناء جميع الحزم مع التحقق
python scripts/build_package.py --all --verify

# تحديث إصدار patch لجميع المهارات
python scripts/update_version.py --all patch

# توليد catalog entry
python scripts/generate_catalog_entry.py \
    --skill <id> \
    --github-owner <owner> \
    --github-repo <repo>

# معاينة changelog
python scripts/changelog.py --preview <id>

# التحقق من حزمة
python scripts/verify_package.py dist/<id>-<version>.zip
```

---

## ✅ Checklist قبل الإطلاق

```markdown
## قبل إنشاء Tag

- [ ] `validate_manifest.py` يمر بنجاح
- [ ] `build_package.py --verify` يمر بنجاح
- [ ] تم اختبار الحزمة محلياً
- [ ] CHANGELOG محدث (اختياري)
- [ ] `update_version.py` تم تشغيله
- [ ] جميع التغييرات ملتزمة (committed)

## إنشاء Tag

- [ ] Tag يتبع التنسيق: `<skill_id>-v<version>`
- [ ] Tag version يطابق manifest version
- [ ] تم push Tag إلى remote

## بعد الإطلاق

- [ ] GitHub Release تم إنشاؤه
- [ ] ZIP و catalog entry مرفقات
- [ ] تم تحديث الكتالوج المركزي (إذا لزم)
```

---

## 🎓 نصائح متقدمة

### 1. استخدام Git Hooks للتحقق التلقائي
```bash
# .git/hooks/pre-commit
#!/bin/bash
python scripts/validate_manifest.py --all
if [ $? -ne 0 ]; then
    echo "Validation failed. Commit rejected."
    exit 1
fi
```

### 2. دمج مع CI/CD
```yaml
# .github/workflows/ci.yml
- name: Validate all skills
  run: python scripts/validate_manifest.py --all

- name: Build all packages
  run: python scripts/build_package.py --all --verify
```

### 3. توليد catalog entries تلقائياً
```bash
# في GitHub Release workflow
- name: Generate catalog entry
  run: |
    python scripts/generate_catalog_entry.py \
        --skill ${{ steps.parse.outputs.skill_id }} \
        --github-owner ${{ github.repository_owner }} \
        --github-repo ${{ github.event.repository.name }}
```

### 4. Batch operations
```bash
# تحديث جميع المهارات وبناءها
for skill in skills/*/; do
    skill_id=$(basename "$skill")
    python scripts/update_version.py "$skill_id" patch
    python scripts/build_package.py "$skill_id" --verify
done
```

---

هذا الدليل يغطي جميع جوانب استخدام سكربتات الأتمتة. للمزيد من التفاصيل، راجع ملف `README.md` في كل سكربت أو الوثائق الرسمية في https://github.com/peldrun/peldrun/tree/main/docs/skills
