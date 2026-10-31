# הגדרת מאגר F-Droid של Hebrish באמצעות adb

Hebrish היא אפליקציית הכתבה באנגלית שרצה על המכשיר עצמו וכותבת נכון מילים בעברית בתעתיק לטיני
(מעבר בין שפות, code-switching). זהו פרויקט אישי של Daniel Rosehill. F-Droid היא הדרך המומלצת להתקין
את אפליקציית האנדרואיד (`app.hebrish.dictation`, אנדרואיד 9 ומעלה, arm64 בלבד): אחרי שמוסיפים את
מאגר Hebrish, F-Droid מתקינה את Hebrish ומציעה כל גרסה חדשה. המיומנות הזו מבצעת את ההגדרה ממחשב
באמצעות adb. אנדרואיד מבקשת ממי שמחזיק את הטלפון לאשר כמה צעדים; adb אינו יכול ללחוץ על הכפתורים
האלה, ולכן יש לומר למשתמש על מה ללחוץ ולהמתין.

עובדות שבהן נעשה שימוש בהמשך:

| | |
| --- | --- |
| מאגר Hebrish | `https://fdroid.hebrish.app/repo` |
| טביעת האצבע של מפתח החתימה של המאגר (SHA-256) | `EF8C2D63D994A10CD489A978585510B47E47452EC09248591CB2C0F39C2BC1FA` |
| קישור הוספה (מאגר + טביעת אצבע) | `https://fdroid.hebrish.app/repo?fingerprint=EF8C2D63D994A10CD489A978585510B47E47452EC09248591CB2C0F39C2BC1FA`; אותו קישור עם הסכמה `fdroidrepos://` נפתח ישירות בלקוח F-Droid |
| לקוח F-Droid | `https://f-droid.org/F-Droid.apk`, חבילה `org.fdroid.fdroid`, תעודת חתימה SHA-256 ‏`43238d512c1e5eb2d6569f4a3afbf5523418b82e0a3ed1552770abb9a9c9ccab` |
| חבילות Hebrish במאגר | `app.hebrish.dictation` (‏Hebrish), וכן `app.hebrish.keyboard` (‏Hebrish Keyboard) ו-`app.hebrish.meorav` (‏Meorav, בטא) |

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
  debugging. קודם מבצעים התאמה באמצעות "Pair device with pairing code" (עם IP:port וקוד משלו), ואז
  מתחברים אל ה-IP:port שמוצג במסך Wireless debugging עצמו:

```bash
adb pair IP:PAIRING_PORT
adb connect IP:PORT
```

- **כמה מכשירים ואף אחד לא צוין:** להציג אותם ולשאול באיזה מהם מדובר. לעולם לא לנחש.
- `unauthorized`: לבקש מהמשתמש לבטל את נעילת הטלפון ולאשר "Allow USB debugging?", ואז להציג את
  הרשימה שוב. לא מוצג דבר דרך USB: ניפוי השגיאות ב-USB כבוי או שהכבל מיועד לטעינה בלבד.

בדיקה שהמכשיר יכול להריץ את Hebrish:

```bash
adb -s SERIAL shell getprop ro.build.version.sdk
adb -s SERIAL shell getprop ro.product.cpu.abilist
```

ה-SDK חייב להיות **28 ומעלה** והרשימה חייבת לכלול `arm64-v8a`; אחרת יש לעצור.

## 3. לקוח F-Droid

```bash
adb -s SERIAL shell pm list packages org.fdroid.fdroid
```

הפלט `package:org.fdroid.fdroid` פירושו ש-F-Droid מותקנת: לעבור לצעד 4. אם המשתמש משתמש בלקוח F-Droid
אחר (Neo Store ‏`com.machiav3lli.fdroid`, ‏Droid-ify ‏`com.looker.droidify`), יש לציין זאת ולהשתמש
בו בצעד 4 במקום `org.fdroid.fdroid`. אחרת יש להתקין את F-Droid:

```bash
curl -fL -o F-Droid.apk https://f-droid.org/F-Droid.apk
apksigner verify --print-certs F-Droid.apk
```

הערך `Signer #1 certificate SHA-256 digest` חייב להיות
`43238d512c1e5eb2d6569f4a3afbf5523418b82e0a3ed1552770abb9a9c9ccab`; אם הוא שונה, יש לעצור. אם
`apksigner` (מ-Android build-tools) אינו זמין, יש לומר למשתמש שהחותם לא נבדק ולשאול אם להמשיך.

