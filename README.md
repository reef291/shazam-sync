# Shazam Sync

כלי דפדפן בודד (קובץ HTML אחד) שמסנכרן שירים שתייגת ב-Shazam אל Apple Music ו/או
Spotify. אין שרת, אין התקנה - הכל רץ בג'אווהסקריפט בתוך הדפדפן שלך.

## איך זה עובד

Shazam לא חושף API ציבורי לתגיות שלך. הדרך היציבה היחידה היא דרך אפל: אם מפעילים
"Sync Library" בהגדרות Shazam, כל תיוג נכנס אוטומטית לפלייליסט בשם **Shazam Library**
בתוך ספריית Apple Music שלך. הדף קורא את הפלייליסט הזה (דרך MusicKit JS), ולכל שיר חדש:

1. מוסיף אותו לספריית Apple Music שלך (My Music) - כי Shazam רק מכניס אותו לפלייליסט,
   לא לספרייה עצמה.
2. מחפש אותו ב-Spotify (לפי ISRC קודם, ואז לפי שם+אמן) ומוסיף אותו לפלייליסט ייעודי
   (או ל-Liked Songs אם לא הוגדר שם פלייליסט).

מצב הסנכרון (אילו שירים כבר טופלו) נשמר ב-localStorage של הדפדפן, כך שכל הרצה
מוסיפה רק שירים חדשים.

## דרישות מוקדמות

### Apple Music
צריך **חשבון Apple Developer בתשלום** (99$/שנה) כדי ליצור MusicKit key - זו דרישה
של אפל עצמה, לא ניתנת לעקיפה:
1. היכנס ל-[developer.apple.com](https://developer.apple.com/account/resources/authkeys/list)
2. Certificates, Identifiers & Profiles → Keys → צור מפתח חדש עם MusicKit מופעל
3. הורד את קובץ ה-`.p8`, ורשום את ה-Key ID ואת ה-Team ID (מופיע בפינה הימנית העליונה)

### Spotify (חינם)
1. היכנס ל-[developer.spotify.com/dashboard](https://developer.spotify.com/dashboard)
2. צור אפליקציה חדשה
3. ב-Redirect URIs הוסף את הכתובת שבה תארח את `index.html` (למשל כתובת GitHub Pages)
4. שמור את ה-Client ID (לא צריך Client Secret - הזרימה היא PKCE, בטוחה גם בצד לקוח)

### Shazam
בהגדרות האפליקציה (Shazam → Settings) ודא ש-"Sync Library" עם Apple Music דלוק.

## איך מריצים

Spotify דורש ש-Redirect URI יהיה כתובת http/https אמיתית (לא ניתן להשתמש בקובץ
מקומי `file://`), אז צריך לארח את `index.html` איפשהו עם https. הכי פשוט וחינמי:
**GitHub Pages**.

```bash
# בתוך תיקיית הפרויקט
git init
git add index.html README.md
git commit -m "Shazam Sync"
git remote add origin https://github.com/<your-username>/shazam-sync.git
git push -u origin main
```

ואז ב-GitHub: Settings → Pages → Deploy from branch → main. הכתובת שתקבל
(`https://<your-username>.github.io/shazam-sync/`) היא זו שרושמים ב-Spotify
כ-Redirect URI, ופותחים בדפדפן כדי להשתמש בכלי.

## שימוש

1. פתח את הדף.
2. במקטע Apple Music: מלא Team ID, Key ID, הדבק את תוכן קובץ ה-`.p8`, ולחץ
   "צור Developer Token" (הטוקן נשמר, המפתח עצמו נמחק מהזיכרון מיד). אז לחץ
   "התחבר ל-Apple Music" ואשר בחלון שנפתח.
3. במקטע Spotify: מלא Client ID ולחץ "התחבר ל-Spotify".
4. לחץ "הרץ סנכרון עכשיו".

הרץ שוב בכל פעם שתרצה לסנכרן שירים חדשים - הכלי מדלג אוטומטית על שירים שכבר טופלו.
ה-developer token של Apple בתוקף ל-6 חודשים וה-Spotify token מתחדש אוטומטית, אז
בדרך כלל לא תצטרך להתחבר מחדש בכל פעם.

## הגדרות אופציונליות

- שם פלייליסט Shazam - אם שונה מ-"Shazam Library" אצלך
- שם פלייליסט Spotify - ריק = הוספה ל-Liked Songs במקום פלייליסט ייעודי

## מגבלות ידועות

- ההתאמה ל-Spotify מבוססת על ISRC כשזמין, ואחרת על השוואת שם+אמן מטושטשת -
  יכולות להיות התאמות שגויות נדירות עבור remixes/covers עם שמות דומים מאוד.
- אם קטלוג Spotify לא כולל שיר מסוים (למשל תוכן בלעדי ל-Apple Music), הוא ידווח
  כ"לא נמצא" בסיכום.
- ה-private key (.p8) מוקלד/מודבק ישירות בדפדפן ומעולם לא נשמר או נשלח לשום מקום -
  רק ה-JWT החתום (developer token) נשמר מקומית ב-localStorage.
