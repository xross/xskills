---
name: fix-repo-checks
description: Use when test_repo_checks fails, when repo checks need fixing, or when user mentions "repo_checks", "repo checks", "infr_apps checks", "xmos checks". Automatically runs update scripts for fixable checks and AI-edits for unfixable checks in XMOS library repositories.
---

# fix-repo-checks

Automatically fixes failures from `test_repo_checks` in XMOS library repositories by running update scripts for auto-fixable checks and using AI to repair unfixable checks.

## When to Use

- User reports `test_repo_checks` failures
- User mentions "fix repo checks", "repo_checks", "infr_apps checks"
- Jenkins pipeline fails on "Repo checks" stage
- Working on XMOS library repositories (lib_*, fwk_*, etc.)

## Workflow

### 1. DISCOVERY PHASE

Run the test suite to identify all failing checks:

```bash
cd <repo_root>
pytest ../infr_apps/lib/python/test_repo_checks.py --tb=short --dir=.
```

Parse the output to identify which specific checks failed (e.g., `test_check_source`, `test_check_readme`, etc.).

### 2. ENVIRONMENT SETUP

**Check for infr_apps:**

1. Look for `../infr_apps` relative to the repository
2. If not found, clone it:
   ```bash
   cd <repo_parent_dir>
   git clone git@github.com:xmos/infr_apps.git
   ```

**Setup Python virtual environment:**

1. Look for existing venv named `.venv_<repo_name>` (e.g., `.venv_lib_sw_pll` for lib_sw_pll)
2. If found:
   ```bash
   source .venv_<repo_name>/bin/activate
   # Test if infr_apps is importable:
   python -c "import sys; sys.path.append('../infr_apps/lib/python'); import xmos_source_check"
   ```
3. If not found or import fails, create new venv:
   ```bash
   python3 -m venv .venv_<repo_name>
   source .venv_<repo_name>/bin/activate
   pip install ../infr_apps
   ```

**Set PYTHONPATH:**
```bash
export PYTHONPATH=../infr_apps/lib/python:$PYTHONPATH
```

### 3. READ LICENSE TYPE

Before fixing source checks, determine the license type:

```bash
# Extract license type from settings.yml
grep "^license:" settings.yml
```

**If license is `xmos_public_v1`:**

Scan the repository for potential issues that might make the public license inappropriate:

- Third-party code in the repository (check for external dependencies, vendored code)
- Sensitive IP indicators (proprietary algorithms, confidential comments, internal URLs)
- Files that don't look like standard XMOS open-source code

If concerns are found, **ASK THE USER**:
> "This repository uses `xmos_public_v1` (public open-source license), but I found [specific concerns]. Is this license appropriate for this code, or should it use a different license like `xmos_old` (proprietary)?"

Wait for user confirmation before proceeding.

Store the license type for use in source check updates.

### 4. FIX EACH FAILING CHECK

Process each failing check according to its type:

---

#### AUTO-FIXABLE CHECKS (Run Update Scripts)

These checks have built-in `update` modes that automatically fix issues:

##### `test_check_source` - Copyright Headers

**What it checks:** Source files (.c, .h, .py, .xc, etc.) have correct copyright headers with accurate dates and license references.

**How to fix:**
```bash
cd ../infr_apps/lib/python
python xmos_source_check.py update <repo_path> <license_type>
# Example: python xmos_source_check.py update ../../lib_sw_pll xmos_public_v1
```

**What it does:**
- Adds missing copyright headers
- Updates copyright date ranges based on git history
- Fixes license reference text to match the template

**Common errors:**
- `"File /path/to/file.c does not have current copyright at start of file"`
- `"Copyright date range incorrect"`

**Note:** Some files may be ignored via `.xmos_ignore_source_check` or `.xmos/.xmos_ignore_source_check`.

---

##### `test_check_license` - LICENSE.rst File

**What it checks:** `LICENSE.rst` or `LICENSE.txt` matches the expected template for the license type in `settings.yml`.

**How to fix:**
```bash
cd ../infr_apps/lib/python
python xmos_license_check.py update <repo_path>
# Example: python xmos_license_check.py update ../../lib_sw_pll
```

**What it does:**
- Reads license type from `settings.yml`
- Writes the correct LICENSE.rst template

**Common errors:**
- `"License file LICENSE.rst is incorrect"`
- `"Missing LICENSE.rst"`

---

##### `test_check_changelog` - CHANGELOG.rst Formatting

**What it checks:** `CHANGELOG.rst` has correct RST formatting, version entries, and dependency tracking.

