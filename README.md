# skill-installer — מתקין סקילים ופלאגינים ל-Claude Code

סקיל חכם שמתקין סקילים ופלאגינים חדשים מכל מקור — GitHub, Gist, Playbooks, מרקטפלייס, קובץ מקומי, ZIP, או כל URL אחר.

מיועד למשתמשים לא-טכניים. מלווה אותך צעד אחרי צעד בעברית.

## מה הסקיל עושה?

1. **מזהה את המקור** — GitHub repo, Gist, Playbooks.com, קובץ מקומי, ZIP, או כל URL
2. **מזהה את הסוג** — סקיל בודד, ריפו עם כמה סקילים, פלאגין, או מרקטפלייס
3. **מוריד את הקבצים** — GitHub API, git clone, דפדפן, או העתקה מקומית
4. **סורק אבטחה** — מחפש דפוסים מסוכנים ומציג הערכת סיכונים (ירוק/צהוב/אדום)
5. **מתקין** — סקילים ל-`~/.claude/skills/`, פלאגינים דרך `/plugin install`
6. **מגדיר API keys** — מזהה מפתחות נדרשים ומדריך בהגדרה

---

## התקנה

### הורדת ZIP (מומלץ - הכי קל!)

> **[לחץ כאן להורדת skill-installer.zip](https://github.com/Asher-pro/skill-installer/raw/main/skill-installer.zip)**

אחרי ההורדה:

1. פתח את Claude Code
2. לך ל-**Customize** → **Skills** → לחץ על **+** → בחר **Upload a skill**
3. בחר את קובץ ה-ZIP שהורד
4. זהו! הסקיל מותקן

![איפה מעלים סקיל](images/upload-skill.png)

**לחלופין**, מהטרמינל:
```bash
unzip ~/Downloads/skill-installer.zip -d ~/.claude/skills/
```

### Clone מ-GitHub

```bash
git clone https://github.com/Asher-pro/skill-installer.git
cp -R skill-installer/skill-installer ~/.claude/skills/
```

---

## שימוש

אחרי ההתקנה, פשוט אמור לקלוד:

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
| פלאגין Claude Code | `github.com/org/plugin-repo` (מכיל `.claude-plugin/`) |
| מרקטפלייס | `github.com/org/marketplace` (אוסף פלאגינים) |
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
