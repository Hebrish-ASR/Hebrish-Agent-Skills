# התקנת Hebrish בלינוקס

Hebrish היא אפליקציית הכתבה באנגלית שרצה על המכשיר עצמו וכותבת נכון מילים בעברית בתעתיק לטיני
(מעבר בין שפות, code-switching): מודלי NVIDIA Parakeet שעברו כוונון עדין רצים על המעבד (CPU) באמצעות sherpa-onnx.
זהו פרויקט של אדם אחד, ללא מטרות רווח, מאת דניאל רוזהיל (Daniel Rosehill). אפליקציית הלינוקס היא החבילה `hebrish`:
חלון ועוד סמל במגש המערכת; לוחצים על קיצור מקשים, מדברים, והטקסט מודבק במיקום הסמן.

## כללים לסוכן

- **לפני כל פקודת `sudo`: קודם להציג, אחר כך לשאול.** יש להדפיס את הפקודה או הפקודות המדויקות שעומדים
  להריץ ולקבל את אישור המשתמש. בדיקות לקריאה בלבד (ללא `sudo`) אפשר להריץ בלי לשאול.
- אין לדלג על בדיקת טביעת האצבע של המפתח, ויש לעצור אם היא אינה תואמת.
- אין לערוך קבצים תחת `/usr/lib/hebrish/`, ואין להמציא דגלים, נתיבים או הגדרות שהמיומנות הזו
  אינה מזכירה. אם משהו כאן אינו מכסה את מצבו של המשתמש, יש להשתמש במיומנות
  `troubleshoot-hebrish-linux` או להפנות את המשתמש אל
  https://github.com/Hebrish-ASR/Hebrish-App/issues.
- הורדת המודל (כ-660 MB) מתבצעת על ידי האפליקציה, לא על ידי `apt`. יש לציין זאת לפני שהיא
  מתחילה בחיבור עם הגבלת נפח.

## 1. בדיקות מקדימות (קריאה בלבד)

יש להריץ את הפקודות הבאות ולקרוא את הפלט:

```bash
. /etc/os-release; echo "ID=$ID VERSION_ID=$VERSION_ID CODENAME=$VERSION_CODENAME"
dpkg --print-architecture
dpkg -s hebrish 2>/dev/null | grep -E '^(Status|Version):' || echo "hebrish: not installed"
dpkg -l parakeet-dictation codeswitch-dictation voice-to-hebrish 2>/dev/null | grep '^ii' || echo "no predecessor apps"
echo "session: $XDG_SESSION_TYPE  desktop: $XDG_CURRENT_DESKTOP"
id -nG | tr ' ' '\n' | grep -qx input && echo "in input group" || echo "NOT in input group"
```

החלטה:

| תוצאה | מה לעשות |
| --- | --- |
| `ID=ubuntu`, `VERSION_ID=26.04` (resolute), ארכיטקטורה `amd64` | נתמך. להמשיך. |
| כל הפצה, גרסת Ubuntu או ארכיטקטורה אחרת | **עדיין לא נתמך.** החבילה זמינה ל-amd64 בלבד ומקובעת לגרסת המשנה של python3 ב-Ubuntu 26.04 (3.14), כך ש-`apt` יסרב להתקין אותה או שהיא לא תעלה. יש לעצור, להסביר זאת למשתמש ולהפנות אותו אל https://hebrish.app/download. אין להוסיף את המאגר. |
| `hebrish` כבר מותקנת | לבדוק אם המאגר כבר מוגדר (באמצעות `apt-cache policy` משלב 3). אם כן, עדכון הוא פשוט `sudo apt update && sudo apt upgrade`. אם לא (הותקנה מקובץ ‎.deb לפני 0.6.2), להוסיף את המאגר (שלב 2) כדי שעדכונים יגיעו. |
| מותקנת אפליקציה קודמת (`parakeet-dictation`, `codeswitch-dictation`, `voice-to-hebrish`) | החבילה `hebrish` מתנגשת בהן ומחליפה אותן, ולכן `apt` יציע להסיר אותן. יש לומר זאת למשתמש ולבקש ממנו **לסגור את האפליקציה הישנה לפני ההפעלה הראשונה של Hebrish**: בהפעלה הראשונה Hebrish מעבירה את המודל שהאפליקציה הישנה הורידה אל התיקייה שלה, ואפליקציה ישנה שעדיין רצה תוריד אז שוב 660 MB. Hebrish מאמצת את ההגדרות ואת המודל שלה, כך ששום דבר לא יורד פעמיים. |
| `parlance` מותקנת | בסדר. Parlance היא אפליקציה נפרדת ומותקנת לצד Hebrish; בהפעלה הראשונה Hebrish מעתיקה את ההגדרות של Parlance (לא את קיצור המקשים) ויוצרת קישורים קשיחים (hardlinks) למודלים שלה. Parlance משתמשת ב-F13 ו-Hebrish ב-F14, כך שאין להן קיצור מקשים משותף. |

