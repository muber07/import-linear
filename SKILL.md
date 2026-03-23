---
name: import-linear
description: Import projects from an Excel or CSV file into Linear using a JSON config. Use when the user says "import to linear", "run linear import", "sync excel to linear", or "import projects". Supports dry-run preview before committing changes.
allowed-tools:
  - Bash
  - Read
  - Write
  - AskUserQuestion
---

# import-linear Skill

Imports projects (and optionally issues) from an Excel/CSV file into Linear using the [linear-solutions projects_import](https://github.com/linear/linear-solutions) script and a JSON config file.

---

## Step 0: Parse Arguments

The skill accepts optional arguments in any order:
- `--config <path>` — path to JSON config file
- `--csv <path>` or `--xlsx <path>` — path to Excel/CSV file
- `--dry-run` — preview without making changes
- `--batch <N>` — test with first N items only
- `--api-key <key>` — Linear API key (overrides env var)

If no args provided, use defaults (see Step 1).

---

## Step 1: Resolve Inputs

**Script location:**
```bash
SCRIPT_DIR=~/linear-solutions-main/scripts/projects_import
```

**Config file** (in order of precedence):
1. `--config` argument if provided
2. Ask the user which config to use if multiple exist:
```bash
ls ~/finprod_critical_path*.json ~/.claude/skills/import-linear/configs/*.json 2>/dev/null
```
3. Default: `~/finprod_critical_path_v2.json`

**Excel/CSV file** (in order of precedence):
1. `--csv` or `--xlsx` argument if provided
2. Ask the user which file to use, showing recently modified xlsx files in Downloads:
```bash
ls -lt ~/Downloads/*.xlsx 2>/dev/null | head -5
```
3. Default: most recently modified `.xlsx` in `~/Downloads/`

**API key** (in order of precedence):
1. `--api-key` argument if provided
2. `$LINEAR_API_KEY` environment variable
3. Ask the user: "Please provide your Linear API key (Settings → API → Personal API keys)"

---

## Step 2: Validate

Check dependencies are installed:
```bash
cd "$SCRIPT_DIR"
python3 -c "import openpyxl" 2>/dev/null || pip3 install openpyxl --break-system-packages -q
```

Confirm the config and file exist:
```bash
test -f "$CONFIG" && echo "✅ Config: $CONFIG" || echo "❌ Config not found: $CONFIG"
test -f "$XLSX" && echo "✅ File: $XLSX" || echo "❌ File not found: $XLSX"
```

---

## Step 3: Dry Run (always first)

Always run a dry run first and show the summary to the user:

```bash
cd "$SCRIPT_DIR" && python3 import_linear.py \
  --api-key "$API_KEY" \
  --config "$CONFIG" \
  --csv "$XLSX" \
  --dry-run -y 2>&1
```

Parse and show the user:
- How many projects will be **created** (new)
- How many projects will be **updated** (existing)
- How many projects will be **skipped**
- Any ⚠️ warnings (unmatched leads, missing columns, etc.)
- New labels/groups that will be created

Then ask: **"Looks good — run the full import? [y/N]"**

If `--dry-run` was passed as an argument, stop here and do not ask to proceed.

---

## Step 4: Full Import

If the user confirms, run the real import:

```bash
cd "$SCRIPT_DIR" && python3 import_linear.py \
  --api-key "$API_KEY" \
  --config "$CONFIG" \
  --csv "$XLSX" \
  -y 2>&1
```

---

## Step 5: Report Results

Show a clean summary:

```
✅ Import complete!

📦 Projects:  X created, Y updated, Z skipped
🏷️  Labels:    A groups, B labels created
❌ Failures:  0

⚠️  Unmatched leads (no Linear user found):
   - Name1, Name2 (these fields were left blank)
```

If there were failures, show the error messages and suggest fixes.

---

## Notes

- **No duplicates:** The script is idempotent — existing projects (matched by name) are never duplicated. Re-running updates labels, milestones, content, and links.
- **Unmatched leads:** Partial names like "John D" won't match Linear users. Full emails work best.
- **Config docs:** See the [linear-solutions README](https://github.com/linear/linear-solutions/tree/main/scripts/projects_import) for full config reference.
- **Multiple sheets:** Set `"xlsx_sheet": "Sheet Name"` in the config to target a specific worksheet.
