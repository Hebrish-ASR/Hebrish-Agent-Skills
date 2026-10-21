# פתרון תקלות ב-Hebrish על Linux

Hebrish היא אפליקציית הכתבה של Daniel Rosehill, פרויקט של אדם אחד ללא מטרות רווח (hebrish.app). אפליקציית
ה-Linux היא החבילה `hebrish`; ההגדרות נמצאות ב-`~/.config/hebrish/config.json`, המודלים, ההיסטוריה
והמצב ב-`~/.local/share/hebrish/`, והיומן נכתב ל-stderr.

## כללים לסוכן

- **להציג, ואז לבקש אישור, לפני כל פקודת `sudo`.** בדיקות לקריאה בלבד מותר להריץ בלי לשאול.
- להחיל רק את התיקונים שלהלן. לא להמציא דגלים, מפתחות הגדרה או נתיבים, ולא לערוך את
  `/usr/lib/hebrish/`. אם שום דבר כאן לא מתאים, לאסוף את הנתונים לפי "איסוף נתונים" ולהפנות את
  המשתמש אל https://github.com/Hebrish-ASR/Hebrish-App/issues (או ללשונית About באפליקציה, שפותחת
  הודעת דוא"ל ממולאת מראש אל feedback@hebrish.app).
- לא להריץ `pkill -f hebrish`: התבנית תואמת גם את המעטפת (shell) שמריצה את הפקודה. במקום זאת לצאת
  מהאפליקציה דרך תפריט מגש המערכת (**Quit**).

## איסוף נתונים (קריאה בלבד)

```bash
. /etc/os-release; echo "$ID $VERSION_ID $VERSION_CODENAME / $(dpkg --print-architecture)"
dpkg -s hebrish | grep -E '^(Status|Version):'
apt-cache policy hebrish
echo "session=$XDG_SESSION_TYPE desktop=$XDG_CURRENT_DESKTOP WAYLAND_DISPLAY=$WAYLAND_DISPLAY DISPLAY=$DISPLAY"
id -nG | tr ' ' '\n' | grep -qx input && echo "in input group" || echo "NOT in input group"
ls -l /dev/uinput; getfacl -p /dev/uinput 2>/dev/null | grep user
command -v wl-copy xclip ffmpeg
hebrish-model status
```

**צפייה ביומן של האפליקציה:** לצאת מ-Hebrish דרך המגש, להפעיל אותה מחדש ממסוף עם `hebrish`
ולשחזר את התקלה; ההודעות ועקבות השגיאה (tracebacks) מודפסים שם. כדאי גם לקרוא כל פס הודעה בראש
החלון: האפליקציה מדווחת שם על בעיות בקיצור המקשים ובהדבקה כ-4 שניות אחרי ההפעלה.

## קיצור המקשים לא עושה כלום

1. **המשתמש לא בקבוצה `input`** (פס ההודעה אומר "Can't read /dev/input, so the hotkey is off").
   קיצור המקשים נקרא מתוך `/dev/input`. התיקון, באישור המשתמש:
   `sudo usermod -aG input $USER`, ואז **להתנתק ולהתחבר מחדש** (מסוף חדש לא מספיק;
   `id -nG` בהפעלה החדשה חייב להציג את `input`). בינתיים אפשר להכתיב עם הכפתור Talk, דרך
   המגש (לחיצה אמצעית על סמל המגש) או עם `hebrish --toggle`.
2. **המקש לא קיים במקלדת הזו.** מקש ההפעלה/כיבוי שמוגדר כברירת מחדל הוא F14, ורוב המקלדות
   חסרות אותו. להגדיר מקש אחר ב-**Settings → Dictation → Toggle key** (או **Push-to-talk
   key**, מקש לחיצה-לדיבור).
3. **אפליקציה אחרת מגיבה לאותו מקש.** המקשים נקראים בלי לתפוס אותם באופן בלעדי, ולכן מקש
   משותף עם כלי הכתבה אחר מפעיל את שניהם, והמקש מגיע גם לאפליקציה שבפוקוס. מסיבה זו ברירת המחדל
   של Parlance היא F13 ושל Hebrish היא F14. לבחור מקש שאף כלי אחר לא משתמש בו.
4. **מקש משנה (modifier) כקיצור** (למשל Ctrl הימני): לחיצה על מקש אחר בזמן שהוא מוחזק מבטלת
   את ההקלטה, וזה מכוון, כדי ש-Ctrl ימני+C יישאר קיצור מקשים רגיל.
5. מקלדות שמחוברות אחרי ההפעלה מזוהות תוך כ-3 שניות; אין צורך בהפעלה מחדש.
6. עדיין כלום: לבדוק שההפעלה דרך האפליקציה עובדת בכלל, באמצעות `hebrish --toggle`.
   אם זה מקליט, הבעיה בקלט המקשים; לדווח עליה יחד עם הפלט של איסוף הנתונים.

## הטקסט לא מודבק

איך זה עובד: האפליקציה שמה את הטקסט בלוח ההעתקה, ואז שולחת Ctrl+V דרך מקלדת וירטואלית
ב-`/dev/uinput` (בשם `hebrish-keyboard`). כל הקלטה מופיעה גם בתיבת הטקסט של הלשונית
Dictate.