## 2. הוספת המאגר והתקנה

מאגר: `https://cdn.hebrish.app/apt`, סוויטה `stable`, רכיב `main`, ארכיטקטורה `amd64`.
מפתח חתימה: ‏"Hebrish APT Repository <info@hebrish.app>", ‏RSA 4096, טביעת אצבע
`4745 D635 77C8 1400 DBB7  43DF DBD1 F5A2 2CA1 B27B`.

יש להציג למשתמש את השלבים האלה, ואז להריץ אותם בלוק אחר בלוק (כל `sudo` דורש הסכמה):

```bash
# 2a. Download the signing keyring
sudo install -d -m 0755 /etc/apt/keyrings
curl -fsSL https://cdn.hebrish.app/apt/hebrish-archive-keyring.gpg | sudo tee /etc/apt/keyrings/hebrish-archive-keyring.gpg >/dev/null

# 2b. Verify the fingerprint (no sudo)
gpg --show-keys /etc/apt/keyrings/hebrish-archive-keyring.gpg
```

הפלט חייב להציג את `4745D63577C81400DBB743DFDBD1F5A22CA1B27B` ואת מזהה המשתמש (uid)
‏`Hebrish APT Repository <info@hebrish.app>`. אם לא, **יש לעצור**, למחוק את הקובץ
(`sudo rm /etc/apt/keyrings/hebrish-archive-keyring.gpg`) ולדווח על כך בכתובת
https://github.com/Hebrish-ASR/Hebrish-App/issues. (עותק בפורמט armored של אותו מפתח נמצא בכתובת
https://cdn.hebrish.app/apt/hebrish-archive-keyring.asc.)

```bash
# 2c. Add the source. Keep this content byte-for-byte as written: the package ships the
#     same file as a conffile, and an identical file is adopted silently on install.
sudo tee /etc/apt/sources.list.d/hebrish.sources >/dev/null <<'EOF'
Types: deb
URIs: https://cdn.hebrish.app/apt
Suites: stable
Components: main
Architectures: amd64
Signed-By: /etc/apt/keyrings/hebrish-archive-keyring.gpg
EOF

# 2d. Install
sudo apt-get update && sudo apt-get install -y hebrish
```

**הגרסה המקוצרת.** אם המשתמש מעדיף פקודה אחת, סקריפט ההתקנה מבצע את 2a, ‏2c ו-2d (צרור המפתחות,
קובץ המקורות, `apt-get update`, ‏`apt-get install hebrish`). הוא אינו מציג את טביעת האצבע, ולכן
יש להריץ את 2b אחריו:

```bash
curl -fsSL https://cdn.hebrish.app/apt/setup.sh | sudo sh
```

**חלופה: קובץ ה-‎.deb.** התקנת החבילה שהורדה מ-https://hebrish.app/get/ubuntu
(`sudo apt install ./hebrish_*_amd64.deb`) גם היא רושמת את המחשב במאגר, החל מגרסת
השולחן 0.6.2: החבילה עצמה מתקינה את צרור המפתחות ואת קובץ המקורות שלמעלה.

## 3. אימות

```bash
apt-cache policy hebrish        # the Installed version, and a line with https://cdn.hebrish.app/apt stable/main amd64
dpkg -s hebrish | grep -E '^(Status|Version):'    # Status: install ok installed
command -v hebrish hebrish-model                  # /usr/bin/hebrish, /usr/bin/hebrish-model
```

