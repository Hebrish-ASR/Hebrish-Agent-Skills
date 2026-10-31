# התקנת Hebrish במכשיר אנדרואיד (Sideload)

Hebrish היא אפליקציית הכתבה באנגלית שרצה על המכשיר עצמו וכותבת נכון מילים בעברית בתעתיק לטיני
(מעבר בין שפות, code-switching). זהו פרויקט אישי של Daniel Rosehill. אפליקציית האנדרואיד היא החבילה
`app.hebrish.dictation`, ל-**אנדרואיד 9 ומעלה על arm64 (arm64-v8a)** בלבד. המיומנות הזו מורידה את
קובץ ה-APK של הגרסה האחרונה ומתקינה אותו באמצעות adb במכשיר שהמשתמש בוחר.

עותקים שהותקנו ב-sideload אינם מתעדכנים לבד. לעדכונים אוטומטיים ההתקנה המומלצת היא מאגר F-Droid של
Hebrish (המיומנות `setup-hebrish-fdroid-android`). שתי השיטות משתמשות באותו מפתח חתימה, ולכן עותק
שהותקן ב-sideload ועותק מ-F-Droid מעדכנים זה את זה בלי לאבד נתונים.

עובדות שבהן נעשה שימוש בהמשך:

| | |
| --- | --- |
| חבילה | `app.hebrish.dictation` |
| קובץ הגרסאות | `https://cdn.hebrish.app/downloads/latest.json`, הרשומה `files.android` (‏`url`, ‏`name`, ‏`sha256`, ‏`size`, ‏`version`) |
| הורדה קבועה | `https://cdn.hebrish.app/apps/android/Hebrish-latest.apk` (וגם `https://hebrish.app/get/android`) |
| תעודת החתימה של ה-APK (SHA-256) | `5cf6822350d0b2ff124021a0df10bec1c63a6e5e6159fa6029507134a1446994` |

יש להריץ כל פקודה כפי שהיא כתובה, להציג למשתמש מה היא הדפיסה, ולעצור ולשאול כשצעד מורה על כך.
בכל פקודה יש להחליף את `SERIAL` במספר הסידורי של המכשיר מצעד 2.

## 1. adb

```bash
adb version
```

אם adb חסר, יש להציע להתקין את Android platform-tools: באובונטו/דביאן `sudo apt install adb`
(יש להציג את הפקודה ולשאול לפני שמריצים כל דבר עם `sudo`), במק `brew install
android-platform-tools`, ובווינדוס קובץ ה-zip של platform-tools מהכתובת
<https://developer.android.com/tools/releases/platform-tools>.

## 2. בחירת המכשיר

```bash
adb devices -l
```

- **המשתמש ציין מכשיר** (מספר סידורי, שם דגם או כתובת IP): להשתמש בו. שם דגם תואם לשדה `model:`
  ברשימה.
- **Wireless debugging** (אנדרואיד 11 ומעלה): בטלפון, Settings → Developer options → Wireless
  debugging. מחשב חדש מותאם קודם באמצעות "Pair device with pairing code", שמציג IP:port משלו
  וקוד; לאחר מכן מתחברים אל ה-IP:port שמוצג במסך Wireless debugging עצמו (פורט אחר):

```bash
adb pair IP:PAIRING_PORT
adb connect IP:PORT
```

  המספר הסידורי הוא אז `IP:PORT`.
- **כמה מכשירים ואף אחד לא צוין:** להציג אותם ולשאול באיזה מהם מדובר. לעולם לא לנחש.
- `unauthorized`: לבקש מהמשתמש לבטל את נעילת הטלפון ולאשר "Allow USB debugging?" (לסמן
  "Always allow from this computer"), ואז להציג את הרשימה שוב. `offline`: לנתק ולחבר מחדש, או
  `adb kill-server` ולהציג את הרשימה שוב. לא מוצג דבר דרך USB: ניפוי השגיאות ב-USB כבוי (Settings →
  About phone → לוחצים על Build number שבע פעמים, ואז Developer options → USB debugging) או
  שהכבל מיועד לטעינה בלבד.

## 3. בדיקת המכשיר

```bash
adb -s SERIAL shell getprop ro.product.model
adb -s SERIAL shell getprop ro.build.version.sdk
adb -s SERIAL shell getprop ro.product.cpu.abilist
adb -s SERIAL shell dumpsys package app.hebrish.dictation | grep -E "versionName|versionCode|installerPackageName"
```

- הערך `ro.build.version.sdk` חייב להיות **28 ומעלה** (אנדרואיד 9), והערך `ro.product.cpu.abilist`
  חייב לכלול `arm64-v8a`. אחרת יש לעצור: Hebrish לא רצה על המכשיר הזה.
