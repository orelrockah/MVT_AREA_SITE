# MVT AREA SITE — מדריך פרויקט (מקור אמת)

דף נחיתה סטטי של **MVT AREA** (תוסף Revit, ר' `../MVT_AREA/CLAUDE.md`), מוצר של **MAXIOM**.
קובץ אחד (`index.html`), בלי build step. מתפרסם ב-**Cloudflare Pages**, מחובר לריפו הזה —
כל push ל-`main` עולה לאוויר תוך פחות מדקה. דומיין: `mvt.maxiom.co.il`.

**הריפו ציבורי** (`github.com/orelrockah/MVT_AREA_SITE`) — כל מה שנדחף אליו גלוי לכולם.

---

## זרימת הלידים

טופס "השארת פרטים" ב-`index.html` שולח `POST` (כ-`text/plain`, כדי להימנע מ-CORS preflight
ש-Apps Script לא יודע לענות עליו) לכתובת ב-`LEADS_ENDPOINT` (קבוע ב-`index.html`, ~שורה 954).
זו כתובת `/exec` של **Google Apps Script Web App**, שכותב שורה לגיליון "MAXIOM — לידים"
ושולח מייל התראה.

**קוד ה-Apps Script חי מקומית בלבד ב-`apps-script/Leads.gs`** — מודבק ידנית ל-script.google.com,
**לא מסונכרן אוטומטית** (אין clasp). כל עריכה בקובץ המקומי דורשת: להעתיק-להדביק אותה בעורך
Apps Script בענן, ואז **Deploy → Manage deployments → New version** — אחרת הכתובת החיה
ממשיכה להגיש את הקוד הישן. התיקייה `apps-script/` מוצאת מה-git (`.gitignore`) בכוונה,
כי הקוד מכיל כתובות מייל פרטיות והריפו ציבורי.

---

## תקרית: מיילי התראה נכנסו לזבל (08/2026) — נפתרה, מאומת

**הבעיה:** מיילי "ליד חדש" מ-Apps Script נכנסו ל-Junk ב-Outlook. כלל Outlook ידני (לפי נושא)
לא הצליח לחלץ אפילו מיילים קיימים מהזבל, גם כשהורץ ידנית על כל התיבה — כי **כללי Outlook
רצים אחרי סינון הזבל** ולא יכולים לעקוף כשל אימות שולח. הדרך היחידה שכן הייתה עובדת
(Safe Senders) נפסלה בכוונה: זה היה אומר להוסיף את orelrockah@outlook.co.il עצמו לרשימה
הבטוחה — פותח לצמיתות פתח spoofing לכתובת של עצמנו.

**הסיבה השורשית:** ב-[Leads.gs](apps-script/Leads.gs) הפונקציה `notify_` קוראת ל-
`MailApp.sendEmail` **בלי `from` מפורש**. ללא `from`, גוגל שולחת בזהות **בעל הפרויקט**
(Execute as: Me). הפרויקט המקורי container-bound תחת חשבון Google שמחובר עם
**orelrockah@outlook.co.il** (כתובת התחברות לגוגל, לא Gmail אמיתי) — כלומר גוגל שלחה
`From: orelrockah@outlook.co.il` דרך השרתים שלה (`maestro.bounces.google.com`), אבל
דומיין outlook.co.il שייך למיקרוסופט וגוגל לא מוסמכת ב-SPF/DKIM שלו ⇒ כשל אימות ⇒ Outlook
מסווג כזבל אוטומטית.

**ההחלטה (08/2026):** להעביר את זהות ההרצה ל-**orelrockah@gmail.com** — Gmail אמיתי,
מאומת מלידה, ולכן לא נכנס לזבל. זה נעשה כפרויקט **standalone חדש** (לא container-bound),
שנפתח לגיליון הקיים לפי `SHEET_ID` אחרי ששותף עם הג'ימייל כעורך — כדי לא לפצל את היסטוריית
הלידים לגיליון חדש. `NOTIFY_EMAIL` (ה-**to**) נשאר orelrockah@outlook.co.il כרגיל — הוא
מעולם לא היה חלק מהבעיה, רק ה-from.

**סטטוס המיגרציה (08/2026):**
1. ✅ הגיליון "MAXIOM — לידים" שותף עם orelrockah@gmail.com כעורך.
2. ✅ `SHEET_ID` מולא ב-[Leads.gs](apps-script/Leads.gs) (`17SL-9gcba3wVGEfsLfV5Zg07U4_L03leudJU0blEWKs`).
3. ✅ פרויקט Apps Script standalone חדש נוצר ונפרס תחת orelrockah@gmail.com — Execute as: Me,
   Who has access: Anyone. כתובת: `.../AKfycbzlsjNaJmjRHQoGmeepDX8K85BBWxXLXsD7NkNwkiIoZzxXGzNzrPHlXf5Ipua4pcWv/exec`.
4. ✅ `testLead()` הורץ — המייל נחת ב-Inbox של Outlook, לא בזבל. האבחון אושר.
5. ✅ `LEADS_ENDPOINT` ב-`index.html` (~שורה 954) עודכן לכתובת החדשה, נדחף ל-`main`.
6. ⏳ **פתוח:** הדפלוימנט הישן (חשבון outlook, ה-`/exec` הישן) עדיין לא בוטל במפורש —
   כדאי לארכב אותו ב-Manage deployments כדי שלא תישאר נקודת קצה ישנה חיה בטעות.

**למה לא לגעת בקובץ בלבד ולצפות שזה יעבוד:** אין clasp — עריכת `Leads.gs` המקומי לא משפיעה
על שום דבר עד שהתוכן מודבק ידנית בעורך הענן ונפרס Deploy חדש (ר' הערות הראש של הקובץ).

**רעיון ל-backlog (לא מיושם, לא דחוף):** כתובת מייל מקצועית תחת דומיין `maxiom.co.il`
(למשל `leads@maxiom.co.il`) במקום כתובת Gmail גולמית. הדומיין ככל הנראה מנוהל ב-Cloudflare
(כי Pages מחובר אליו) — **חשוב:** Cloudflare Email Routing נותן **רק קבלה/forwarding בחינם**,
לא שליחה מאומתת. לשליחה אמיתית מהדומיין צריך תיבת Workspace/M365 בתשלום, **או** שירות
טרנזקציוני (Resend/SendGrid, tier חינמי מספיק לנפח הזה) + רשומות SPF/DKIM ב-DNS + שינוי
בקוד מ-`MailApp.sendEmail` לקריאת API. לא לבצע בלי בקשה מפורשת מאוראל — זה שינוי גדול יותר
מהמיגרציה לג'ימייל.

---

## כללי עבודה לסוכן

1. אל תדחוף שינוי ל-`LEADS_ENDPOINT` בלי שהכתובת `/exec` החדשה כבר נבדקה ועובדת בפועל —
   שינוי שגוי שובר את קליטת הלידים באתר החי.
2. `apps-script/` נשאר מחוץ ל-git תמיד (יש כתובות מייל פרטיות בקוד, הריפו ציבורי) — אל תסיר
   את השורה מ-`.gitignore` ואל תציע לדחוף את התיקייה.
3. `assets/*` נשמר במטמון Cloudflare/דפדפן לשנה כ-`immutable` — קובץ מוחלף חייב **שם חדש**
   (ר' README.md), לא דריסה של קובץ קיים.
4. אין build step ואין תלויות — `index.html` הוא כל האתר. לבדוק שינויים בפתיחה ישירה בדפדפן.