‏`apt` מושך את התלויות בעצמו (PySide6, ‏python3-evdev, ‏python3-sounddevice,
‏libportaudio2, ‏wl-clipboard, ‏xclip, ‏qt6-wayland, ...); ‏`ffmpeg` מומלץ, לפתיחת
קובצי שמע בכל פורמט.

## 4. הגדרה בהפעלה הראשונה

**4a. הרשאה לקיצור המקשים (דורשת התחברות מחדש).** קיצור המקשים נקרא מ-`/dev/input`, ולשם כך נדרשת
החברות בקבוצה `input`. אם הבדיקות המקדימות החזירו "NOT in input group", יש להציג ולהריץ (בהסכמה):

```bash
sudo usermod -aG input $USER
```

לאחר מכן המשתמש חייב **להתנתק ולהתחבר מחדש** (או לאתחל) לפני שקיצור המקשים יעבוד. עד אז
האפליקציה מציגה על כך הודעה בפס עליון, וההכתבה עדיין עובדת מכפתור Talk, ממגש המערכת או
מ-`hebrish --toggle`.

ההדבקה עצמה עוברת דרך מקלדת וירטואלית ב-`/dev/uinput`. החבילה מתקינה לשם כך כלל udev
(`/usr/lib/udev/rules.d/70-hebrish-uinput.rules`, אותו כלל `uaccess` ש-Steam
מתקינה) וטוענת מחדש את כללי udev בזמן ההתקנה; אין צורך לעשות דבר נוסף.

**4b. הפעלה.** יש לפתוח את **Hebrish** מתפריט האפליקציות (או להריץ `hebrish`). היא נשארת במגש המערכת
כשהחלון נסגר. ב-GNOME של Ubuntu סמל המגש מוצג באמצעות ההרחבה AppIndicator,
ש-Ubuntu מפעילה כברירת מחדל.

**4c. בחירת מודל והורדתו.** בהפעלה הראשונה האפליקציה פותחת את הלשונית **Models**. יש להמליץ
על **Sabich** (המודל המלא: מעבר בין שפות ועוד אוצר מילים טכני). כל מודל בגודל 0.6B הוא
הורדה חד-פעמית של כ-660 MB, שנבדקת ב-SHA-256. המודלים שאפליקציית השולחן יכולה להריץ:

| מודל | מזהה | הורדה | מתי |
| --- | --- | --- | --- |
| **Sabich** (מומלץ) | `sabich` | ~660 MB | להתחיל כאן; כל מחשב עדכני, ללא GPU, כ-1.5 GB זיכרון RAM |
| Sabich-Parakeet-110M | `sabich-parakeet-110m` | ~137 MB | מחשבים ישנים או עם מעט זיכרון: מהיר יותר, יותר טעויות |
| Jachnun | `codeswitch` | ~660 MB | מילים בעברית בלי אוצר המילים הטכני (גם Sabich מכסה זאת) |
| Parakeet (המקורי) | `stock` | ~630 MB | אנגלית רגילה, ללא מילים בעברית |

מודלים אחרים שמופיעים ב-hebrish.app ‏(Sabich-Qwen3-ASR-1.7B, ‏Sabich-Canary-Qwen-2.5B,
‏Sabich-asr4all, ‏Shakshuka) עדיין אינם רצים באפליקציית השולחן; הלשונית Models מציגה אותם באפור.
הכפתור **Help me choose** באותה לשונית בוחר מודל מתאים למחשב.

המשתמש לוחץ על **Download** בכרטיס של Sabich, ואחר כך על **Switch to Sabich** אם הוא עדיין
אינו בשימוש. או, מהמסוף (ללא sudo; האפליקציה קוראת את הבחירה מחדש בהפעלה הבאה שלה):

```bash
hebrish-model list               # ids and which one is selected/installed
hebrish-model download sabich
hebrish-model use sabich
```