- הפקודה האחרונה מציגה את הגרסה שכבר מותקנת, אם יש (אין פלט: לא מותקנת).

## 4. הורדת ה-APK ואימות

```bash
curl -fsSL https://cdn.hebrish.app/downloads/latest.json
```

יש לקרוא ממנו את `files.android`: ‏`version`, ‏`name`, ‏`sha256`. אם ה-`versionName` המותקן מצעד 3 
כבר שווה ל-`version`, יש לומר למשתמש שהגרסה מעודכנת ולשאול אם להתקין מחדש.

```bash
curl -fL -o Hebrish-latest.apk https://cdn.hebrish.app/apps/android/Hebrish-latest.apk
sha256sum Hebrish-latest.apk
```

(במק: `shasum -a 256 Hebrish-latest.apk`; ב-PowerShell של ווינדוס:
`Get-FileHash Hebrish-latest.apk -Algorithm SHA256`.) הגיבוב חייב להיות שווה ל-`files.android.sha256`.
אם הוא לא שווה, יש להמתין חמש דקות (ה-CDN עשוי להגיש את הקובץ הקודם עד פרק זמן כזה אחרי
שחרור גרסה), להוריד שוב ולהשוות שוב. אם עדיין יש הבדל, יש לעצור ולדווח על שני הגיבובים.

אם `apksigner` (מ-Android build-tools) זמין, יש לבדוק גם את החותם:

```bash
apksigner verify --print-certs Hebrish-latest.apk
```

הערך `Signer #1 certificate SHA-256 digest` חייב להיות
`5cf6822350d0b2ff124021a0df10bec1c63a6e5e6159fa6029507134a1446994`. בלי apksigner יש לומר
שהחותם לא נבדק; אנדרואיד עצמה מסרבת לעדכון שנחתם במפתח אחר.

## 5. התקנה

```bash
adb -s SERIAL install -r Hebrish-latest.apk
```

הדגל `-r` מעדכן התקנה קיימת ושומר את הנתונים וההגדרות שלה. `Success` פירושו שההתקנה הצליחה.
כשלונות:

| הודעה | מה לעשות |
| --- | --- |
| `INSTALL_FAILED_UPDATE_INCOMPATIBLE` | העותק המותקן נחתם במפתח אחר. רק הסרה פותרת זאת, והיא **מוחקת את נתוני האפליקציה** (היסטוריה, מילון, מודלים שהורדו). יש להסביר זאת ולשאול לפני הרצת `adb -s SERIAL uninstall app.hebrish.dictation`, ואז להתקין שוב. |
| `INSTALL_FAILED_VERSION_DOWNGRADE` | בטלפון יש גרסה חדשה יותר מזו שהורדה. להשאיר אותה. |
| `INSTALL_FAILED_NO_MATCHING_ABIS` או `INSTALL_FAILED_OLDER_SDK` | המכשיר אינו arm64 או שהוא ישן מאנדרואיד 9 (צעד 3). |
| `INSTALL_FAILED_INSUFFICIENT_STORAGE` | יש לפנות מקום בטלפון; המודלים שיורדו בהמשך דורשים עוד כמה מאות MB. |
| `INSTALL_FAILED_USER_RESTRICTED` או `INSTALL_FAILED_ABORTED` | הטלפון חסם את ההתקנה או שאל לגביה: יש לעקוב אחרי מסך הטלפון ולאשר. בטלפונים מסוימים (למשל Xiaomi) צריך גם להפעיל Developer options → "Install via USB". |

## 6. אישור ודיווח

```bash
adb -s SERIAL shell dumpsys package app.hebrish.dictation | grep -E "versionName|versionCode"
```

הערך `versionName` חייב להיות שווה ל-`files.android.version`. יש לדווח למשתמש: המכשיר (דגם ומספר
סידורי), הגרסה לפני ואחרי, ה-SHA-256 של ה-APK והאם הוא תאם, והאם החותם נבדק.

כדי לפתוח את האפליקציה בטלפון להגדרה בהפעלה הראשונה (היא מורידה מודל דיבור ומבקשת גישה למיקרופון,
ולהקלדה באפליקציות אחרות גם הרשאות מקלדת או נגישות):

```bash
adb -s SERIAL shell monkey -p app.hebrish.dictation -c android.intent.category.LAUNCHER 1
```

יש למחוק אחר כך את `Hebrish-latest.apk` אם המשתמש לא רוצה לשמור אותו. לבעיה שהמיומנות הזו
אינה מכסה, יש להפנות את המשתמש אל <https://hebrish.app/feedback>.
