---
name: build-aab
description: Build AAB – העלאת גרסה ל-Google Play. Use when bumping version and building a release AAB for Android upload to Google Play Console.
license: MIT
version: 1.0.0
---

# Build AAB – העלאת גרסה ל-Google Play

תמיד ענה בעברית. ערכים טכניים (מספרי גרסה, נתיבים, פקודות) נשארים באנגלית.

## שלב 1 – קרא מצב נוכחי

קרא את הקבצים הבאים במקביל:
- `app.json` – שדה `android.versionCode` ו-`version`
- `android/app/build.gradle` – שדה `versionCode` ו-`versionName` בתוך `defaultConfig`

חשב את ה-versionCode הבא: versionCode הנוכחי מ-`app.json` + 1.

## שלב 2 – קרא git commits אחרונים

הרץ: `git log --oneline -15`

נתח את ה-commits:
- אם רוב השינויים הם תיקוני באגים / fixes → הצע **patch** (למשל 1.0.2 → 1.0.3)
- אם יש פיצ'רים חדשים / features → הצע **minor** (למשל 1.0.2 → 1.1.0)
- אם יש שינוי גדול / breaking → הצע **major** (למשל 1.0.2 → 2.0.0)

## שלב 3 – שאל את המשתמש על שם הגרסה

הצג:
```
🔢 versionCode: [נוכחי] → [חדש]

📋 Commits אחרונים:
[רשימת ה-commits]

💡 הצעות לשם גרסה (versionName):
  1. [patch option] – תיקונים קטנים
  2. [minor option] – פיצ'רים חדשים
  3. [major option] – שינוי גדול

מה שם הגרסה שתרצה? (הקלד מספר מהרשימה או שם מותאם אישית)
```

**המתן לתשובת המשתמש לפני שממשיכים.**

## שלב 4 – עדכן קבצים

לאחר שהמשתמש בחר שם גרסה:

**עדכן `app.json`:**
- `android.versionCode` → versionCode החדש
- `version` → שם הגרסה שנבחר

**עדכן `android/app/build.gradle`:**
- `versionCode` בתוך `defaultConfig` → versionCode החדש
- `versionName` בתוך `defaultConfig` → שם הגרסה שנבחר (בתוך מרכאות: `"1.0.3"`)

## שלב 5 – Git commit

הרץ:
```
git add app.json
git commit -m "chore: bump version to [versionName] (versionCode [versionCode])"
```

הערה: `android/` נמצא ב-`.gitignore` — אין צורך לנסות להוסיפו ל-git.

## שלב 6 – בדיקות מקדימות לפני בילד

בצע את כל הבדיקות הבאות לפני הרצת הבילד. **אל תתחיל בילד עד שכולן עברו.**

### 6א – עצור Gradle daemons ונקה תהליכי Java

הרץ בסדר הבא:
```
cd android && ./gradlew --stop
```
לאחר מכן הרוג תהליכי Java שנותרו וחכה לשחרור file handles:
```powershell
Get-Process -Name 'java' -ErrorAction SilentlyContinue | Stop-Process -Force -ErrorAction SilentlyContinue
Start-Sleep -Seconds 3
```

### 6ב – הוסף חרגות ל-Windows Defender

Windows Defender נועל קבצים שנוצרים במהלך הבילד ומונע מ-Gradle לנקות אותם. הרץ פעם אחת:
```powershell
Add-MpPreference -ExclusionPath 'C:\Users\i0548\Documents\BodyBuddy\BodyBuddy-client\node_modules' -ErrorAction SilentlyContinue
Add-MpPreference -ExclusionPath 'C:\Users\i0548\Documents\BodyBuddy\BodyBuddy-client\android' -ErrorAction SilentlyContinue
```

### 6ג – מחק תיקיות build ישנות ב-node_modules

תיקיות build שנותרו מבילדים קודמים גורמות לנעילות.
הגישה הנכונה: עבור על כל package ב-node_modules ומחק את תיקיית `android/build` ישירות (הגישה עם `-Recurse -Filter` פספסה חלק מהתיקיות בעבר):
```powershell
$nm = 'C:\Users\i0548\Documents\BodyBuddy\BodyBuddy-client\node_modules'
Get-ChildItem $nm -Directory | ForEach-Object {
    $ab = Join-Path $_.FullName 'android\build'
    if (Test-Path $ab) { Remove-Item $ab -Recurse -Force -ErrorAction SilentlyContinue }
}
```

### 6ד – מחק את תיקיית הבילד הראשית

```powershell
Remove-Item 'C:\Users\i0548\Documents\BodyBuddy\BodyBuddy-client\android\app\build' -Recurse -Force -ErrorAction SilentlyContinue
```

### 6ה – בדוק קישוריות רשת

הרץ:
```powershell
(Test-NetConnection -ComputerName 'dl.google.com' -Port 443 -WarningAction SilentlyContinue).TcpTestSucceeded
```

- אם `True` → המשך לבילד
- אם `False` → **הזהר את המשתמש:**
  ```
  ⚠️  אין גישה ל-dl.google.com.
  הבילד יכול להצליח אם כל התלויות כבר ב-Gradle cache,
  אבל אם חסרות תלויות — הבילד ייכשל לאחר זמן רב (עד שעות).
  האם להמשיך בכל זאת? (כן / לא)
  ```
  **המתן לאישור המשתמש לפני שממשיכים.**

## שלב 7 – Build

לאחר שכל הבדיקות עברו, הרץ את הבילד עם `--no-daemon` ברקע וצפה בפלט:
```bash
./gradlew bundleRelease --no-daemon 2>&1 | tail -60
```

**שים לב:** הפלט עלול להיחתך כשהוא ארוך. אם הפקודה נכשלה (exit code 1) אך השגיאה לא ברורה, הרץ שנית עם `| tail -80` כדי לראות את סוף הפלט בלבד.

אם ה-build נכשל:

- **אם השגיאה היא `Unable to delete directory`** — תהליך Java עדיין מחזיק קבצים. בצע ניקוי מלא ואז נסה שנית:
  ```powershell
  # 1. הרוג Java
  Get-Process -Name 'java' -ErrorAction SilentlyContinue | Stop-Process -Force -ErrorAction SilentlyContinue
  Start-Sleep -Seconds 3
  # 2. מחק את כל תיקיות android/build בכל ה-packages (לא רק בזו שנכשלה)
  $nm = 'C:\Users\i0548\Documents\BodyBuddy\BodyBuddy-client\node_modules'
  Get-ChildItem $nm -Directory | ForEach-Object {
      $ab = Join-Path $_.FullName 'android\build'
      if (Test-Path $ab) { Remove-Item $ab -Recurse -Force -ErrorAction SilentlyContinue }
  }
  ```
  לאחר מכן חזור להרצת הבילד.

- **אם השגיאה היא `AccessDeniedException`** → חזור לשלב 6ב (Defender לא עודכן)
- **אם השגיאה היא `Could not download` / `Could not GET`** → בעיית רשת, בדוק חיבור
- **אחרת** → הצג את השגיאה המלאה למשתמש

## שלב 8 – סיכום

אם ה-build הצליח, הצג:
```
✅ Build הושלם בהצלחה!

📦 קובץ ה-AAB:
android/app/build/outputs/bundle/release/app-release.aab

🏷️  גרסה: [versionName] | versionCode: [versionCode]
📤 עכשיו אפשר להעלות ל-Google Play Console
```

$ARGUMENTS
