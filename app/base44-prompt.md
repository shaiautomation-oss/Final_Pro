<div dir="rtl" align="right">

# אפליקציית הניהול

ממשק הניהול של המערכת — מה שבעל העסק פותח בבוקר. בניתי אותו בתיאור בשפה
חופשית, בלי לכתוב backend.

---

## עקרון האבטחה

האפליקציה **אינה** מחזיקה את מפתח ה-Airtable. כל קריאה וכל כתיבה עוברות דרך
ה-Webhook של WF13, ו-n8n הוא זה שמחזיק את ה-credentials ואוכף את הכללים
במקום אחד.

```
האפליקציה  ──POST { action, table, payload }──►  n8n (WF13)  ──►  Airtable
           ◄─────────────── JSON ───────────────              ◄──
```

מה שנשמר באפליקציה כ-secret הוא **כתובת ה-Webhook בלבד** — לא טוקן ולא מפתח.

---

## הפרומט הראשון — בניית המסכים

זה הפרומט שממנו בניתי את השלד: מסכים, טבלאות, טפסים ופאנל צ'אט.

```
Build an internal admin app for a small Israeli electronics business.
Hebrew UI, RTL layout, ILS currency (₪), dd/mm/yyyy dates.

Screens: Dashboard, Invoices, Leads, Products, Tasks.

Data comes from a single REST endpoint I will provide (an n8n webhook).
Do NOT create your own database and do NOT store any API key in the browser.
Store the endpoint URL as a secret named N8N_WEBHOOK_URL and call it from
the server side.

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
renders the "reply" field from the response.
```

---

## הפרומט השני — חיבור ל-n8n

אחרי שהשלד עמד, חיברתי את האפליקציה למערכת האמיתית והסרתי כל מקור נתונים
מקומי:

```
Connect this app to my backend.

There is a single REST endpoint (an n8n webhook). Its URL is stored as a
secret named N8N_WEBHOOK_URL. Call it only from the server side, never
from the browser. Remove any local or mock data source.

Every request is a POST to that same URL with a JSON body:

  { "action": "list",   "table": "<Table>", "limit": 100 }
  { "action": "create", "table": "<Table>", "payload": { ...fields } }
  { "action": "update", "table": "<Table>", "payload": { "id": "...", ...fields } }
  { "action": "chat",   "message": "..." }

Responses:
  list   -> { ok: true, table, records: [ { id, ...fields } ] }
  create -> { ok: true, action: "create", record: { id, ...fields } }
  update -> { ok: true, action: "update", record: { id, ...fields } }
  chat   -> { ok: true, reply: "markdown text" }
  error  -> { ok: false, error: "..." } with HTTP 400

Tables and fields:
  Invoices: InvoiceNumber, CustomerId, Amount, VatAmount, Total, Status, PdfUrl, Created
  Leads:    Name, Email, Company, Status, Created
  Products: Name, Category, Price, Description, InStock
  Tasks:    Title, Status

The chat panel must POST { "action": "chat", "message": <text> } and render
the "reply" field as Markdown - it contains ** bold ** and newlines.

When creating an invoice, send only CustomerId and Amount. The backend fills
InvoiceNumber, VatAmount, Total and PdfUrl within about two minutes, so
refresh the Invoices table automatically every 30 seconds.

In the Invoices table, render PdfUrl as a button labeled "פתח מסמך" that
opens in a new tab.
```

---

## השיפורים שהוספתי אחר כך

כך עובדים בפלטפורמה: פרומט אחד גדול, ואז שיפורים קטנים בזה אחר זה. אלה
השיפורים שהחלתי, לפי הסדר:

1. דשבורד בשתי עמודות במסך רחב, ובעמודה אחת בנייד
2. סכומים עם הפרדת אלפים וסימן ₪ מימין למספר
3. צביעת סטטוסים — `Ready` כחול, `Issued` ירוק, `Invalid` אדום, `Paid` אפור
4. סינון לפי סטטוס מעל כל טבלה
5. `PdfUrl` כפתור "פתח מסמך" במקום קישור גולמי
6. כפתור "סמן כשולם" בשורת חשבונית, ששולח `update` עם `Status = Paid`
7. ולידציה של כתובת המייל בטופס ליד חדש
8. הודעת שגיאה קריאה כשהתשובה מהשרת היא `ok: false`
9. מצב טעינה בכל טבלה בזמן שהנתונים נטענים
10. חיווי "מקליד..." בפאנל הצ'אט בזמן ההמתנה לתשובה

---

## נקודה שקל לפספס

כשהאפליקציה יוצרת חשבונית היא שולחת **רק** `CustomerId` ו-`Amount`.
את `VatAmount`, `Total` ו-`InvoiceNumber` ממלא WF1 תוך כדקה, ואת `PdfUrl`
ממלא WF8 תוך דקה נוספת.

כלומר אחרי יצירת חשבונית השורה נראית חלקית למשך רגע — זו התנהגות תקינה, ולכן
הגדרתי רענון אוטומטי כל 30 שניות במסך החשבוניות.

---

## נעילת פלטפורמה

**החלק הנייד של המערכת הוא הטבלאות וה-Webhook** — לא הממשק. מעבר לפלטפורמת
בנייה אחרת משאיר את שכבת הנתונים והלוגיקה כפי שהן, ודורש רק בנייה מחדש של
המסכים מאותם שני פרומטים.

</div>
