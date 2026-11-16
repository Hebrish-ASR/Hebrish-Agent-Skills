# הקמת Macro Deck עבור היבריש

היבריש היא אפליקציית ההכתבה של דניאל רוזהיל, פרויקט אישי (hebrish.app). לאפליקציית שולחן העבודה
יש **API מקומי** בכתובת `http://127.0.0.1:8723`, שתוכנות אחרות באותו מחשב יכולות להשתמש בו כדי
להתחיל ולעצור הכתבה ולהחליף מודל. [Macro Deck](https://macro-deck.app) 3 היא אפליקציית stream deck
חינמית וקוד פתוח: היא רצה במחשב ומציגה את הכפתורים שלה בטלפון, בטאבלט או בדפדפן. כל כפתור בדק שולח
בקשת HTTP אחת מהמחשב שמריץ את Macro Deck אל היבריש, ולכן אין צורך בפוקוס על חלון והכול עובד גם
ב-Wayland. הטקסט מוקלד בחלון שנמצא בפוקוס במחשב, כמו בקיצור המקשים של היבריש.

הדק (2 עמודות, 4 שורות, נוח לאגודל בטלפון שמוחזק זקוף):

| כפתור | טריגר | בקשה |
| --- | --- | --- |
| Sabich | לחיצה | `POST /v1/app/models/select` `{"id": "sabich-rolling", "download": false}` |
| Shakshuka | לחיצה | `POST /v1/app/models/select` `{"id": "shakshuka", "download": false}` |
| Falafel | לחיצה | `POST /v1/app/models/select` `{"id": "falafel", "download": false}` |
| Stop | לחיצה | `POST /v1/app/dictation` `{"action": "stop"}` |
| Tap: start / stop (ברוחב 2) | לחיצה | `POST /v1/app/dictation` `{"action": "toggle"}` |
| Hold to talk (ברוחב 2, למטה) | תחילת מגע, סוף מגע | `{"action": "start"}`, ואחר כך `{"action": "stop"}` |

התבנית, כמפרט נייד, נמצאת בכתובת
<https://raw.githubusercontent.com/Hebrish-ASR/Hebrish-Agent-Skills/main/skills/set-up-hebrish-macro-deck/assets/hebrish-dictation.deck.json>.

## כללים לסוכן

- הכול כאן רץ במחשב של המשתמש עצמו. אל תפתחו את הפורטים של Macro Deck או של היבריש לאינטרנט.
- אל תבדקו בעצמכם את כפתורי ההכתבה: `start` מקליט מהמיקרופון של המשתמש ומקליד בחלון שנמצא אצלו
  בפוקוס. בדיקה לא מזיקה היא מעבר למודל שכבר נבחר.
- החלפות מודל משתמשות ב-`"download": false`. מודל שלא הורד עונה בשגיאה; אמרו למשתמש להוריד אותו
  בלשונית Models של היבריש (כ-660 MB) ואל תורידו אותו מהדק.
- הראו למשתמש כל פקודה שדורשת `sudo` ובקשו אישור לפני שאתם מריצים אותה.

## 1. בדיקת היבריש והפעלת ה-API המקומי

```bash
curl -s http://127.0.0.1:8723/v1/app/status
```

- תשובת JSON עם `"state"` (`idle`, `recording` או `busy`): מוכן. עברו לשלב 2.
- `403` עם `control`: ה-API המקומי פועל אבל השליטה כבויה.
- החיבור נדחה: ה-API המקומי כבוי, או שהיבריש לא פועלת.

בשני המקרים האחרונים, בקשו מהמשתמש לפתוח את היבריש, **Settings → Local API**, ולסמן את שתי
האפשרויות *Serve transcription to apps on this computer* ו-*Let agents read history and control the app*.
אם הוא מעדיף שתעשו זאת אתם: סגרו את היבריש, הגדירו `"local_api": true` ו-
`"local_api_control": true` בקובץ ההגדרות שלה (`~/.config/hebrish/config.json` בלינוקס,
`%APPDATA%\Hebrish\config.json` בווינדוס), הפעילו את היבריש מחדש והריצו שוב את הבדיקה. אם בהגדרות
אין כרטיס Local API, הגרסה של היבריש ישנה מ-1.018: עדכנו אותה קודם (הסקיל `update-hebrish`, או
באובונטו `sudo apt update && sudo apt install hebrish`).

המתג השני מאפשר גם לתוכנות מקומיות לקרוא את היסטוריית התמלולים; אמרו זאת כשאתם מבקשים.

אחר כך הציגו את רשימת המודלים, כדי שתוכלו להגיד למשתמש אילו כפתורים יעבדו כבר עכשיו:

```bash
curl -s http://127.0.0.1:8723/v1/app/models
```

מודלים עם `installed: true` מתחלפים מיד; את האחרים צריך קודם להוריד בהיבריש.

## 2. בדיקת Macro Deck

Macro Deck 3 חייבת להיות מותקנת ופועלת באותו מחשב (לינוקס: מאגר ה-APT שלה בכתובת
`packages.macro-deck.app`; ווינדוס ואחרות: <https://macro-deck.app>). המארח שלה מאזין כברירת מחדל
בפורט 8193:

```bash
ss -ltn | grep -E ':8193'          # Linux
```

המשתמש פותח את הדק בטלפון או בטאבלט בסריקת קוד ההתאמה ש-Macro Deck מציגה, או בדפדפן. Macro Deck 2
ואפליקציית האנדרואיד הישנה שלה לא עובדות עם הדק הזה: הוא דורש גרסה 3.

## 3. התקנת הדק

בחרו את המסלול הראשון שמתאים.

### א. ייבוא התבנית (כל משתמש, בלי כלי סוכן)

1. הורידו את
   <https://hebrish.app/downloads/macro-deck/hebrish-dictation.macroDeckProfile>.
2. ב-Macro Deck, פתחו את תפריט הפרופיל ובחרו **Import profile**, ואז את הקובץ. הוא מגיע כפרופיל
   *Hebrish Dictation*, עם האייקונים שלו.
3. בטלפון, החליפו את הדק לפרופיל הזה.

אם ההורדה עדיין לא זמינה, השתמשו במסלול ב או ג.

### ב. בנייה דרך שרת ה-MCP של Macro Deck

אם יש לכם את כלי ה-MCP של Macro Deck (`catalog`, `list`, `apply_changes`, ...): המשתמש יוצר
טוקן ב-**Settings → AI clients → Create token** ומחבר אתכם; ראו את התיעוד של Macro Deck. אחר כך:

1. הביאו את JSON התבנית שלמעלה.
2. אייקונים: קראו את `${XDG_DATA_HOME:-$HOME/.local/share}/MacroDeck/data/icons/packs/*/pack.json`
   (ווינדוס: תיקיית הנתונים של Macro Deck) ומצאו את החבילה בשם `MDI-Icons`; ה-`icon` של כל כפתור
   הוא `icons[].name` שם, וה-`icons[].id` שלו הוא ההפניה. בלי החבילה, בקשו מהמשתמש להתקין את
   *MDI-Icons* מה-**Store** של Macro Deck, או בנו בלי אייקונים.
3. `list {entity: "profiles"}`. אם אין פרופיל *Hebrish Dictation*, צרו אותו עם
   `apply_changes` (`{op: "create", entity: "profile", data: {name: "Hebrish Dictation"}}`). אחר כך
   `list {entity: "folders", profileId}` והשתמשו בתיקיית ברירת המחדל שלו (צרו `Home` אם אין כזו).
   שימו לב ש-`list` ו-`get` מקבלים שמות ישויות ברבים, ו-`apply_changes` ביחיד.
4. אצווה אחת של `apply_changes`, קודם עם `dryRun: true` ואז באמת:
   `update folder {id, data: {name: "Home", rows: 4, columns: 2}}`, ואחריו
   `create widget` אחד לכל כפתור:

```json
{"op": "create", "entity": "widget", "data": {
  "folderId": "<folder id>", "type": "ActionButton",
  "positionX": 0, "positionY": 3, "width": 2, "height": 1,
  "data": {
    "label": "Hold to talk", "backgroundColor": "#0f6b5c",
    "labelColor": "#ffffff", "labelPosition": "bottom",
    "icon": {"type": "icon-pack", "reference": "<icon id>"}, "iconColor": "#ffffff",
    "iconDisplay": {"fit": "contain", "zoom": 55, "offsetX": 0, "offsetY": -12, "opacity": 100},
    "flows": [
      {"triggerId": "<new uuid>", "triggerType": "onTouchStart", "triggerLabel": "Touch start",
       "children": [{"id": "<new uuid>", "type": "action",
         "blockType": "app.macro-deck.http.send-request", "label": "Send HTTP Request",
         "color": "#4f7cff", "integrationId": "app.macro-deck.http", "actionId": "send-request",
         "parameters": [
           {"name": "method", "type": "choice", "label": "Method", "value": "POST"},
           {"name": "url", "type": "url", "label": "URL", "value": "http://127.0.0.1:8723/v1/app/dictation"},
           {"name": "bodyType", "type": "choice", "label": "Body type", "value": "json"},
           {"name": "jsonBody", "type": "json", "label": "JSON body", "value": "{\"action\": \"start\"}"},
           {"name": "expectedStatus", "type": "string", "label": "Expected status", "value": "2xx"},
           {"name": "timeout", "type": "duration", "label": "Timeout", "value": 5000}]}]},
      {"triggerId": "<new uuid>", "triggerType": "onTouchEnd", "triggerLabel": "Touch end",
       "children": ["... the same block with {\"action\": \"stop\"} ..."]}
    ]}}}
```

   `positionX` הוא ה-`col` של התבנית, ו-`positionY` הוא ה-`row` שלה. כפתורי לחיצה משתמשים ב-`triggerType`
   `onShortPress` (‏`triggerLabel` הוא `Short press`). `jsonBody` הוא **מחרוזת** JSON. כל `id` ו-
   `triggerId` הוא UUID חדש. אצווה שעברה את ה-dry run עדיין יכולה להיכשל בהחלה אם שני
   כפתורים חולקים תא: היא מתבטלת (rollback), אז תקנו והחילו שוב.
5. הוכיחו שהמסלול עובד בלי לגעת במיקרופון: `run_action` עם `app.macro-deck.http` /
   `send-request`, ‏`POST` אל `/v1/app/models/select` עם המודל שכבר נבחר כרגע. התשובה
   צריכה להיות `Succeeded`.

### ג. ידנית ב-Macro Deck

צרו את הפרופיל ואת ששת הכפתורים בעורך של Macro Deck לפי הטבלה שבראש המסמך. כל כפתור
מקבל פעולה **HTTP → Send HTTP Request**, שיטה `POST`, סוג גוף JSON. עבור *Hold to talk*, הוסיפו
שני טריגרים לאותו כפתור: **Touch start** שולח `start`, **Touch end** שולח `stop`.

## 4. מסירה

בקשו מהמשתמש להניח את הסמן בשדה טקסט במחשב, ואז בטלפון:

- להחזיק את **Hold to talk**, להגיד משפט ולשחרר: הטקסט מוקלד;
- ללחוץ על **Tap: start / stop**, לדבר וללחוץ שוב.

אם שום דבר לא קורה, בדקו את `curl -s http://127.0.0.1:8723/v1/app/status` בזמן שהם לוחצים (הוא
אמור להראות `recording`), ושהטלפון מציג את הפרופיל *Hebrish Dictation*. לחיצה שלא משנה דבר בסטטוס
אומרת שה-URL או הפורט של הכפתור שגויים (ייתכן שהמשתמש שינה את הפורט ב-Settings, Local API: השתמשו
בשלו). בעיות לא מוכרות:
<https://github.com/Hebrish-ASR/Hebrish-App/issues>.
