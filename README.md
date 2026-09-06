<div align="center">

<img src="assets/banner.svg" alt="ERP-AI — מערכת ניהול עסק חכמה על n8n" width="100%">

<br>

[![n8n](https://img.shields.io/badge/n8n-10%20workflows-EA4B71?style=for-the-badge&logo=n8n&logoColor=white)](docs/03-workflows.md)
[![Airtable](https://img.shields.io/badge/Airtable-4%20tables-18BFFF?style=for-the-badge&logo=airtable&logoColor=white)](docs/02-data-schema.md)
[![OpenAI](https://img.shields.io/badge/OpenAI-3%20agents-412991?style=for-the-badge&logo=openai&logoColor=white)](#שלושת-הסוכנים)
[![Telegram](https://img.shields.io/badge/Telegram-2%20bots-26A5E4?style=for-the-badge&logo=telegram&logoColor=white)](docs/05-telegram-bots.md)

[![No Code](https://img.shields.io/badge/code_nodes-0-0d9488?style=flat-square)](workflows)
[![RAG](https://img.shields.io/badge/RAG-68%20chunks-1f4e79?style=flat-square)](#שלושת-הסוכנים)
[![Hebrew](https://img.shields.io/badge/UI-Hebrew%20RTL-6b7c93?style=flat-square)](#מה-בניתי)
[![VAT](https://img.shields.io/badge/VAT-18%25-b26a00?style=flat-square)](docs/06-tax-and-vat.md)

**[🌐 לתצוגת הפרויקט](https://shaiautomation-oss.github.io/Final_Pro/)**

</div>

<div dir="rtl" align="right">

---

## מה בניתי

מערכת ERP לעסק אלקטרוניקה ישראלי, שבה **סוכני בינה מלאכותית ותהליכים אוטומטיים
עושים את רוב העבודה**: עונים ללקוחות בטלגרם, שולחים מיילי מכירות, מחשבים מע"מ,
מפיקים מסמכי חשבונית ומנתחים את העסק — בלי שורת קוד אחת.

| שכבה | במה מימשתי | תפקיד |
|---|---|---|
| **נתונים** | Airtable — בסיס `FINAL` | 4 טבלאות, מקור האמת לכל הרשומות |
| **לוגיקה ואוטומציה** | n8n Cloud | 10 תהליכים + 3 סוכני AI + מאגר וקטורי |
| **ממשק** | אפליקציית ווב | דשבורד, טבלאות, טפסים וצ'אט |

### הארכיטקטורה

```mermaid
flowchart RL
  subgraph IN["נכנס"]
    direction TB
    T1["בוט טלגרם מנהל"]
    T2["בוט טלגרם לקוחות"]
    T3["Gmail — דואר נכנס"]
    T4["לוחות זמנים"]
    T5["אפליקציית הניהול"]
  end

  subgraph CORE["n8n — 10 תהליכים"]
    direction TB
    AG["3 סוכני AI"]
    VS["מאגר וקטורי — RAG"]
    WF["אוטומציות<br/>מעמ · מסמכים · לידים"]
    AG <--> VS
    AG --- WF
  end

  subgraph OUT["יוצא"]
    direction TB
    AT["Airtable<br/>4 טבלאות"]
    GD["Google Drive<br/>מסמכי חשבונית"]
    GS["Gmail<br/>מיילי מכירות"]
  end

  T1 --> AG
  T2 --> AG
  T3 --> WF
  T4 --> WF
  T5 --> WF

  WF --> AT
  WF --> GD
  WF --> GS
  AG --> AT

  classDef in fill:#e8f4fd,stroke:#5fa4dd,color:#0f1b2d
  classDef core fill:#e6f7f4,stroke:#0d9488,color:#0f1b2d
  classDef out fill:#f3f0fb,stroke:#7c6bd4,color:#0f1b2d
  class T1,T2,T3,T4,T5 in
  class AG,VS,WF core
  class AT,GD,GS out
```

במרכז יושב n8n. אליו נכנסים אירועים — הודעות טלגרם, מיילים, לוחות זמנים
וקריאות מהאפליקציה — וממנו יוצאות פעולות לשירותים החיצוניים. האפליקציה אינה
ניגשת ל-Airtable ישירות אלא רק דרך n8n.

---

## עשרת התהליכים

| # | מה עושה | טריגר |
|---|---|---|
| **WF1** | אימות מסמכי מס והכנסה לתור הפקה | חשבונית חדשה |
| **WF3** | קליטת לידים וסינון כפילויות | Webhook |
| **WF4a** | סוכן מכירות — מיילים קרים | כל 3 שעות |
| **WF4b** | סוכן מכירות — זיהוי תשובות | כל 30 דקות |
| **WF5** | סוכן שירות לקוחות | בוט טלגרם #2 |
| **WF6** | מדיניות ← מאגר וקטורי | ידני |
| **WF7** | מוצרים ← מאגר וקטורי | ידני |
| **WF8** | הפקת חשבונית PDF והעלאה לדרייב | כל דקה |
| **WF9** | סוכן המנהל | בוט טלגרם #1 |
| **WF13** | Webhook לאפליקציה | POST מהאפליקציה |

> [!NOTE]
> **98 צמתים, אפס צמתי קוד.** מה שבפרויקטים אחרים נכתב בקוד מימשתי בצמתים
> מוכנים: חישוב מע"מ ב-Edit Fields, ספירת כפילויות ב-Summarize, סיכום הכנסות
> ב-Summarize, החלטות ב-IF, ניתוב ב-Switch והפקת קובץ ב-Convert to File.

**הכול בעברית:** שמות התהליכים, שמות 84 הצמתים, תיאור מתחת לכל צומת, פתקי
הסבר על הקנבס, הנחיות שלושת הסוכנים ותבנית החשבונית.

---

## שלושת הסוכנים

| סוכן | איפה | מה יודע | מגבלה שהגדרתי |
|---|---|---|---|
| **מנהל** | בוט טלגרם #1 | הכנסות, מספר חשבוניות, יתרות | בלי כלים · עד 100 חשבוניות · רק לבעלים |
| **שירות לקוחות** | בוט טלגרם #2 | מדיניות וקטלוג מוצרים דרך RAG | לא מאשר החזרים ולא נותן ייעוץ משפטי |
| **מכירות** | ברקע | מיילים קרים וזיהוי תשובות | ליד אחד בכל הרצה |

<details>
<summary><b>איך RAG עובד כאן</b></summary>

<br>

12 מסמכי מדיניות נחתכים ל-**68 קטעים** ו-34 מוצרים נטענים למאגר וקטורי בזיכרון.
כשלקוח שואל שאלה, השאלה הופכת לווקטור, המערכת שולפת את הקטעים הדומים ביותר
במשמעות, ורק אז הסוכן מנסח תשובה — מבוססת עליהם בלבד.

כך הסוכן אינו ממציא: כשהמידע לא נמצא בכלים הוא אומר זאת במפורש ומפנה לנציג אנושי.

מודל ה-embeddings זהה בכל ארבעת המקומות שבהם הוא מופיע — WF6, WF7, WF5 ו-WF13.

</details>

<details>
<summary><b>מחזור החשבונית מקצה לקצה</b></summary>

<br>

```
רשומה נוצרת  →  WF1 (עד דקה)  →  WF8 (עד דקה)  →  ידני
  CustomerId       מחשב מע"מ        מפיק מסמך        סימון
  Amount           ומספר מסמך       ומעלה לדרייב     תשלום
      ↓                ↓                 ↓             ↓
  (ללא סטטוס)       Ready            Issued          Paid
```

בעל העסק ממלא שני שדות בלבד — לקוח וסכום. מע"מ 18%, סכום כולל, מספר מסמך רץ
וקישור למסמך בדרייב מתמלאים לבד.

</details>

---

## המערכת בפעולה

המערכת רצה מקצה לקצה. אלה צילומי המסך, לפי סדר הזרימה.

<div align="center">
<img src="screenshots/01-app-new-invoice.png" alt="יצירת חשבונית באפליקציית הניהול — לקוח וסכום בלבד" width="85%">

*יצירת חשבונית באפליקציית הניהול — לקוח וסכום בלבד*
<img src="screenshots/02-airtable-invoice-created.png" alt="הרשומה נוצרת ב-Airtable דרך ה-Webhook" width="85%">

*הרשומה נוצרת ב-Airtable דרך ה-Webhook*
<img src="screenshots/03-airtable-invoice-processed.png" alt="WF1 חישב מע"מ 18%, סכום כולל ומספר מסמך רץ" width="85%">

*WF1 חישב מע"מ 18%, סכום כולל ומספר מסמך רץ*
<img src="screenshots/04-invoice-pdf-drive.png" alt="WF8 הפיק PDF בעברית RTL והעלה לגוגל דרייב" width="85%">

*WF8 הפיק PDF בעברית RTL והעלה לגוגל דרייב*
<img src="screenshots/05-app-new-lead.png" alt="יצירת ליד מהאפליקציה" width="85%">

*יצירת ליד מהאפליקציה*
<img src="screenshots/06-airtable-lead-created.png" alt="הליד נקלט ב-Airtable בסטטוס New" width="85%">

*הליד נקלט ב-Airtable בסטטוס New*
<img src="screenshots/07-cold-email.png" alt="המייל הקר שסוכן המכירות ניסח ושלח" width="85%">

*המייל הקר שסוכן המכירות ניסח ושלח*
<img src="screenshots/08-airtable-lead-contacted.png" alt="הסטטוס מתעדכן ל-Contacted אחרי השליחה" width="85%">

*הסטטוס מתעדכן ל-Contacted אחרי השליחה*
<img src="screenshots/09-customer-bot.png" alt="סוכן שירות הלקוחות עונה מתוך המדיניות והקטלוג" width="85%">

*סוכן שירות הלקוחות עונה מתוך המדיניות והקטלוג*
<img src="screenshots/10-manager-bot.png" alt="סוכן המנהל מדווח הכנסות ויתרות" width="85%">

*סוכן המנהל מדווח הכנסות ויתרות*
<img src="screenshots/11-manager-alert.png" alt="התראה אוטומטית כשליד משיב למייל" width="85%">

*התראה אוטומטית כשליד משיב למייל*
<img src="screenshots/12-app-dashboard.png" alt="הדשבורד המרכזי של אפליקציית הניהול" width="85%">

*הדשבורד המרכזי של אפליקציית הניהול*
</div>

<details>
<summary><b>אחד-עשר הקנבסים ב-n8n</b></summary>

<div align="center">

<img src="screenshots/workflows/wf1.png" alt="WF1 — אימות מסמכי מס — מע"מ, מספור ותור הפקה" width="92%">

**WF1** — אימות מסמכי מס — מע"מ, מספור ותור הפקה

<img src="screenshots/workflows/wf3-create.png" alt="WF3 — קליטת ליד חדש — מסלול היצירה" width="92%">

**WF3** — קליטת ליד חדש — מסלול היצירה

<img src="screenshots/workflows/wf3-duplicate.png" alt="WF3 — אותו ליד בשנית — מסלול הכפילות" width="92%">

**WF3** — אותו ליד בשנית — מסלול הכפילות

<img src="screenshots/workflows/wf4a.png" alt="WF4a — סוכן מכירות — ניסוח ושליחת מייל קר" width="92%">

**WF4a** — סוכן מכירות — ניסוח ושליחת מייל קר

<img src="screenshots/workflows/wf4b.png" alt="WF4b — זיהוי תשובה, עדכון סטטוס והתראה למנהל" width="92%">

**WF4b** — זיהוי תשובה, עדכון סטטוס והתראה למנהל

<img src="screenshots/workflows/wf5.png" alt="WF5 — סוכן שירות לקוחות עם שני כלי RAG" width="92%">

**WF5** — סוכן שירות לקוחות עם שני כלי RAG

<img src="screenshots/workflows/wf6.png" alt="WF6 — 12 מסמכי מדיניות אל המאגר הווקטורי" width="92%">

**WF6** — 12 מסמכי מדיניות אל המאגר הווקטורי

<img src="screenshots/workflows/wf7.png" alt="WF7 — 34 מוצרים אל המאגר הווקטורי" width="92%">

**WF7** — 34 מוצרים אל המאגר הווקטורי

<img src="screenshots/workflows/wf8.png" alt="WF8 — בניית המסמך, המרה ל-PDF והעלאה לדרייב" width="92%">

**WF8** — בניית המסמך, המרה ל-PDF והעלאה לדרייב

<img src="screenshots/workflows/wf9.png" alt="WF9 — סוכן המנהל עם בקרת הרשאות" width="92%">

**WF9** — סוכן המנהל עם בקרת הרשאות

<img src="screenshots/workflows/wf13.png" alt="WF13 — נקודת הכניסה של האפליקציה — ארבע פעולות" width="92%">

**WF13** — נקודת הכניסה של האפליקציה — ארבע פעולות

</div>

</details>

---

## מפת התיקייה

```
ERP-AI/
├── index.html                     עמוד התצוגה של הפרויקט
├── workflows/                     10 קובצי n8n מוכנים לייבוא
├── data/
│   ├── policies/                  12 מסמכי מדיניות — מקור ה-RAG
│   └── products/products.csv      34 מוצרים — מקור ה-RAG
├── app/
│   ├── base44-prompt.md           שני הפרומטים שמהם נבנתה האפליקציה
│   └── Dashboard.html             דשבורד עצמאי להרצה מקומית
├── screenshots/                   תיעוד ויזואלי
└── docs/                          6 מסמכי תיעוד
```

---

## התיעוד

| מסמך | תוכן |
|---|---|
| [כך הקמתי את המערכת](docs/01-setup.md) | שמונה שלבים, מהחיבורים ועד האפליקציה |
| [סכימת הנתונים](docs/02-data-schema.md) | ארבע טבלאות ומילון סטטוסים |
| [מפרט האוטומציות](docs/03-workflows.md) | צומת אחר צומת, עשרת התהליכים |
| [שני הבוטים](docs/05-telegram-bots.md) | הפרדת הרשאות ובקרת גישה |
| [מע"מ ומסמכי מס](docs/06-tax-and-vat.md) | 18%, מספור רץ ותבנית החשבונית |
| [הבדיקות שביצעתי](docs/07-end-to-end-tests.md) | עשר בדיקות קבלה עם תוצאות |

---

## החלטות תכנון ומגבלות מכוונות

בחרתי לשמור על פשטות במקומות שבהם פתרון מלא היה דורש תשתית נוספת:

- **המאגר הווקטורי יושב בזיכרון** של n8n ונבנה מחדש בהרצת WF6 + WF7. כך אין
  צורך במסד וקטורי חיצוני.
- **החשבונית מופקת כ-PDF** דרך PDFShift, שמרנדר את התבנית בדפדפן אמיתי — כך המסמך זהה לתצוגה.
- **קשרים בין טבלאות הם מפתחות זרים כטקסט** (למשל `CUST-0001`) ולא קישורי
  Airtable — פשוט יותר לכתיבה מ-n8n ולהצגה באפליקציה.
- **WF4a שולח ליד אחד בכל הרצה**, כדי שטעות בהגדרה לא תשלח עשרות מיילים.
- **לסוכן המנהל אין כלים** והוא רואה עד 100 חשבוניות. כל המספרים מחושבים
  לפניו בצמתי Summarize, כך שאין לו מקום להמציא נתונים.
- **מספור החשבוניות עלול להתנגש** אם שתי חשבוניות נוצרות באותה דקה. פתרון
  מלא היה דורש טבלת מונים עם נעילה.

---

## אבטחה

> [!IMPORTANT]
> מפתח ה-Airtable אינו נמצא באפליקציה ואינו נמצא בקוד שרץ בדפדפן. הוא יושב רק
> ב-Credential של n8n, והאפליקציה מדברת עם המערכת דרך ה-Webhook של WF13 בלבד —
> כך שכל הכללים נאכפים במקום אחד.

הפרדתי בין שני בוטי טלגרם: בוט המנהל מוגן בתנאי שמשווה את מזהה הצ'אט לזה של
הבעלים, וכל פנייה אחרת מקבלת סירוב בלי נתונים.

`app/Dashboard.html` הוא היוצא מן הכלל: הוא קורא ישירות מ-Airtable ודורש טוקן
קריאה, ולכן מיועד להרצה מקומית בלבד. הטוקן נשמר ב-`localStorage` של הדפדפן
ואינו נכתב לשום קובץ — בריפו עצמו אין שום מפתח, טוקן או סוד.

</div>
