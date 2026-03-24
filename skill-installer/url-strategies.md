# URL Detection & Download Strategies

Reference file for identifying skill sources and downloading them. Read this file when a user provides a URL, file path, or installation command.

## URL Classification Rules

Test these patterns **in order**. Use the first match.

### 1. Local File Path → `local-file`

**Match:** Input starts with `/`, `~/`, `./`, `../`, or contains no `://` and looks like a filesystem path.

**Sub-types:**
- **Directory with SKILL.md**: Path is a directory → check for `SKILL.md` or `skill.md` inside
- **Single SKILL.md file**: Path ends with `SKILL.md` or `skill.md`
- **ZIP file**: Path ends with `.zip`

**Download strategy:**
```bash
# Directory
ls "{path}/SKILL.md" 2>/dev/null || ls "{path}/skill.md" 2>/dev/null
# If found → copy entire directory to temp
cp -R "{path}" /tmp/skill-install-TIMESTAMP/

# Single file
cp "{path}" /tmp/skill-install-TIMESTAMP/SKILL.md

# ZIP
unzip -o "{path}" -d /tmp/skill-install-TIMESTAMP/
# Then scan recursively for SKILL.md
```

### 2. GitHub Repository → `github-repo`

**Match:** URL contains `github.com/{owner}/{repo}` (NOT `gist.github.com`)

**Extract from URL:**
- `owner`: path segment after github.com
- `repo`: next path segment (strip `.git` suffix if present)
- `branch`: segment after `/tree/` (default: `main`)
- `path`: remaining segments after branch (optional, may point to subdirectory)

**Example URLs:**
```
github.com/org/repo
github.com/org/repo/tree/main
github.com/org/repo/tree/main/skills/my-skill
github.com/org/repo.git
```

**Download strategy (Option A - GitHub API, preferred):**
```bash
# List directory contents
curl -s "https://api.github.com/repos/{owner}/{repo}/contents/{path}?ref={branch}" \
  -H "Accept: application/vnd.github.v3+json"
```

Response is a JSON array. For each item:
- `type: "file"` → download via `download_url` field
- `type: "dir"` → recurse into subdirectory

Save all files preserving directory structure under `/tmp/skill-install-TIMESTAMP/`.

**If API returns 403 (rate limit) or 404 → Fallback to Option B:**
```bash
git clone --depth 1 -b {branch} "https://github.com/{owner}/{repo}.git" /tmp/skill-install-TIMESTAMP/repo
# If path specified, skill is in the subdirectory
```

**If branch `main` returns 404, try `master`.**

### 3. GitHub Gist → `github-gist`

**Match:** URL contains `gist.github.com`

**Extract:** `gist_id` = last path segment (the hex string)

**Example URLs:**
```
gist.github.com/username/abc123def456
gist.github.com/abc123def456
```

**Download strategy:**
```bash
curl -s "https://api.github.com/gists/{gist_id}"
```

Response JSON has a `files` object. Each key is a filename, value has:
- `filename`: the file name
- `content`: full file content (if not truncated)
- `raw_url`: URL to download raw content
- `truncated`: boolean

For each file:
1. If `truncated: false` → use `content` directly
2. If `truncated: true` → download from `raw_url`

Save all files to `/tmp/skill-install-TIMESTAMP/`.

### 4. Playbooks.com → `playbooks`

**Match:** URL contains `playbooks.com`

**Download strategy:** Browser-based extraction (Claude in Chrome).

```
1. mcp__Claude_in_Chrome__tabs_context_mcp (createIfEmpty: true)
2. mcp__Claude_in_Chrome__tabs_create_mcp
3. mcp__Claude_in_Chrome__navigate (url)
4. mcp__Claude_in_Chrome__computer (action: wait, duration: 3)
5. mcp__Claude_in_Chrome__get_page_text (tabId)
```

In the extracted text, look for:
- YAML frontmatter: text between `---` markers at the start
- Code blocks containing SKILL.md content
- Multiple files shown in tabs (SKILL.md, SETUP.md, etc.)

If a "Copy" or "Download" or "Raw" button exists:
```
6. mcp__Claude_in_Chrome__find (query: "copy button" or "download button")
7. mcp__Claude_in_Chrome__computer (action: left_click, coordinate: ...)
```

Save extracted content to `/tmp/skill-install-TIMESTAMP/SKILL.md`.

After extraction, close the tab:
```
mcp__Claude_in_Chrome__tabs_close_mcp (tabId)
```

### 5. Raw SKILL.md URL → `raw-url`

**Match:** URL ends with `SKILL.md`, `skill.md`, or URL contains `raw.githubusercontent.com`

**Download strategy:**
```bash
curl -sL "{url}" -o /tmp/skill-install-TIMESTAMP/SKILL.md
```

Verify the downloaded file starts with `---` (YAML frontmatter).

### 6. npm/npx/pip/brew Command → `command`

**Match:** Input starts with `npx `, `npm install`, `pip install`, `brew install`, or similar package manager commands.

**Strategy:**
- This is NOT a direct skill download. The user is providing an installation command.
- Execute the command and look for the resulting skill files.
- Common patterns:
  - `npx` commands may scaffold a skill directory
  - `npm install -g` may install a CLI that includes skills
- After execution, scan for SKILL.md in common locations:
  - Current directory
  - `node_modules/` (for npm installs)
  - Output of the command may indicate where files were placed

### 7. Unknown URL → `unknown`

**Match:** Anything not matching the above patterns.

