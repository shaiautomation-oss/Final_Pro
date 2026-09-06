<div dir="rtl" align="right">

# אפליקציית הניהול — Base44

האפליקציה היא ממשק הניהול של המערכת — מה שבעל העסק פותח בבוקר. בונים אותה
בתיאור בשפה חופשית, בלי לכתוב backend.

---

## עקרון האבטחה — לא לדלג

האפליקציה **אינה** מחזיקה את מפתח ה-Airtable. כל קריאה וכל כתיבה עוברות דרך
ה-Webhook של WF13, ו-n8n הוא זה שמחזיק את ה-credentials ואוכף את הכללים.

```
Base44  ──POST { action, table, payload }──►  n8n (WF13)  ──►  Airtable
        ◄─────────── JSON ──────────────────              ◄──
```

מה שנשמר באפליקציה כ-secret הוא **כתובת ה-Webhook בלבד** — לא טוקן, לא מפתח.

---

## פרומט הבנייה הראשוני

העתיקו את הבלוק הזה לצ'אט הבנייה של Base44:

```
Build an internal admin app for a small Israeli electronics business.
Hebrew UI, RTL layout, ILS currency (₪), dd/mm/yyyy dates.

Screens: Dashboard, Invoices, Leads, Products, Tasks.

Data comes from a single REST endpoint I will provide (an n8n webhook).
Do NOT create your own database and do NOT store any API key in the browser.
Store the endpoint URL as a secret named N8N_WEBHOOK_URL and call it from
the server side.

Every request is a POST to that same URL with a JSON body:
  read:    { "action": "list",   "table": "<Table>", "limit": 100 }
           -> { "ok": true, "records": [ { "id": "...", ...fields } ] }
  create:  { "action": "create", "table": "<Table>", "payload": { ...fields } }
           -> { "ok": true, "record": { ... } }
  update:  { "action": "update", "table": "<Table>", "payload": { "id": "...", ...fields } }
           -> { "ok": true, "record": { ... } }
  chat:    { "action": "chat",   "message": "..." }
           -> { "ok": true, "reply": "..." }

Table names are exactly: Invoices, Leads, Products, Tasks.

Fields:
  Invoices: InvoiceNumber, CustomerId, Amount, VatAmount, Total, Status, PdfUrl, Created
  Leads:    Name, Email, Company, Status, Created
  Products: Name, Category, Price, Description, InStock
  Tasks:    Title, Status

Dashboard shows four KPI cards: total revenue (sum of Total), number of
invoices, open amount (sum of Total where Status is not "Paid"), and leads
by status. Below the cards show the 5 most recent invoices and today's tasks.

Invoices table: columns InvoiceNumber, CustomerId, Amount, VatAmount, Total,
Status, Created, and a "מסמך" column that renders PdfUrl as a link opening
in a new tab. Search box and a Status filter.

Leads table: columns Name, Email, Company, Status, Created. Status filter.
A "ליד חדש" form creates Name, Email, Company and always sets Status to "New".

Products table: Name, Category, Price, Description, InStock. Read-only,
with a Category filter.

Tasks: Title and Status, with a checkbox to toggle Status between
"Open" and "Done".

Add a chat panel that POSTs { "action": "chat", "message": <text> } and
renders the "reply" field from the response. Keep the conversation visible
on screen.
```

---

## שיפורים אחרי הפרומט הראשוני

כך עובדים בפלטפורמות האלה — פרומט אחד גדול, ואז שיפורים קטנים בזה אחר זה.
סדר מומלץ:

1. `הפוך את הדשבורד לשתי עמודות במסך רחב ולעמודה אחת בנייד`
2. `הצג את הסכומים עם הפרדת אלפים וסימן ₪ מימין למספר`
3. `צבע את הסטטוסים: Ready כחול, Issued ירוק, Invalid אדום, Paid אפור`
4. `הוסף סינון לפי סטטוס מעל כל טבלה`
5. `בטבלת החשבוניות, הפוך את PdfUrl לכפתור "פתח מסמך" במקום קישור גולמי`
6. `הוסף כפתור "סמן כשולם" בשורת חשבונית — הוא שולח update עם Status = Paid`
7. `בטופס ליד חדש, ולידציה שכתובת המייל תקינה לפני שליחה`
8. `הצג הודעת שגיאה קריאה כשהתשובה מהשרת היא ok: false`
9. `הוסף מצב טעינה (spinner) לכל טבלה בזמן שהנתונים נטענים`
10. `בפאנל הצ'אט, הצג "מקליד..." בזמן ההמתנה לתשובה`

---

## מה חייב להיות באפליקציה (מינימום נדרש)

| רכיב | תוכן |
|---|---|
| **דשבורד** | הכנסות, מספר חשבוניות, סכום פתוח, לידים לפי סטטוס, משימות להיום |
| **מסכי טבלה** | חשבוניות, לידים, מוצרים, משימות — תצוגה, חיפוש וסינון |
| **טפסים** | יצירת ליד / חשבונית — הכתיבה מפעילה את הטריגרים ב-n8n |
| **קישור למסמכים** | לינק למסמך שהופק ונשמר ב-Google Drive (`PdfUrl`) |
| **צ'אט** | פאנל שמדבר עם הסוכן דרך `action: "chat"` |

---

## נקודה שקל לפספס

כשהאפליקציה יוצרת חשבונית, היא שולחת **רק** `CustomerId` ו-`Amount`.
את `VatAmount`, `Total` ו-`InvoiceNumber` ממלא WF1 תוך דקה, ואת `PdfUrl`
ממלא WF8 תוך דקה נוספת.

כלומר: אחרי יצירת חשבונית, השורה תיראה חלקית למשך רגע. זו התנהגות תקינה.
כדאי להוסיף לאפליקציה רענון אוטומטי כל 30 שניות במסך החשבוניות:

```
רענן את טבלת החשבוניות אוטומטית כל 30 שניות
```

---

## נעילת פלטפורמה

מה שנבנה ב-Base44 אינו זהה למה שנבנה ב-Lovable. **החלק הנייד הוא הטבלאות
וה-Webhook** — הממשק לא. אם תעברו פלטפורמה, שכבת הנתונים והלוגיקה נשארות
בדיוק כפי שהן, ורק את המסכים בונים מחדש מאותו פרומט.

---

## חלופה — `Dashboard.html`

בתיקייה הזו יש גם דשבורד עצמאי בקובץ HTML אחד, שקורא ישירות מ-Airtable
דרך ה-REST API. הוא שימושי לבדיקה מהירה בלי לפתוח את Base44.

**⚠️ הוא דורש שתדביקו בו טוקן Airtable, ולכן הוא מיועד להרצה מקומית בלבד
על המחשב שלכם. אל תעלו אותו לאינטרנט עם הטוקן בפנים** — זה בדיוק מה שהמערכת
בנויה למנוע. האפליקציה האמיתית היא Base44 מול WF13.

</div>