1. **מצב הפלט.** בלשונית Dictate, השדה Output חייב להיות *Paste at the cursor* (ברירת המחדל).
   *Clipboard only* ו-*This window's text box only* לא מדביקים, וזה מכוון.
2. **פס ההודעה "Cannot open /dev/uinput (...); transcripts go to the clipboard only".** החבילה
   כוללת את כלל ה-udev `/usr/lib/udev/rules.d/70-hebrish-uinput.rules`, שמעניק למשתמש המחובר
   לעמדה גישה ל-`/dev/uinput` (`uaccess`). לוודא שהכלל קיים ולטעון אותו מחדש, באישור המשתמש:
   ```bash
   cat /usr/lib/udev/rules.d/70-hebrish-uinput.rules
   sudo udevadm control --reload-rules && sudo udevadm trigger --sysname-match=uinput
   ```
   אחר כך להתנתק ולהתחבר מחדש (הגישה ניתנת לכל הפעלת התחברות בנפרד) ולהפעיל מחדש את האפליקציה.
   אם קובץ הכלל חסר, להתקין מחדש: `sudo apt install --reinstall hebrish`.
3. **מסופים** מדביקים עם Ctrl+Shift+V, לא Ctrl+V. להגדיר את **Settings → Paste shortcut** ל-
   `ctrl+shift+v` אם המשתמש מכתיב בעיקר לתוך מסופים. האפליקציה לא מחליפה קיצור לפי אפליקציה.
4. **כלי לוח ההעתקה חסר** (שגיאה "Install wl-clipboard and xclip"): החבילה תלויה בשניהם,
   ולכן להתקין מחדש את התלויות: `sudo apt install wl-clipboard xclip`.
   איזה כלי משמש: ב-Wayland, `wl-copy` (KDE, מנהלי תצוגה מבוססי wlroots, Hyprland); ב-GNOME,
   קודם `xclip` דרך XWayland, כי ל-Mutter של GNOME אין את פרוטוקול data-control, ו-`wl-copy`
   שם היה גונב את הפוקוס מחלון היעד. `xclip` דורש ש-`$DISPLAY` יהיה מוגדר (XWayland
   פועל); בלעדיו האפליקציה חוזרת ל-`wl-copy`.
5. **פוקוס.** ההדבקה הולכת לחלון שיש לו את פוקוס המקלדת ברגע שהפענוח מסתיים. להשאיר את
   הסמן בחלון היעד עד שהטקסט מגיע.
6. **פריסת מקלדת עברית:** לא הסיבה. האפליקציה מדביקה ולא מקלידה, ולכן היא עובדת עם כל פריסה
   פעילה.

## אין מיקרופון, או שעוצמת הקלט נשארת שטוחה

1. בלשונית Dictate, השדה **Microphone** מציג את ההתקן שבשימוש (עבור *System default*, איזה התקן
   זה כרגע). להשתמש בבורר (עם סריקה מחדש) כדי לבחור את המיקרופון האמיתי, ואז **Test mic**: המד
   אמור להגיע לאזור *good* בזמן דיבור; *too quiet* פירושו להגביר את עוצמת הקלט בהגדרות הקול
   של המערכת; נורית CLIP פירושה להנמיך אותה.
2. לבדוק שהמערכת רואה את המיקרופון: `pactl list short sources` (PipeWire/PulseAudio). לוודא שהוא
   לא מושתק בהגדרות הקול של שולחן העבודה.
3. **"No speech" / שום דבר לא מוכנס בהקלטה קצרה.** זיהוי הקול (Voice detection, בלשונית Dictate,
   מופעל כברירת מחדל) מדלג על הקלטות שהוא מעריך כשקטות. עם מיקרופון שקט מאוד, להגביר את עוצמת
   הקלט, או לכבות את Voice detection לצורך בדיקה.

## הורדת המודל נכשלת

ההורדות מגיעות מ-cdn.hebrish.app (מ-Hugging Face עבור מודל הבסיס המקורי), גודלן כ-660 MB
(137 MB עבור Sabich-Parakeet-110M), והן נבדקות ב-SHA-256.

1. **"Not enough disk space: need N MB, have M MB"**: לפנות מקום ב-`~/.local/share/hebrish/` (או
   בתיקייה שנבחרה בלשונית Models → *Change folder…*), או לבחור שם תיקייה אחרת.
2. **"SHA-256 mismatch"** או **"size ... != ..."**: העברה פגומה או קטועה. לנסות שוב את
   ההורדה; הקובץ החלקי לא נמצא בשימוש.
3. **שגיאות רשת / פסקי זמן:** לבדוק ש-`curl -fsSI https://cdn.hebrish.app/models/parakeet-sabich/latest.json`
   מחזיר HTTP 200 מהמחשב הזה (שרתי פרוקסי וחומות אש), ואז לנסות שוב.
