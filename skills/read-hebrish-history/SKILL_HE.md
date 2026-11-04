# קריאת ההיסטוריה של Hebrish

Hebrish היא אפליקציית ההכתבה של דניאל רוזהיל, פרויקט אישי (hebrish.app). אפליקציית שולחן העבודה
שומרת כל תמליל במחשב הזה, וה-**API המקומי** שלה מגיש אותם לתוכנות באותו מחשב. הטקסט הוא אנגלית
שבה מילים בעברית כתובות בתעתיק לטיני (*makolet*, *arnona*).

## כללים לסוכן

- ה-API נמצא בכתובת `http://127.0.0.1:8723`, אלא אם המשתמש שינה את הפורט בהגדרות, Local API.
- אל תשלחו כותרת `Origin`: בקשות עם כותרת כזו נדחות (403).
- התמלילים פרטיים. הביאו רק מה שהמשימה צריכה, ואל תעתיקו קטעים ארוכים או תשלחו אותם לשירות אחר
  אלא אם המשתמש ביקש.
- ה-API קורא בלבד. עריכה או מחיקה של תמלילים נעשות בלשונית History באפליקציה.

## 1. בדיקה שהשליטה פעילה

```bash
curl -s http://127.0.0.1:8723/health
```

`"control": true` אומר להמשיך. `"control": false`: בקשו מהמשתמש לפתוח את Hebrish, **Settings →
Local API**, ולסמן את *Let agents read history and control the app*. אין מפתח `control`: האפליקציה
ישנה מדי, השתמשו בסקיל `update-hebrish`. החיבור נדחה: Hebrish לא פועלת, או שה-API המקומי כבוי.

## 2. התמלילים האחרונים

```bash
curl -s 'http://127.0.0.1:8723/v1/history?limit=20'
```

התשובה היא `{"object": "list", "total": N, "limit": ..., "offset": ..., "data": [...]}`, מהחדש
לישן. בכל שורה יש `id`, `uid`, `at` (זמן יוניקס), `text`, `words`, `seconds`, `model`, `source`
(`dictate`, `record`, `file`, `import`) ו-`original` (הטקסט של המודל, אם המשתמש ערך אותו; אחרת null).

## 3. חיפוש וסינון

```bash
curl -s -G http://127.0.0.1:8723/v1/history --data-urlencode 'q=arnona bill' -d limit=50
```

`q` הוא חיפוש מילות מפתח, ההתאמה הטובה ביותר קודם, ומוסיף `snippet` עם ההתאמות ב-`[סוגריים]`.
פרמטרים נוספים, שאפשר לשלב איתו:

| פרמטר | משמעות |
| --- | --- |
| `limit`, `offset` | גודל עמוד (1 עד 1000, ברירת מחדל 50) ונקודת התחלה |
| `since`, `until` | זמן יוניקס; `until` לא כולל |
| `source` | אחד מערכי `source` שלמעלה |
| `edited=1` | רק תמלילים שהמשתמש תיקן ידנית |

אתמול, בלינוקס:

```bash
curl -s -G http://127.0.0.1:8723/v1/history \
  -d since=$(date -d 'yesterday 00:00' +%s) -d until=$(date -d 'today 00:00' +%s) -d limit=1000
```

תמליל אחד: `curl -s http://127.0.0.1:8723/v1/history/123`.

## 4. ייצוא של הכול

עברו על `offset` עד שעמוד חוזר קצר:

```bash
offset=0
while :; do
  page=$(curl -sf "http://127.0.0.1:8723/v1/history?limit=1000&offset=$offset") || break
  echo "$page" | jq -c '.data[]' >> hebrish-history.jsonl
  [ "$(echo "$page" | jq '.data | length')" -lt 1000 ] && break
  offset=$((offset + 1000))
done
```

נדרש `jq`. שאלו לאן לכתוב אם המשתמש לא אמר. גם ב-Settings, Your data של האפליקציה אפשר לייצא
ולגבות.

## מגבלות

- ב-`GET /v1/app/status` יש `keep_history`. כשהוא false האפליקציה לא שומרת תמלילים חדשים, אז תוצאה
  ריקה לא אומרת כלום על מה שהוכתב.
- ההיסטוריה היא לכל מחשב בנפרד. ההיסטוריה של אפליקציית Android לא נמצאת כאן.
- קבצים שנשלחים ל-API המקומי לתמלול לא נשמרים ב-History.
