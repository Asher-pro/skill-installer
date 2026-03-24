# skill-installer — מתקין סקילים ל-Claude Code

סקיל חכם שמתקין סקילים חדשים מכל מקור — GitHub, Gist, Playbooks, קובץ מקומי, ZIP, או כל URL אחר.

מיועד למשתמשים לא-טכניים. מלווה אותך צעד אחרי צעד בעברית.

## מה הסקיל עושה?

1. **מזהה את המקור** — GitHub repo, Gist, Playbooks.com, קובץ מקומי, ZIP, או כל URL
2. **מוריד את הקבצים** — GitHub API, git clone, דפדפן, או העתקה מקומית
3. **סורק אבטחה** — מחפש דפוסים מסוכנים ומציג הערכת סיכונים (ירוק/צהוב/אדום)
4. **מתקין** — מעתיק ל-`~/.claude/skills/` ומוודא שהכל תקין
5. **מגדיר API keys** — מזהה מפתחות נדרשים ומדריך בהגדרה

## התקנה מהירה

### אפשרות 1: העתקה ישירה

```bash
cp -R skill-installer ~/.claude/skills/
```

### אפשרות 2: Clone מ-GitHub

```bash
git clone https://github.com/aviz85/skill-installer.git
cp -R skill-installer/skill-installer ~/.claude/skills/
```

### אפשרות 3: הורדת ZIP

הורד את `skill-installer.zip` מעמוד ה-Releases, ואז:

```bash
unzip skill-installer.zip -d ~/.claude/skills/
```

## שימוש

פשוט אמור לקלוד:

- "תתקין את הסקיל הזה: https://github.com/..."
- "תתקין סקיל מהקובץ ~/Downloads/SKILL.md"
- "install skill https://gist.github.com/..."
- "הוסף סקיל מ-playbooks.com/..."

## מקורות נתמכים

| מקור | דוגמה |
|------|-------|
| GitHub Repository | `github.com/org/repo/tree/main/skill-name` |
| GitHub Gist | `gist.github.com/user/abc123` |
| Playbooks.com | `playbooks.com/skills/user/collection/skill` |
| Raw URL | כל URL שמוביל ל-SKILL.md |
| קובץ מקומי | `~/Downloads/SKILL.md` |
| תיקייה מקומית | `~/Downloads/my-skill/` |
| קובץ ZIP | `~/Downloads/skill.zip` |
| פקודה | `npx some-skill-installer` |
| URL לא מוכר | כל URL — הסקיל ינסה למצוא SKILL.md דרך דפדפן |

## אבטחה

הסקיל סורק כל קובץ לפני התקנה ומחפש:
- פקודות shell מסוכנות
- גישה לקבצי credentials
- Reverse shells ו-data exfiltration
- Prompt injection
- קוד מעורפל

התוצאה מוצגת כהערכת סיכונים בעברית:
- **ירוק** — בטוח, רק קבצי הוראות
- **צהוב** — יש קוד, נראה לגיטימי, כדאי לבדוק
- **אדום** — זוהו דפוסים מחשידים, לא מומלץ להתקין

## מבנה הקבצים

```
skill-installer/
├── SKILL.md                  # הלוגיקה המרכזית + תקשורת בעברית
├── security-patterns.md      # רפרנס: דפוסים מסוכנים לסריקה
├── url-strategies.md         # רפרנס: זיהוי URL + אסטרטגיות הורדה
└── installation-checklist.md # רפרנס: צ'קליסט התקנה
```

## רישיון

MIT
