# עדכון Hebrish

Hebrish היא אפליקציית ההכתבה של דניאל רוזהיל, פרויקט אישי (hebrish.app). לאפליקציית שולחן העבודה
יש **API מקומי** במחשב הזה. הסקיל הזה משתמש בו כדי לבדוק ולהתקין עדכונים לאפליקציה ולמודלים שלה.

## כללים לסוכן

- ה-API נמצא בכתובת `http://127.0.0.1:8723`, אלא אם המשתמש שינה את הפורט בהגדרות, Local API.
- אל תשלחו כותרת `Origin`: בקשות עם כותרת כזו נדחות (403).
- אמרו מה ישתנה לפני ההתקנה. מודלים יכולים לשקול מאות MB.
- אל תפעילו מחדש את האפליקציה ואל תחליפו מודל כש-`state` אינו `idle`; גם ה-API מסרב לכך.

## 1. בדיקה שהשליטה פעילה

```bash
curl -s http://127.0.0.1:8723/health
```

- `"control": true`: המשיכו.
- `"control": false`: בקשו מהמשתמש לפתוח את Hebrish, **Settings → Local API**, ולסמן את *Let agents
  read history and control the app*.
- אין מפתח `control`: הגרסה היא 1.017 או ישנה ממנה. עדכנו בדרך הישנה (באובונטו,
  `sudo apt update && sudo apt install --only-upgrade hebrish`, אחרי שהראיתם למשתמש את הפקודה).
- החיבור נדחה: Hebrish לא פועלת, או שה-API המקומי כבוי. בקשו מהמשתמש להפעיל אותה ולסמן את
  *Serve transcription to apps on this computer* באותו כרטיס.

## 2. האפליקציה

```bash
curl -s http://127.0.0.1:8723/v1/app/update
```

בתשובה יש `current`, `latest`, `update_available`, `method`, `can_install` ו-`download_page`.

- `can_install` הוא true (`method` הוא `apt`): התקינו. המשתמש צריך לאשר בקשת סיסמה של המערכת על
  המסך שלו, אז הודיעו לו קודם.

  ```bash
  curl -s -X POST http://127.0.0.1:8723/v1/app/update
  ```

  בדקו את `GET /v1/app/status` כל כמה שניות: `app_update.running` הופך ל-false, ואז
  `app_update.installed_needs_restart` מכיל את הגרסה, או ש-`app_update.error` אומר מה נכשל (גם
  דחיית בקשת הסיסמה היא שגיאה). ייתכן שהאפליקציה גם תשאל את המשתמש על הפעלה מחדש. אם הוא לא ענה
  והאפליקציה פנויה, הפעילו אותה מחדש:

  ```bash
  curl -s -X POST http://127.0.0.1:8723/v1/app/restart
  ```

- `update_available` הוא true אבל `can_install` הוא false: ההתקנה הזו (`windows`, `appimage`,
  `deb-manual`) לא יכולה לעדכן את עצמה. תנו למשתמש את `download_page`. `source` הוא עותק מקוד
  מקור: אמרו שיש גרסה חדשה.

## 3. המודלים

```bash
curl -s http://127.0.0.1:8723/v1/app/models/updates
```

`models` ו-`addons` מפרטים מה חדש יותר ממה שמותקן; `errors` מפרט בדיקות שנכשלו. דווחו על הכישלונות,
ואל תקראו להם מעודכנים. כדי להוריד את כל מה שברשימה:

```bash
curl -s -X POST http://127.0.0.1:8723/v1/app/models/updates
```

בדקו את `GET /v1/app/status` עד ש-`models.downloading` הוא null, `models.queued` הוא 0
ו-`models.addons_downloading` ריק.

## 4. איזה מודל, והחלפה

```bash
curl -s http://127.0.0.1:8723/v1/app/models
curl -s http://127.0.0.1:8723/v1/app/status
```

לכל מודל יש `id`, `name`, `installed`, `version`, `current` ו-`recommended`. כדי להחליף:

```bash
curl -s -X POST http://127.0.0.1:8723/v1/app/models/select \
  -H 'Content-Type: application/json' -d '{"id": "sabich"}'
```

מודל שלא הורד עונה 409 עם הגודל שלו. הוסיפו `"download": true` אחרי שהמשתמש הסכים.

## שגיאות

שגיאות מגיעות כ-`{"error": {"message": "..."}}`: 400 בקשה שגויה, 403 השליטה כבויה, 404 מזהה או
נתיב לא מוכרים, 409 האפליקציה לא יכולה לעשות זאת עכשיו או בהתקנה הזו. `GET /v1` מפרט את כל הנתיבים.