**How to fix:**
```bash
cd ../infr_apps/lib/python
python xmos_changelog_check.py update <repo_path>
# Example: python xmos_changelog_check.py update ../../lib_sw_pll
```

**What it does:**
- Fixes RST formatting (bullet points, underlines)
- Adds missing version entries
- Updates dependency version tracking

**Common errors:**
- `"CHANGELOG.rst doesn't start with current version"`
- `"Line X does not match expected format"`
- `"Dependency version mismatch"`

---

#### MANUAL-FIX CHECKS (AI Edits Based on Error Messages)

These checks require manual editing based on error analysis:

##### `test_check_readme` - README.rst Structure

**What it checks:** README.rst structure, required sections, title format, RST syntax.

**How to fix:** Edit `README.rst` based on error messages.

**Common issues:**
1. **Title format:** Must be `lib_sw_pll: Descriptive Title`
   ```rst
   lib_sw_pll: Software PLL Library
   ===================================
   ```

2. **Required sections:**
   - Summary
   - Features
   - Known Issues
   - Support

3. **RST syntax errors:** Run `rstcheck README.rst` to identify issues

**Error patterns:**
- `"readme_check: Title doesn't match expected..."`
- `"Missing section: Features"`
- `"RST syntax error at line X"`

**Fix approach:**
- Read the error message carefully
- Edit README.rst to add missing sections or fix formatting
- Ensure proper RST syntax (headings, bullet points, code blocks)

---

##### `test_check_settings` - settings.yml Schema

**What it checks:** `settings.yml` schema validation, version format, cognidox numbers, doxygen config.

**How to fix:** Edit `settings.yml` based on error messages.

**Common issues:**
1. **Missing first-line comment:**
   ```yaml
   # Copyright 2024 XMOS LIMITED.
   ```

2. **Invalid version format:** Must be `X.Y.Z`
   ```yaml
   version: "1.0.0"
   ```

3. **Missing cognidox numbers:**
   ```yaml
   cognidox:
     user_guide: "1234-UG"    # User Guide
     sw_manual: "1234-SM"      # Software Manual
     app_note: "1234-AN"       # Application Note (if applicable)
   ```

4. **Doxygen project:**
   ```yaml
   doxygen:
     project: "lib_sw_pll"
   ```

**Error patterns:**
- `"settings.yml doesn't match basic schema"`
- `"Invalid version format"`
- `"Missing cognidox part numbers"`

**Fix approach:**
- Read the full `settings.yml` file
- Compare against the schema requirements in the error message
- Add missing fields or fix invalid values

---

##### `test_check_versions` - Version Consistency

**What it checks:** Version numbers are consistent across README.rst, settings.yml, CHANGELOG.rst, and lib_build_info.cmake.

**How to fix:** Synchronize versions across all files.

**Files to check:**
1. `README.rst` - First line or title should contain version
2. `settings.yml` - `version:` field
3. `CHANGELOG.rst` - First version entry
4. `lib_build_info.cmake` - `set(LIB_VERSION "X.Y.Z")`

**Error patterns:**
- `"Version mismatch: README.rst: 1.0.0, settings.yml: 1.0.1"`

**Fix approach:**
- Identify the correct version (typically from settings.yml)
- Update all other files to match
- This may require multiple iterations as fixing one file affects version check

---

##### `test_check_metadata` - README.rst Metadata

**What it checks:** Metadata block in README.rst is parseable and contains valid field values.

**How to fix:** Edit the metadata block in README.rst.

**Metadata format:**
```rst
:vendor: XMOS
:version: 1.0.0
:scope: General Use
:description: Software PLL library
:category: DSP
:keywords: PLL, clock recovery, synchronization
```

**Common issues:**
- Invalid field names
- Incorrect field values
- Malformed RST field list syntax

**Error patterns:**
- `"metadata_check: Failed to parse metadata"`
- `"Invalid metadata field: X"`

**Fix approach:**
- Locate the metadata block in README.rst (typically after the title)
- Fix syntax or field values based on error message
- Ensure proper RST field list format (`:field: value`)

---

##### `test_check_documentation` - doc/rst/ Structure

**What it checks:** Documentation folder structure, proper includes, README.rst not included in user guide.

**How to fix:** Create/fix doc/rst/ structure.

**Required structure:**
```
doc/
└── rst/
    ├── user_guide.rst      # Main user guide
    └── ...                 # Other documentation
```

**Common issues:**
1. Missing `doc/` or `doc/rst/` folders
2. README.rst incorrectly included in user guide

**Error patterns:**
- `"Missing doc/ folder"`
- `"User guide document includes README.rst"`

