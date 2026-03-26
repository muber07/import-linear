# import-linear

Import and re-import projects from Google Sheets into **Linear** and **Jira** — with one command in Claude Code.

---

## How it works

1. **Copy the template** → fill in your projects
2. **Tell Claude** to import it → done
3. **Re-run anytime** → it updates, never duplicates

---

## Step 1 — Get the Template

📄 **[Open Template in Google Sheets →](https://docs.google.com/spreadsheets/d/1ZQsuwuM-njtdwyzX7qcCHG2ky6WbtVnhJD6W8z1XVhw/edit)**

> File → Make a copy → rename it → start filling it in

### Columns

| Column | What to put |
|---|---|
| **Issue Type** | `Task`, `Epic`, or `Story` |
| **Assignee** | Their email (e.g. `name@company.com`) |
| **Sub Initiative** | Optional grouping label |
| **Story Points** | Number (e.g. `3`) |
| **Title** | Name of the project or issue |
| **Start Date** | `YYYY-MM-DD` (e.g. `2026-04-01`) |
| **Due Date** | `YYYY-MM-DD` |
| **Blocked By** | Issue title or key that blocks this one |
| **Blocks** | Issue title or key that this one blocks |
| **Team** | Team name in Linear / project key in Jira |
| **Parent Epic** | Epic title or key (for tasks/stories) |
| **Project** | Project name |

---

## Step 2 — Set Up (first time only)

### Install the MCPs in Claude Code

Open Claude Code and run:

```
/mcp
```

Add the following servers if not already connected:

| Tool | MCP Server URL |
|---|---|
| **Linear** | `https://mcp.linear.app/mcp` |
| **Jira (Atlassian)** | `https://mcp.atlassian.com/v1/mcp` |
| **Google Sheets** | `https://sheets.googleapis.com` *(via google-workspace skill)* |

> In Claude Code desktop: Settings → MCP Servers → Add → paste the URL.

### For Linear imports — install the script (one time)

```bash
curl -L https://github.com/linear/linear-solutions/archive/main.zip -o /tmp/linear.zip
unzip /tmp/linear.zip -d ~/
pip3 install openpyxl --break-system-packages -q
```

Set your Linear API key (Settings → API → Personal API keys):
```bash
export LINEAR_API_KEY=lin_api_xxxx
```

---

## Step 3 — Import into Linear

In Claude Code, just say:

```
/import-linear
```

It will:
- Find your most recent `.xlsx` in Downloads automatically
- Show you a **dry-run preview** (what will be created/updated)
- Ask for confirmation before making any changes
- Re-running it later will **update** existing projects, never create duplicates

**With options:**
```
/import-linear --dry-run              # preview only, no changes
/import-linear --batch 5             # test with first 5 rows
/import-linear --xlsx ~/my/file.xlsx # use a specific file
```

---

## Step 4 — Import into Jira

Make sure the **Atlassian MCP** is connected (see Step 2), then tell Claude:

```
Read my Google Sheet at https://docs.google.com/spreadsheets/d/YOUR_SHEET_ID/edit
and create Jira issues in project [YOUR_PROJECT_KEY] for each row:
- summary = Title
- assignee = Assignee (email)
- issuetype = Issue Type
- story_points = Story Points
- duedate = Due Date
- epic link = Parent Epic
After creating, link any Blocked By / Blocks relationships.
```

Replace `YOUR_SHEET_ID` with your sheet's ID and `YOUR_PROJECT_KEY` with your Jira project key (e.g. `FP`, `UUE`).

**To re-import / update** — run the same prompt again. Claude will check for existing issues and update them instead of creating duplicates.

---

## Re-importing

Both Linear and Jira imports are **idempotent** — you can run them as many times as you want:

- ✅ Existing items get **updated** (dates, assignees, status)
- ✅ New rows get **created**
- ✅ Nothing gets **duplicated**

Just update your Google Sheet and re-run.

---

## Milestone Status (Linear only)

In milestone date cells, you can add a status word after the date:

| Cell value | What happens in Linear |
|---|---|
| `2026-06-30` | Milestone with date |
| `2026-06-30, Done` | Milestone marked **completed** ✓ |
| `TBD` | Milestone named `CP1 (TBD)`, no date |
| `Cancelled` | Milestone named `CP1 (Cancelled)` |
| `Not needed` | Milestone named `CP1 (Not needed)` |

When you update the sheet (e.g. change `TBD` to a real date), re-running the import will rename the milestone automatically.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `LINEAR_API_KEY not set` | Run `export LINEAR_API_KEY=lin_api_xxxx` |
| Assignee left blank | Use full email — partial names won't match |
| Jira MCP not connected | `/mcp` → Add `https://mcp.atlassian.com/v1/mcp` |
| Import created duplicates | Check project names match exactly (case-sensitive) |