**4d. קיצור מקשים.** מקש ההפעלה/כיבוי המוגדר כברירת מחדל הוא **F14**: הקשה אחת מתחילה, הקשה נוספת עוצרת;
החזקה שלו פועלת כ"לחץ ודבר" (push-to-talk). במקלדות רבות אין F14, ולכן יש לשאול את המשתמש באיזה מקש הוא רוצה. משנים אותו
ב-**Settings → Dictation → Toggle key** (כל מקש בכל מקלדת, או דוושת רגל), ואפשר גם
להגדיר שם **Push-to-talk key** נפרד. יש להימנע ממקש שכלי הכתבה אחר כבר משתמש בו: המקשים
נקראים בלי ללכוד אותם, כך ששתי האפליקציות יגיבו. כדי להשתמש במקום זאת בקיצור של סביבת שולחן העבודה,
יש לקשר אותו לפקודה `hebrish --toggle`.

**4e. בדיקה מהירה.** יש למקם את הסמן בעורך טקסט, ללחוץ על קיצור המקשים, לומר משפט שיש בו
מילה בעברית ("I need to renew my teudat zehut at the misrad hapnim") וללחוץ שוב. הטקסט
אמור להיות מודבק במיקום הסמן. הגדרות רשות שכדאי להזכיר: **Start at login, in
the tray** (בהגדרות, Settings), ו-**Paste shortcut** ‏`ctrl+shift+v` למשתמשים שמכתיבים בעיקר
לתוך מסופים.

אם משהו נכשל בשלב הזה, יש לעבור למיומנות `troubleshoot-hebrish-linux`.

## 5. עדכונים

- **האפליקציה:** דרך APT כמו כל חבילה אחרת: `sudo apt update && sudo apt upgrade`, או כלי העדכון
  של שולחן העבודה (Software Updater, ‏Discover, ‏GNOME Software). האפליקציה מציעה גם
  עדכונים מתוכה.
- **מודלים:** האפליקציה בודקת לפי לוח זמנים אם יש גרסה חדשה של המודל הנבחר ומתקינה אותה
  בעצמה (Settings → Model updates הופך זאת לכפתור Update במקום). הגרסה הישנה
  נמחקת אחרי שהחדשה עברה אימות.
- אם המשתמש מוחק את `/etc/apt/sources.list.d/hebrish.sources`, הוא נשאר מחוק גם בשדרוגי
  החבילה (זהו conffile), והעדכונים נפסקים עד שהוא נכתב מחדש (שלב 2c).

## 6. הסרה

יש להציג ולקבל אישור, ואז:

```bash
sudo apt purge hebrish    # removes the app, the keyring and the repository's sources file
```

‏`sudo apt remove hebrish` מסיר את האפליקציה אבל משאיר את צרור המפתחות ואת קובץ המקורות (שניהם
conffiles), כך שהמאגר נשאר מוגדר והתקנה מאוחרת יותר ב-`apt install hebrish` תעבוד שוב;
יש להשתמש ב-`purge` כדי להסיר גם אותם. יש לבדוק אם נותרו שאריות
(אם המאגר נוסף ידנית לפני שהחבילה הפכה לבעלים של הקבצים האלה):

```bash
ls /etc/apt/sources.list.d/hebrish.sources /etc/apt/keyrings/hebrish-archive-keyring.gpg 2>/dev/null
# remove any that are listed, with consent:
# sudo rm /etc/apt/sources.list.d/hebrish.sources /etc/apt/keyrings/hebrish-archive-keyring.gpg
```

‏`apt` אינו מסיר את נתוני המשתמש. יש לשאול לפני מחיקתם (הפעולה מוחקת את המודלים שהורדו
ואת ההגדרות; ללא sudo):

```bash
grep -o '"models_dir": *"[^"]*"' ~/.config/hebrish/config.json   # a custom model folder, if one was chosen
rm -rf ~/.config/hebrish ~/.local/share/hebrish                  # settings; models, history, state
rm -f ~/.config/autostart/hebrish.desktop                        # only exists if Start at login was on
```

אם הוגדר `models_dir`, גם התיקייה הזו מכילה מודלים; יש למחוק אותה בנפרד אם המשתמש
רוצה. התיקיות של Parlance ‏(`~/.config/parlance`, ‏`~/.local/share/parlance`) שייכות ל-Parlance:
יש להשאיר אותן.
