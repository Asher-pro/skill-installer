# Installation Checklist

Step-by-step verification checklist. Read this during the installation step to ensure nothing is missed.

## Pre-Installation Checks

- [ ] Downloaded files exist in temp directory
- [ ] SKILL.md (or skill.md) file found
- [ ] YAML frontmatter is valid (starts and ends with `---`)
- [ ] `name` field exists in frontmatter
- [ ] `description` field exists in frontmatter
- [ ] Skill name is valid for directory name (letters, numbers, hyphens only — no spaces or special characters)
- [ ] If name contains invalid characters, sanitize: replace spaces with hyphens, remove special chars, lowercase

## Security Checks

- [ ] Read `security-patterns.md` reference
- [ ] Scanned ALL files in the skill directory
- [ ] Classified risk level (GREEN / YELLOW / RED)
- [ ] Presented risk assessment to user in Hebrew
- [ ] User explicitly confirmed they want to install

## Conflict Check

Check if skill already exists:
```bash
ls ~/.claude/skills/{skill-name}/ 2>/dev/null
```

If exists:
```
סקיל בשם "{skill-name}" כבר מותקן. מה תרצה לעשות?
1. להחליף את הקיים (גיבוי אוטומטי)
2. לבטל את ההתקנה
```

If replacing, back up first:
```bash
mv ~/.claude/skills/{name} ~/.claude/skills/{name}.backup-$(date +%s)
```

## Installation Steps

```bash
# 1. Create target directory
mkdir -p ~/.claude/skills/{skill-name}/

# 2. Copy files (exclude junk)
# Copy SKILL.md / skill.md
cp /tmp/skill-install-XXX/SKILL.md ~/.claude/skills/{skill-name}/

# Copy SETUP.md if exists
cp /tmp/skill-install-XXX/SETUP.md ~/.claude/skills/{skill-name}/ 2>/dev/null

# Copy scripts/ directory if exists
cp -R /tmp/skill-install-XXX/scripts/ ~/.claude/skills/{skill-name}/scripts/ 2>/dev/null

# Copy reference/supporting .md files
cp /tmp/skill-install-XXX/*.md ~/.claude/skills/{skill-name}/ 2>/dev/null

# Copy other supporting directories (templates/, assets/, etc.)
# But SKIP: .git/, node_modules/, __pycache__/, .DS_Store, .env (actual secrets)
```

Files to NEVER copy:
- `.git/` directory
- `node_modules/`
- `__pycache__/`
- `.DS_Store`
- `.env` files with actual secrets (but DO copy `.env.example`)
- `evals/` directory (testing data, not needed for the skill)

```bash
# 3. If scripts/ has package.json, install dependencies
if [ -f ~/.claude/skills/{skill-name}/scripts/package.json ]; then
  cd ~/.claude/skills/{skill-name}/scripts/
  npm install
fi

# 4. If scripts/ has requirements.txt, install Python deps
if [ -f ~/.claude/skills/{skill-name}/scripts/requirements.txt ]; then
  pip install -r ~/.claude/skills/{skill-name}/scripts/requirements.txt
fi
```

## Post-Installation Verification

- [ ] SKILL.md exists at `~/.claude/skills/{skill-name}/SKILL.md`
- [ ] Frontmatter is parseable (read it back)
- [ ] All files from source are in place

Report to user:
```
הסקיל "{skill-name}" הותקן בהצלחה!

קבצים:
- SKILL.md (הוראות ראשיות)
- [list other files]
```

## API Key Configuration

### Step 1: Detect Required Environment Variables

Check frontmatter for:
```yaml
metadata:
  openclaw:
    requires:
      env:
        - VAR_NAME_1
        - VAR_NAME_2
```

Also scan the SKILL.md body for patterns like:
- `$VAR_NAME` or `${VAR_NAME}`
- `process.env.VAR_NAME`
- `os.environ['VAR_NAME']`
- References to setting env vars

### Step 2: Check Current Settings

Read `~/.claude/settings.json` and check the `env` object:

```bash
cat ~/.claude/settings.json
```

Parse JSON and check if each required var exists in the `env` section.

### Step 3: Guide User for Missing Keys

For each missing env var:

1. **Check SETUP.md** — if it exists, read it for instructions about this var
2. **Present instructions in Hebrew:**
   ```
   הסקיל הזה דורש מפתח API: {VAR_NAME}

   [Instructions from SETUP.md translated/presented in Hebrew]

   אנא הדבק את המפתח כאן:
   ```
3. **Wait for user input**

### Step 4: Save to Settings

After user provides the key:

1. Read `~/.claude/settings.json`
2. Parse as JSON
3. If `env` section doesn't exist, create it
4. Add the new key: `env.VAR_NAME = "user_provided_value"`
5. Write back with proper JSON formatting (2-space indent)
6. Verify by re-reading

**Important:** Do NOT overwrite existing env vars without asking. If a var already has a value:
```
המפתח {VAR_NAME} כבר מוגדר. להחליף אותו?
```

### Step 5: Check Required Binaries

If frontmatter has:
```yaml
metadata:
  openclaw:
    requires:
      bins:
        - binary_name
```

Check each:
```bash
which binary_name
```

If missing, tell the user:
```
הסקיל הזה צריך את הכלי "{binary_name}" מותקן על המחשב.

להתקין? (הפקודה: brew install {binary_name})
```

### Step 6: Run SETUP.md if Exists

If frontmatter has `setup: "./SETUP.md"` and `setup_complete` is not `true`:

1. Read SETUP.md
2. Present setup steps to user in Hebrew
3. Guide through each step
4. After completion, update SKILL.md frontmatter: `setup_complete: true`

## Dependencies (enhancedBy)

If frontmatter has `enhancedBy`:
```yaml
enhancedBy:
  - other-skill: "Description of what it adds"
```

After installation, inform user:
```
הסקיל הזה עובד טוב יותר עם הסקילים הבאים:
- other-skill: [description]

רוצה להתקין גם אותם?
```

If user says yes, start the installation flow for each dependency.

## Cleanup

```bash
# Remove temp directory
rm -rf /tmp/skill-install-TIMESTAMP/
```

## Final Success Message

```
הסקיל "{skill-name}" מותקן ומוכן לשימוש!

שם: {name}
תיאור: {description}
מיקום: ~/.claude/skills/{name}/

[If API keys were configured:]
מפתחות API הוגדרו: {list of vars}

[If dependencies noted:]
סקילים משלימים (לא חובה): {list}

כדי להשתמש בסקיל, פשוט דבר על הנושא שלו וקלוד יפעיל אותו אוטומטית.
```