4. **שורת הפקודה לשגיאה ברורה יותר:** `hebrish-model check sabich` ואז `hebrish-model download sabich`
   מדפיסים את ההתקדמות ואת השגיאה המדויקת. `hebrish-model use sabich` בוחר בו; להפעיל מחדש את האפליקציה.
5. אי אפשר לעבור ממודל אחר בזמן שהוא בהורדה (הבורר מושבת). מודלים
   שאינם Sabich, Sabich-Parakeet-110M, Jachnun ו-Parakeet המקורי לא רצים באפליקציית שולחן העבודה
   ואין להם כפתור Download.

## אין סמל במגש המערכת

סמל המגש הוא StatusNotifierItem. KDE מציג אותו באופן מובנה; GNOME של Ubuntu מציג אותו דרך הרחבת
AppIndicator, שמופעלת כברירת מחדל. ב-GNOME רגיל בלי ההרחבה הזו אין מגש: סגירת החלון משאירה את
האפליקציה פועלת, ולכן כדי לקבל את החלון בחזרה יש להפעיל אותה שוב מתפריט האפליקציות, או
להשתמש ב-`hebrish --toggle`.

## שגיאות APT

| שגיאה | סיבה ותיקון |
| --- | --- |
| `NO_PUBKEY DBD1F5A22CA1B27B`, "The following signatures couldn't be verified", או "The signatures couldn't be verified because the public key is not available" | מחזיק המפתחות (keyring) ב-`/etc/apt/keyrings/hebrish-archive-keyring.gpg` חסר או שגוי. להוריד אותו מחדש ולוודא שטביעת האצבע היא `4745D63577C81400DBB743DFDBD1F5A22CA1B27B`: `curl -fsSL https://cdn.hebrish.app/apt/hebrish-archive-keyring.gpg \| sudo tee /etc/apt/keyrings/hebrish-archive-keyring.gpg >/dev/null` ואז `gpg --show-keys /etc/apt/keyrings/hebrish-archive-keyring.gpg`. אם טביעת האצבע שונה, לעצור ולדווח. |
| מחזיק המפתחות חסר אבל `/etc/apt/sources.list.d/hebrish.sources` עדיין קיים | מחזיק המפתחות נמחק ידנית, או שהמאגר הוגדר עם גרסה מוקדמת שמחזיק המפתחות שלה ישב ב-`/usr/share/keyrings`. או להסיר את המקור (`sudo rm /etc/apt/sources.list.d/hebrish.sources`, או `sudo apt purge hebrish` כדי להסיר גם את האפליקציה) או לשחזר את מחזיק המפתחות כמו למעלה, ואז `sudo apt update`. |
| "doesn't support architecture 'i386'" או שהחבילה לא נמצאת ב-arm64 | המאגר הוא ל-amd64 בלבד. השורה `Architectures: amd64` ב-`hebrish.sources` מונעת את i386; לשחזר את הקובץ בדיוק כמו בסקיל ההתקנה. ב-arm64 או בארכיטקטורות אחרות Hebrish ל-Linux עדיין לא זמינה. |
| `hebrish : Depends: python3 (< 3.15)` / `(>= 3.14)` אבל גרסה אחרת עומדת להיות מותקנת, או "Unable to locate package hebrish" בגרסת Ubuntu אחרת | החבילה מקובעת ל-python3 של Ubuntu 26.04 (3.14). גרסאות Ubuntu אחרות והפצות אחרות עדיין לא נתמכות: להסיר את המקור (`sudo rm /etc/apt/sources.list.d/hebrish.sources`) ולעיין ב-https://hebrish.app/download. |
| "Unable to locate package hebrish" ב-Ubuntu 26.04 amd64 | המקור של Hebrish לא מוגדר, למשל אחרי `apt purge hebrish` (purge מוחק את קובץ המקורות) או אחרי שהמשתמש מחק אותו (ואז הוא נשאר מחוק גם לאורך שדרוגים). להוסיף מחדש את המאגר לפי שלב 2 בסקיל ההתקנה, ואז `sudo apt update`. |
| התנגשות עם `parakeet-dictation`, `codeswitch-dictation` או `voice-to-hebrish` | צפוי: `hebrish` מחליפה את האפליקציות הקודמות האלה ו-apt מסיר אותן. לצאת מהאפליקציה הישנה לפני ההפעלה הראשונה של Hebrish. `parlance` לא מתנגשת. |
| dpkg שואל על `/etc/apt/sources.list.d/hebrish.sources` ("modified (by you or by a script)") | הקובץ בדיסק שונה מזה שהחבילה מספקת. לשמור את הגרסה של החבילה (לענות `Y`/להתקין את גרסת המתחזק) אלא אם המשתמש שינה אותו בכוונה. |

## אחרי תיקון

להפעיל מחדש את האפליקציה (מגש → Quit, ואז להפעיל את Hebrish שוב) ולחזור על הבדיקה המהירה: סמן
בעורך טקסט, קיצור המקשים, משפט אחד, קיצור המקשים. אם הבעיה נשארת, לפתוח דיווח ב-
https://github.com/Hebrish-ASR/Hebrish-App/issues עם הפלט של איסוף הנתונים ויומן המסוף,
ולציין מה כבר נוסה.
