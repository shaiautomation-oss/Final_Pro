<div dir="rtl" align="right">

# צילומי מסך

תיעוד המערכת בפעולה, לפי סדר הזרימה. התמונות מוטמעות ב-[README הראשי](../README.md)
וב-[עמוד התצוגה](../index.html).

## זרימת המערכת

| קובץ | תוכן |
|---|---|
| `01-app-new-invoice.png` | יצירת חשבונית באפליקציית הניהול — לקוח וסכום בלבד |
| `02-airtable-invoice-created.png` | הרשומה נוצרת ב-Airtable דרך ה-Webhook |
| `03-airtable-invoice-processed.png` | WF1 חישב מע"מ 18%, סכום כולל ומספר מסמך רץ |
| `04-invoice-pdf-drive.png` | WF8 הפיק PDF בעברית RTL והעלה לגוגל דרייב |
| `05-app-new-lead.png` | יצירת ליד מהאפליקציה |
| `06-airtable-lead-created.png` | הליד נקלט ב-Airtable בסטטוס New |
| `07-cold-email.png` | המייל הקר שסוכן המכירות ניסח ושלח |
| `08-airtable-lead-contacted.png` | הסטטוס מתעדכן ל-Contacted אחרי השליחה |
| `09-customer-bot.png` | סוכן שירות הלקוחות עונה מתוך המדיניות והקטלוג |
| `10-manager-bot.png` | סוכן המנהל מדווח הכנסות ויתרות |
| `11-manager-alert.png` | התראה אוטומטית כשליד משיב למייל |
| `12-app-dashboard.png` | הדשבורד המרכזי של אפליקציית הניהול |

## קנבסים ב-n8n

| קובץ | תוכן |
|---|---|
| `workflows/wf1.png` | WF1 — אימות מסמכי מס — מע"מ, מספור ותור הפקה |
| `workflows/wf3-create.png` | WF3 — קליטת ליד חדש — מסלול היצירה |
| `workflows/wf3-duplicate.png` | WF3 — אותו ליד בשנית — מסלול הכפילות |
| `workflows/wf4a.png` | WF4a — סוכן מכירות — ניסוח ושליחת מייל קר |
| `workflows/wf4b.png` | WF4b — זיהוי תשובה, עדכון סטטוס והתראה למנהל |
| `workflows/wf5.png` | WF5 — סוכן שירות לקוחות עם שני כלי RAG |
| `workflows/wf6.png` | WF6 — 12 מסמכי מדיניות אל המאגר הווקטורי |
| `workflows/wf7.png` | WF7 — 34 מוצרים אל המאגר הווקטורי |
| `workflows/wf8.png` | WF8 — בניית המסמך, המרה ל-PDF והעלאה לדרייב |
| `workflows/wf9.png` | WF9 — סוכן המנהל עם בקרת הרשאות |
| `workflows/wf13.png` | WF13 — נקודת הכניסה של האפליקציה — ארבע פעולות |

</div>
