# import-linear — Claude Code Skill

A [Claude Code](https://claude.ai/claude-code) skill that imports projects from an Excel or CSV file into [Linear](https://linear.app) using a config-driven approach. Built on top of the [linear-solutions](https://github.com/linear/linear-solutions) `projects_import` script.

## What it does

- Reads an Excel (`.xlsx`) or CSV file
- Maps columns to Linear fields via a JSON config
- Creates new projects and updates existing ones (idempotent — no duplicates)
- Adds milestones, labels, health status, progress updates, and description metadata
- Always runs a dry-run preview before committing changes

## Usage

Once installed, just type in Claude Code:

```
/import-linear
```

Or with arguments:

```
/import-linear --config ~/my_config.json --csv ~/data/projects.xlsx
/import-linear --dry-run
/import-linear --batch 5   # test with first 5 rows
```

## Installation

### 1. Install the linear-solutions import script

```bash
# Download and unzip
curl -L https://github.com/linear/linear-solutions/archive/refs/heads/main.zip -o ~/linear-solutions-main.zip
unzip ~/linear-solutions-main.zip -d ~/

# Install Python dependency for Excel support
pip3 install openpyxl
```

### 2. Install this skill

```bash
# Clone into your Claude Code skills directory
git clone https://github.com/muber07/import-linear ~/.claude/skills/import-linear
```

Claude Code auto-discovers skills in `~/.claude/skills/` — no restart needed.

### 3. Set your Linear API key

Get your key from Linear → **Settings → API → Personal API keys**, then either:

```bash
# Option A: env var (add to ~/.zshrc or ~/.bashrc)
export LINEAR_API_KEY=lin_api_your_key_here

# Option B: pass it each time
/import-linear --api-key lin_api_your_key_here
```

## Config file

The import is driven by a JSON config that maps your spreadsheet columns to Linear fields.

### Minimal example

```json
{
  "name": "My Team Import",
  "import_mode": "standard",
  "xlsx_sheet": "Sheet1",

  "team": {
    "target_key": "MYTEAM"
  },

  "projects": {
    "source": "column:Project Name",
    "columns": {
      "name": "Project Name",
      "lead": "Owner",
      "health": "Status",
      "target_date": "Target Date"
    },
    "health_keywords": [
      { "keyword": "On track", "health": "onTrack" },
      { "keyword": "At Risk", "health": "atRisk" },
      { "keyword": "Blocked", "health": "offTrack" }
    ]
  },

  "issues": { "enabled": false }
}
```

See [`configs/example_config.json`](configs/example_config.json) for a full example with milestones, label groups, and description extras.

For the complete config reference, see the [linear-solutions README](https://github.com/linear/linear-solutions/tree/main/scripts/projects_import).

## How deduplication works

Projects are matched by **name**. If a project with the same name already exists in Linear:
- It is **not** duplicated
- Its labels, milestones, content, and lead are **updated** with fresh data from the spreadsheet

This means you can re-run the import whenever your spreadsheet changes and it will safely sync.

## Example: FinProd Critical Path Tracker

This skill was originally built to sync the FinProd project tracker from Excel into Linear. The config used:

```json
{
  "name": "FinProd Critical Path Tracker",
  "import_mode": "standard",
  "xlsx_sheet": "Critical Path Dates",
  "team": { "target_key": "FIN0" },
  "projects": {
    "source": "column:uPlan Project Name / Title (linked)",
    "multi_date": true,
    "columns": {
      "name": "uPlan Project Name / Title (linked)",
      "lead": "Prod Owner",
      "health": "Status",
      "target_date": "🚀 Target Rollout",
      "update_text": "Progress Update - Feb '26 (Covering 01-Feb-26 to 28-Feb-26)"
    },
    "health_keywords": [
      { "keyword": "On track", "health": "onTrack" },
      { "keyword": "At Risk", "health": "atRisk" },
      { "keyword": "Delayed", "health": "offTrack" },
      { "keyword": "Done", "health": "onTrack" }
    ],
    "label_groups": [
      { "group_name": "Product", "column": "Product" },
      { "group_name": "Bet", "column": "Bet" }
    ],
    "milestone_columns": [
      { "column": "📋 BRD Approved", "name": "📋 BRD Approved" },
      { "column": "🔍 Partner Selected", "name": "🔍 Partner Selected" },
      { "column": "📍CP3", "name": "📍CP3" },
      { "column": "🛠️ Target Dev Complete", "name": "🛠️ Target Dev Complete" }
    ]
  },
  "issues": { "enabled": false }
}
```

## Requirements

- [Claude Code](https://claude.ai/claude-code)
- Python 3.7+
- `openpyxl` (`pip3 install openpyxl`) for Excel support
- [linear-solutions](https://github.com/linear/linear-solutions) cloned to `~/linear-solutions-main/`
- Linear API key (Personal API key from Linear settings)