```bash
adb -s SERIAL install F-Droid.apk
```

`Success` פירושו שההתקנה הצליחה. אם הטלפון מציג בקשה (בטלפונים מסוימים, למשל Xiaomi, צריך
Developer options → "Install via USB"), יש לבקש מהמשתמש לאשר.

## 4. הוספת מאגר Hebrish

יש לבקש מהמשתמש לבטל את נעילת הטלפון, ואז:

```bash
adb -s SERIAL shell am start -a android.intent.action.VIEW -d "fdroidrepos://fdroid.hebrish.app/repo?fingerprint=EF8C2D63D994A10CD489A978585510B47E47452EC09248591CB2C0F39C2BC1FA" org.fdroid.fdroid
```

F-Droid פותחת מסך "Add repository" עבור `fdroid.hebrish.app`. יש לבקש מהמשתמש לבדוק את הכתובת
וללחוץ על **Add** (הנוסח משתנה בין גרסאות F-Droid). לאחר מכן F-Droid מורידה את אינדקס המאגר; זה
יכול לקחת דקה. אם F-Droid מדווחת שטביעת האצבע אינה תואמת, יש לעצור: אין להוסיף את המאגר בלעדיה.

אם `am start` מדפיס `Error: Activity not started` או ש-F-Droid נפתחת בלי מסך ההוספה, יש לבקש מהמשתמש
לפתוח את F-Droid פעם אחת, לתת לה לסיים את העדכון הראשון שלה, ולהריץ את הפקודה שוב. אם זה עדיין לא
עובד, המשתמש מוסיף ידנית: F-Droid → Settings → Repositories → **+**, ומדביק את קישור ההוספה
שמתחיל ב-`https://` מהטבלה שלמעלה (הוא כולל את טביעת האצבע).

## 5. התקנת Hebrish מ-F-Droid

```bash
adb -s SERIAL shell am start -a android.intent.action.VIEW -d "market://details?id=app.hebrish.dictation" org.fdroid.fdroid
```

F-Droid פותחת את הדף של Hebrish. יש לבקש מהמשתמש ללחוץ על **Install**. בפעם הראשונה אנדרואיד שואלת
אם F-Droid רשאית להתקין אפליקציות ("Install unknown apps"): המשתמש מאשר וחוזר אחורה. אם
F-Droid מדווחת שהאפליקציה לא נמצאה, אינדקס המאגר עדיין לא סיים להתעדכן: יש להמתין ולהריץ את
הפקודה שוב. אם Hebrish כבר מותקנת (מ-APK מ-hebrish.app או מ-GitHub),
F-Droid מציגה **Update** או שאין מה לעשות: לשתי ההתקנות אותו מפתח חתימה, ולכן F-Droid מעדכנת את
העותק הזה ושומרת את הנתונים שלו.

## 6. אישור ודיווח

```bash
adb -s SERIAL shell dumpsys package app.hebrish.dictation | grep -E "versionName|versionCode|installerPackageName"
```

הופעת `versionName` פירושה ש-Hebrish מותקנת; הערך `installerPackageName=org.fdroid.fdroid` (או
הלקוח האחר) פירושו ש-F-Droid התקינה אותה ותציע את העדכונים שלה. יש להשוות לגרסה החדשה ביותר:

```bash
curl -fsSL https://cdn.hebrish.app/downloads/latest.json
```

(‏`files.android.version`.) יש לדווח למשתמש: המכשיר (דגם ומספר סידורי), האם F-Droid הותקנה על ידי
המיומנות הזו והאם החותם שלה נבדק, שמאגר Hebrish נוסף, והגרסה של Hebrish שמותקנת כעת. לאחר מכן
המשתמש פותח את Hebrish להגדרה בהפעלה הראשונה (היא מורידה מודל דיבור ומבקשת גישה למיקרופון, והרשאת
מקלדת או נגישות כדי להקליד באפליקציות אחרות). יש למחוק את `F-Droid.apk` אם הוא הורד. לבעיה שהמיומנות
הזו אינה מכסה, יש להפנות את המשתמש אל <https://hebrish.app/feedback>.
