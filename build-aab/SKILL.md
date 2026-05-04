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

הערה: תיקיית `android/` **מנוהלת ב-git** בפרויקט זה — אין לשנות זאת.

## שלב 5.5 – אחרי expo prebuild --clean (רק אם הורץ במפורש!)

> ⚠️ **אין להריץ `expo prebuild --clean` כחלק שגרתי מהבילד.**
> הפקודה הזו גורמת לנזק נרחב ודרשה שיקום ארוך בעבר.
> יש להריצה **רק** כאשר המשתמש מבקש זאת במפורש, למשל לתיקון אייקון.
> בבילד רגיל — דלג על שלב זה לחלוטין.

אם הורץ `expo prebuild --clean` לפני הבילד, חובה לבצע את הבדיקות הבאות לפני הבניה — הפקודה מוחקת ומשחזרת את כל תיקיית `android/` ופוגעת ב-3 דברים:

### א – שחזר את keystore המקורי מ-git

```bash
git show HEAD:android/app/bodybuddy-release.keystore > android/app/bodybuddy-release.keystore
```

### ב – שחזר את פרטי ה-signing ב-gradle.properties

בדוק מה ה-git מכיל:
```bash
git show HEAD:android/gradle.properties | grep -A5 "BODYBUDDY\|STORE\|KEY"
```
ואז עדכן את `android/gradle.properties` הנוכחי עם אותם ערכים:
```
BODYBUDDY_STORE_FILE=bodybuddy-release.keystore
BODYBUDDY_STORE_PASSWORD=[מה-git]
BODYBUDDY_KEY_ALIAS=bodybuddy
BODYBUDDY_KEY_PASSWORD=[מה-git]
```

### ג – תקן את signing config ב-build.gradle

`expo prebuild` מוחק את ה-`release` signingConfig ומחליף אותו ב-`debug`. יש לשחזר ידנית ב-`android/app/build.gradle`:

```groovy
signingConfigs {
    debug { ... }
    release {
        storeFile file(BODYBUDDY_STORE_FILE)
        storePassword BODYBUDDY_STORE_PASSWORD
        keyAlias BODYBUDDY_KEY_ALIAS
        keyPassword BODYBUDDY_KEY_PASSWORD
    }
}
buildTypes {
    release {
        signingConfig signingConfigs.release   // לא signingConfigs.debug!
        ...
    }
}
```

### ד – הסר buildDir ישן אם קיים

בדוק אם `android/build.gradle` מכיל שורה כמו:
```groovy
buildDir = "C:/tmp/bb-build/..."
```
אם כן — **מחק אותה**. היא גורמת לכשלון כי התיקייה לא קיימת.

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
- **אם השגיאה היא `Could not get unknown property 'BODYBUDDY_STORE_FILE'`** → פרטי ה-signing חסרים ב-`android/gradle.properties`. חזור לשלב 5.5ב ושחזר מ-git.
- **אם השגיאה היא `Filename longer than 260 characters`** → ninja גרסה ישנה. הפתרון הקבוע כבר בוצע (ninja 1.12.1 הותקן ב-`C:\Users\i0548\AppData\Local\Android\Sdk\cmake\3.22.1\bin\ninja.exe`). אם הבעיה חוזרת (למשל אחרי עדכון NDK), הרץ:
  ```powershell
  $zipPath = "$env:TEMP\ninja-win.zip"
  Invoke-WebRequest -Uri "https://github.com/ninja-build/ninja/releases/download/v1.12.1/ninja-win.zip" -OutFile $zipPath -UseBasicParsing
  Expand-Archive -Path $zipPath -DestinationPath "$env:TEMP\ninja-new" -Force
  Copy-Item "$env:TEMP\ninja-new\ninja.exe" "C:\Users\i0548\AppData\Local\Android\Sdk\cmake\3.22.1\bin\ninja.exe" -Force
  Remove-Item 'C:\Users\i0548\Documents\BodyBuddy\BodyBuddy-client\android\app\.cxx' -Recurse -Force -ErrorAction SilentlyContinue
  ```
  ואז חזור להרצת הבילד.
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