**Download strategy:** Browser-based extraction (same as Playbooks).

```
1. Navigate to URL in Chrome
2. Wait for page load
3. get_page_text to extract content
4. Search for YAML frontmatter pattern (--- ... ---)
5. Search for code blocks containing skill content
6. If found → extract and save
7. If not found → read_page for interactive elements
8. Try clicking "download", "raw", "copy" buttons if found
9. If still not found → report to user that no skill content was detected
```

## Post-Download: Plugin vs Skill Detection

**IMPORTANT:** After downloading files, BEFORE looking for SKILL.md files, check if this is a **Claude Code plugin** (not just skills).

### How to Detect a Plugin

```bash
# Check for plugin manifest
ls /tmp/skill-install-TIMESTAMP/.claude-plugin/plugin.json 2>/dev/null
# Also check one level deep (if repo was cloned into a subdirectory)
find /tmp/skill-install-TIMESTAMP/ -maxdepth 2 -path "*/.claude-plugin/plugin.json" 2>/dev/null
```

**If `.claude-plugin/plugin.json` exists → this is a PLUGIN, not a standalone skill.**

### Plugin Installation Flow

A plugin is a complete package with skills + commands + metadata. It must be installed as a plugin, not as individual skills.

**Step 1: Identify plugin type**

Check if `.claude-plugin/marketplace.json` also exists:
- **If marketplace.json exists** → This is a **marketplace** (collection of plugins). Install the marketplace first, then install plugins from it.
- **If only plugin.json exists** → This is a **single plugin**. Install it directly.

**Step 2a: Marketplace installation**

```
הזיהוי: זה מרקטפלייס (אוסף פלאגינים) ולא סקיל בודד.
כדי להתקין, צריך להריץ את הפקודה הזו ב-Claude Code:

/plugin marketplace add {owner}/{repo}

אחרי שהמרקטפלייס נוסף, אפשר להתקין ממנו פלאגינים:
/plugin install {plugin-name}@{marketplace-name}
```

Read `marketplace.json` to list available plugins:
```
המרקטפלייס מכיל את הפלאגינים הבאים:
1. plugin-name-1 - description
2. plugin-name-2 - description

איזה פלאגינים להתקין?
```

For each selected plugin, run:
```bash
# Claude Code CLI command (run in the user's terminal)
/plugin install {plugin-name}@{marketplace-name}
```

**Step 2b: Single plugin installation**

Read `plugin.json` to get the plugin name and source repo URL.

```
הזיהוי: זה פלאגין של Claude Code (ולא סקיל בודד).
פלאגין יכול לכלול כמה סקילים, פקודות, ותוספים.

שם: {name}
תיאור: {description}
גרסה: {version}

כדי להתקין, צריך להריץ את הפקודה הזו ב-Claude Code:

/install-plugin {github-url}

או אם הפלאגין שייך למרקטפלייס:
/plugin install {name}@{marketplace}
```

**Step 3: Verify installation**

After running the install command, check:
```bash
cat ~/.claude/plugins/installed_plugins.json
```

Look for the plugin name in the installed list.

**Step 4: Enable the plugin**

Check if the plugin is enabled in settings:
```bash
cat ~/.claude/settings.json | python3 -c "import sys,json; d=json.load(sys.stdin); print(json.dumps(d.get('enabledPlugins',{}), indent=2))"
```

If not enabled, explain to user:
```
הפלאגין הותקן! כדי להפעיל אותו, הריצו:
/plugin enable {name}
```

### Key Differences: Plugin vs Skill

| | סקיל | פלאגין |
|---|---|---|
| **מיקום** | `~/.claude/skills/` | `~/.claude/plugins/cache/` |
| **התקנה** | העתקת קבצים | `/plugin install` |
| **עדכונים** | ידני | אוטומטי מ-git |
| **מבנה** | SKILL.md + קבצי עזר | .claude-plugin/ + skills/ + commands/ |
| **זיהוי** | יש SKILL.md | יש .claude-plugin/plugin.json |

## Multi-Skill Repository Detection

**Only reach this section if the source is NOT a plugin (no `.claude-plugin/plugin.json`).**

After downloading, scan for multiple skills:

```bash
# Find all SKILL.md files (case-insensitive)
find /tmp/skill-install-TIMESTAMP/ -maxdepth 3 -iname "SKILL.md" -o -iname "skill.md"
```

If **one** SKILL.md found → single skill, proceed normally.

If **multiple** SKILL.md files found:
1. Parse the `name` and `description` from each frontmatter
2. Present a numbered list to the user in Hebrew:
   ```
   מצאתי כמה סקילים במקור הזה:
   1. skill-name-1 - תיאור קצר
   2. skill-name-2 - תיאור קצר
   3. skill-name-3 - תיאור קצר

   איזה סקילים אתה רוצה להתקין? (מספרים מופרדים בפסיקים, או "הכל")
   ```
3. Install selected skills one at a time, running the full flow for each.

If **no** SKILL.md found:
- Check if there's a README.md with installation instructions
- Report to user: "לא מצאתי קובץ סקיל (SKILL.md) בכתובת הזו. אולי זה לא סקיל?"

## Temp Directory Management

Always create a unique temp directory at the start:
```bash
TEMP_DIR="/tmp/skill-install-$(date +%s)"
mkdir -p "$TEMP_DIR"
```

Clean up at the end (success or failure):
```bash
rm -rf "$TEMP_DIR"
```