**Fix approach:**
- Create missing directories if needed
- Remove README.rst includes from user guide
- Ensure proper RST structure in documentation

---

##### `test_check_deps` - Dependency Versions

**What it checks:** Dependencies have version specifications in `module_build_info` and `lib_build_info.cmake`.

**How to fix:** Edit build files to add version specifications.

**module_build_info format:**
```
DEPENDENT_MODULES = lib_logging(>=2.0.0) lib_xassert(>=4.0.0)
```

**lib_build_info.cmake format:**
```cmake
set(LIB_NAME lib_sw_pll)
set(LIB_VERSION "1.0.0")
set(LIB_DEPENDENT_MODULES "lib_logging(>=2.0.0) lib_xassert(>=4.0.0)")
```

**Common issues:**
- Dependencies without version specifications
- Invalid version format

**Error patterns:**
- `"module_build_info dependency lib_logging does not specify version"`
- `"Invalid version format in lib_build_info.cmake"`

**Fix approach:**
- Identify dependencies without versions
- Determine appropriate version constraints (check dependent library versions)
- Add version specifications in proper format

---

##### `test_check_compiler_flags` - Debug Flags (APPLICATION_NOTE only)

**What it checks:** APP_COMPILER_FLAGS contains `-g` flag for debug symbols.

**How to fix:** Edit Makefile or CMakeLists.txt to add `-g` flag.

**Note:** This check only applies to APPLICATION_NOTE repositories, not LIBRARY repos.

**Error patterns:**
- `"Compiler flags check FAILED: Debug symbols flag '-g' is missing"`

**Fix approach:**
- Locate APP_COMPILER_FLAGS definition
- Add `-g` flag

---

### 5. VERIFY FIX

After fixing each check, re-run the specific test to verify:

```bash
pytest ../infr_apps/lib/python/test_repo_checks.py::test_check_<name> --tb=short --dir=.
# Example: pytest ../infr_apps/lib/python/test_repo_checks.py::test_check_source --tb=short --dir=.
```

If the check still fails, analyze the new error message and iterate.

### 6. FINAL VERIFICATION

After all fixes are applied, run all originally-failing checks again:

```bash
pytest ../infr_apps/lib/python/test_repo_checks.py --tb=short --dir=.
```

### 7. SUMMARY

Report to the user:

1. **Files modified** (all uncommitted for review):
   - List each file changed
   - Briefly describe what was fixed

2. **Checks fixed:**
   - List each test that now passes

3. **Remaining issues** (if any):
   - List checks that still fail
   - Explain what manual intervention might be needed

4. **Next steps:**
   - User should review changes with `git diff`
   - User should commit when satisfied
   - Suggest running full test suite if available

## Repository Type Skip Logic

Some checks are skipped based on repository type (from settings.yml):

- **APPLICATION_NOTE**: Skips `test_check_deps`, adds compiler flags check
- **DOCUMENTATION**: Skips `test_check_license`, `test_check_readme`, `test_check_metadata`
- **REFERENCE_DESIGN**: Skips `test_check_license`, `test_check_readme`, `test_check_settings`, `test_check_documentation`, `test_check_versions`
- **LIBRARY** (most repos): Runs all checks except `test_check_compiler_flags`

## Important Notes

1. **DO NOT commit changes** - Leave all modifications uncommitted for user review
2. **Iterative fixes** - Some checks (especially versions) may require multiple passes
3. **License verification** - Always verify `xmos_public_v1` is appropriate when that license is used
4. **Ignore files** - Source check respects `.xmos_ignore_source_check` patterns
5. **Virtual environment** - Always use `.venv_<repo_name>` convention for consistency

## Reference Paths

- **Test file:** `../infr_apps/lib/python/test_repo_checks.py`
- **Check scripts:** `../infr_apps/lib/python/xmos_*_check.py`
- **Repository root:** Current working directory when skill is invoked

## Troubleshooting

**If infr_apps clone fails:**
- Check SSH key is set up for GitHub
- Try HTTPS: `git clone https://github.com/xmos/infr_apps.git`

**If pytest can't find test_repo_checks.py:**
- Verify PYTHONPATH includes `../infr_apps/lib/python`
- Check infr_apps was cloned successfully

**If venv activation fails:**
- Ensure Python 3 is installed
- Try creating fresh venv: `rm -rf .venv_<repo_name> && python3 -m venv .venv_<repo_name>`

**If update scripts fail:**
- Check you're running from infr_apps/lib/python directory
- Verify repository path is correct (relative paths from infr_apps/lib/python)
- Check license type is valid (xmos_public_v1, xmos_old, vocalfusion3600, etc.)
