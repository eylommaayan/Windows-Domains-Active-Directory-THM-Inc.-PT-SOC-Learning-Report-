# Windows-Domains-Active-Directory-THM-Inc.-PT-SOC-Learning-Report-
להבין ניהול מרכזי של רשת Windows ארגונית דרך Active Directory, OUs, Groups, GPO, ושיטות אימות (Kerberos/NetNTLM), כולל מבנה מתקדם של Trees/Forests/Trusts

# 🏢 Windows Domains & Active Directory – THM Inc. (Learning Report)

> TryHackMe – Windows Domain / Active Directory (THM Inc.)  
> Focus: AD fundamentals, OU/Groups, GPO, Kerberos vs NetNTLM, Trees/Forests/Trusts  
> Perspective: SOC / Blue Team / PT Basics

---

## 📌 Table of Contents
- [Overview](#-overview)
- [Why Domains?](#-why-domains)
- [Core Concepts](#-core-concepts)
- [Users, Machines, Groups](#-users-machines-groups)
- [OUs vs Groups](#-ous-vs-groups)
- [Managing Users in AD](#-managing-users-in-ad)
- [Delegation](#-delegation)
- [Managing Computers in AD](#-managing-computers-in-ad)
- [Group Policy (GPO)](#-group-policy-gpo)
- [Authentication Methods](#-authentication-methods)
- [Trees, Forests, Trusts](#-trees-forests-trusts)
- [Blue Team / Detection Notes](#-blue-team--detection-notes)
- [Answers (Q&A)](#-answers-qa)
- [Key Takeaways](#-key-takeaways)

---

## 🧾 Overview
במעבדה הזו נכנסתי לתפקיד **IT Admin חדש** בחברה מדומה (THM Inc.) עם דומיין קיים `THM.local`.  
המטרה הייתה להבין ולתרגל ניהול רשת ארגונית דרך:

- **Active Directory (AD)** כמאגר מרכזי
- **Domain Controller (DC)** כשרת הדומיין
- **OUs / Groups** לארגון וניהול הרשאות/מדיניות
- **GPO** להחלת מדיניות אבטחה/ניהול על משתמשים ומחשבים
- **Kerberos / NetNTLM** כשיטות אימות
- מבני ארגון מתקדמים: **Trees / Forests / Trusts**

---

## 🌐 Why Domains?
ברשת קטנה אפשר לנהל מחשב-מחשב, משתמש-משתמש.  
אבל בארגון שגדל (מאות מחשבים/משתמשים בכמה סניפים) ניהול ידני הופך:
- לא עקבי
- איטי
- מסוכן אבטחתית
- מלא שגיאות

לכן משתמשים ב-**Windows Domain** שמרכז:
- יצירת משתמשים
- הרשאות
- מדיניות אבטחה
- הגדרות אחידות לארגון  
הכל במקום אחד: **Active Directory**.

---

## 🧠 Core Concepts
| מושג | מה זה | למה זה חשוב |
|---|---|---|
| Active Directory (AD) | מאגר אובייקטים (users/computers/groups/…) | ניהול מרכזי, הרשאות, אימות |
| Domain Controller (DC) | שרת שמריץ AD DS | מחזיק/מאמת זהויות ומדיניות |
| Security Principals | אובייקטים שמקבלים הרשאות (User/Computer/Group) | בסיס לניהול גישה והרשאות |

---

## 👤 Users, 🖥️ Machines, 👥 Groups
### 👤 Users
שני סוגים עיקריים:
- **People** – עובדים בארגון
- **Services** – חשבונות שירות (IIS/MSSQL וכו’) עם מינימום הרשאות (Least Privilege)

### 🖥️ Machine Accounts
כל מחשב שמצטרף לדומיין יוצר אובייקט ב-AD.  
תבנית שם חשבון מכונה:
- `COMPUTERNAME$`  
דוגמה: `TOM-PC$`

> סיסמאות חשבונות מכונה מתחלפות אוטומטית ובנויות ממחרוזות ארוכות.

### 👥 Security Groups (Permissions)
קבוצות מאפשרות להקצות הרשאות למשאבים בצורה נוחה.  
קבוצות חשובות:
- **Domain Admins** – שליטה מלאה בדומיין
- **Server Operators** – ניהול DC בלי שינוי קבוצות מנהלים
- **Backup Operators** – גישה לקבצים גם בלי הרשאות (רגיש!)
- **Account Operators** – יצירה/שינוי חשבונות
- **Domain Users / Domain Computers / Domain Controllers** – קבוצות בסיס

---

## 🗂️ OUs vs Groups
### ✅ OUs (Policies)
- מיועדים להחלת **מדיניות (GPO)** על משתמשים/מחשבים
- משתמש/מחשב נמצא **ב-OU אחד** בכל רגע

### ✅ Groups (Permissions)
- מיועדים להרשאות למשאבים (תיקיות/מדפסות/שירותים)
- משתמש יכול להיות ב-**הרבה קבוצות**

---

## 👥 Managing Users in AD
### 1) מחיקת OU מיותר (Accidental Deletion Protection)
ברירת מחדל: OU מוגן ממחיקה בטעות.

**איך מבטלים?**
1. `View` → ✅ `Advanced Features`
2. Right click על ה-OU → `Properties`
3. Tab: `Object` → הסרת ✅ `Protect object from accidental deletion`
4. מחיקה מחדש

**למה?**  
מניעת טעויות אנוש שעלולות למחוק מחלקה שלמה + משתמשים/אובייקטים.

### 2) התאמת משתמשים לתרשים ארגוני
יצירה/מחיקה כדי להתאים ל-Org Chart.

**למה זה חשוב אבטחתית?**
- חשבונות לא פעילים = סיכון (Stale Accounts)
- הרשאות לא נכונות = גישה לא מורשית

---

## 🧷 Delegation
Delegation = מתן הרשאה ממוקדת למשתמש על OU ספציפי (בלי להפוך אותו ל-Admin).

בדוגמה: **Phillip** (IT Support) קיבל הרשאה לבצע **Reset Password** למחלקת Sales.

### PowerShell (כ-Phillip)
```powershell
Set-ADAccountPassword sophie -Reset -NewPassword (Read-Host -AsSecureString -Prompt 'New Password') -Verbose
Set-ADUser -ChangePasswordAtLogon $true -Identity sophie -Verbose


להלן **דוח מסכם ומלמד** (בסגנון PT/SOC) שתוכל להדביק ל־**README ב-GitHub**.
כולל: מה עושים בכל חלק, למה עושים, איך זה עובד בפועל, וגם **התשובות לשאלות** שמופיעות אצלך בטקסט.

---

# 🏢 Windows Domains & Active Directory – THM Inc. (PT/SOC Learning Report)

**Platform:** TryHackMe
**Module/Room:** Windows Domain / Active Directory (THM Inc.)
**Role-play:** IT Admin חדש שמסדר דומיין קיים `THM.local`
**Goal:** להבין ניהול מרכזי של רשת Windows ארגונית דרך **Active Directory**, **OUs**, **Groups**, **GPO**, ושיטות אימות (**Kerberos/NetNTLM**), כולל מבנה מתקדם של **Trees/Forests/Trusts**.

---

## 🎯 למה בכלל צריך Domain?

כשיש מעט מחשבים, אפשר לנהל ידנית כל מחשב: משתמשים, הרשאות, הגדרות.
אבל כשארגון גדל (עשרות/מאות מחשבים ויוזרים בכמה סניפים), ניהול ידני נהיה:

* איטי מאוד
* מלא טעויות
* לא עקבי
* מסוכן אבטחתית

**Windows Domain** פותר את זה על ידי ניהול מרכזי:
כל המשתמשים והמחשבים מנוהלים במקום אחד – **Active Directory** – והשרת שמריץ אותו נקרא **Domain Controller (DC)**.

---

## 🧠 מושגים מרכזיים (הבסיס של החדר)

### 1) Active Directory (AD)

מאגר מרכזי שמכיל “אובייקטים” של הרשת:
משתמשים, קבוצות, מחשבים, מדפסות, שיתופים ועוד.

### 2) Domain Controller (DC)

השרת שמחזיק ומריץ את שירותי AD.
כל אימות (Login) לרוב עובר דרכו.

### 3) Security Principals

אובייקטים שיכולים “להזדהות” ולקבל הרשאות:

* Users
* Computers (Machine Accounts)
* Groups

---

## 👤 Users / 🖥️ Machines / 👥 Groups – איך זה עובד ולמה זה חשוב

### 👤 Users

מייצגים:

* אנשים (Employees)
* שירותים (Service Accounts) – כמו IIS / MSSQL
  המסר כאן חשוב ל-Blue Team: שירות צריך מינימום הרשאות (Least Privilege).

### 🖥️ Machines (Computer Objects)

כל מחשב שמצטרף לדומיין יוצר אובייקט ב-AD.
יש לו גם “חשבון מכונה” (Machine Account) עם סיסמה ארוכה שמתחלפת אוטומטית.

✅ תבנית שם: `COMPUTERNAME$`
לדוגמה: `TOM-PC$`

### 👥 Security Groups

הסיבה לקבוצות: לא מנהלים הרשאות אחד־אחד, אלא לקבוצה.
משתמש יכול להיות בכמה קבוצות.

קבוצות חשובות:

* **Domain Admins** – שליטה מלאה על הדומיין
* **Server Operators** – ניהול DCים בלי לשנות חברויות בקבוצות מנהלים
* **Backup Operators** – גישה לקבצים גם בלי הרשאות (מסוכן אם מנוצל)
* **Account Operators** – יצירה/שינוי חשבונות
* **Domain Users / Domain Computers / Domain Controllers** – קבוצות בסיס

---

## 🗂️ OU (Organizational Units) מול Groups – מה ההבדל?

זה אחד החלקים הכי חשובים בחדר.

### ✅ OU = ניהול מדיניות (Policies)

משתמש/מחשב נמצא ב-OU אחד בלבד.
OU מאפשר להחיל **GPO** בצורה מסודרת לפי מחלקות/סוגי מחשבים.

### ✅ Groups = הרשאות למשאבים (Permissions)

משתמש יכול להיות בכמה Groups.
משמשים לתת גישה לתיקייה משותפת, מדפסת, שירות, וכו’.

---

## 🧰 ניהול AD בפועל (Active Directory Users and Computers)

ב־DC פותחים את:
**Active Directory Users and Computers**
רואים היררכיה של:

* Domain
* OUs
* Users/Computers/Groups

בתרגיל שלך: קיימת OU ראשית בשם **THM**, ומתחתיה מחלקות:
IT / Management / Marketing / R&D / Sales

---

## 🧹 Task: Managing Users in AD – מה עושים ולמה?

### 1) מחיקת OU מיותר

ניסיון למחוק OU נותן שגיאה, כי:
✅ ברירת מחדל: OU מוגנים ממחיקה בטעות.

**מה עושים?**
View → ✅ Advanced Features
ואז ב-OU → Properties → Object → מורידים:
✅ “Protect object from accidental deletion”

**למה זה חשוב?**
מונע טעויות אנוש (Human Error) שמוחקות מחלקה שלמה.

---

### 2) התאמת משתמשים לתרשים ארגוני

מוחקים/יוצרים משתמשים כך שה-AD יתאים ל-Org Chart.

**למה זה חשוב?**

* משתמשים לא רלוונטיים = סיכון אבטחתי (Stale Accounts)
* ניהול הרשאות לא נכון = גישה לא מורשית

---

## 🧷 Delegation – העברת סמכויות בצורה בטוחה

Delegation = לתת למשתמש הרשאה לבצע משימה על OU ספציפי, בלי להפוך אותו ל-Domain Admin.

בדוגמה:
Phillip (IT Support) מקבל יכולת **Reset Password** למחלקת Sales.

**למה זה חשוב?**

* Helpdesk יכול לעבוד בלי מנהל דומיין
* מפחית שימוש בהרשאות גבוהות
* מצמצם נזק במקרה של פריצה לחשבון

### בדיקת Delegation עם PowerShell

כ-Phillip מבצעים:

```powershell
Set-ADAccountPassword sophie -Reset -NewPassword (Read-Host -AsSecureString -Prompt 'New Password') -Verbose
Set-ADUser -ChangePasswordAtLogon $true -Identity sophie -Verbose
```

**למה דרך PowerShell?**
כי אין לו הרשאות לפתוח ADUC, אבל יש לו הרשאות פעולה ממוקדות.

---

## 🖥️ Task: Managing Computers in AD – סידור מכונות לפי שימוש

ברירת מחדל: מחשבים נכנסים לקונטיינר “Computers”.
זה בעייתי כי שרתים ותחנות עבודה צריכים **מדיניות שונה**.

יוצרים OUs:

* Workstations
* Servers
  (DC כבר ב-Domain Controllers)

ואז מעבירים:

* PCs/Laptops → Workstations
* Servers → Servers

**למה זה חשוב?**

* מאפשר להחיל GPO שונים על שרתים מול תחנות
* בסיס לאבטחת Domain (hardening)

---

## 📜 Task: Group Policies (GPO) – לב הארגון

GPO = אוסף חוקים/הגדרות שמוחלים דרך OU.

### איך זה מופץ?

דרך שיתוף רשת בשם:
✅ **SYSVOL**
מיקום ב-DC:
`C:\Windows\SYSVOL\sysvol\`

### עדכון מדיניות מיידי

במחשב יעד:

```powershell
gpupdate /force
```

---

## ✅ GPO שביצעתם בחדר – מה ולמה

### 1) חסימת Control Panel לכל מי שלא IT

יוצרים GPO: **Restrict Control Panel Access**
ומפעילים:
User Configuration → Policies → Administrative Templates → Control Panel
✅ Prohibit access to Control Panel and PC settings

מקשרים ל־OUs:
Marketing / Management / Sales

**למה?**

* מונע שינוי הגדרות אבטחה/רשת ע”י משתמשים
* מצמצם “Shadow IT”
* מקטין סיכון של Misconfigurations

---

### 2) נעילה אוטומטית אחרי 5 דקות

יוצרים GPO: **Auto Lock Screen**
Computer Configurations → Security Settings
Machine inactivity limit = 5 min

מקשרים ל־Root domain (או ל-Workstations/Servers/DCs)

**למה?**

* מגן מפני Session Hijacking
* מונע חשיפה כשעובד עוזב מחשב פתוח

---

## 🔐 Authentication Methods – Kerberos vs NetNTLM

### Kerberos (ברירת מחדל היום)

עובד עם “Tickets”.

* המשתמש מקבל **TGT** (Ticket Granting Ticket)
* עם ה-TGT הוא מבקש **TGS** לשירות ספציפי
* השירות מאמת את ה-TGS ומאפשר גישה

**למה זה טוב?**

* לא שולחים סיסמה בכל פעם
* יעיל ומאובטח יותר

### NetNTLM (Legacy)

Challenge-Response:

1. שרת שולח Challenge
2. לקוח מחזיר Response שמבוסס על Hash + Challenge
3. DC מאמת

✅ הסיסמה לא עוברת ברשת (וגם לא ה־hash עצמו ישירות בצורה גלויה)

---

## 🌳 Trees / 🌲 Forests / 🤝 Trusts – מבנה ארגוני מתקדם

### Tree

קבוצת Domains עם אותו Namespace
דוגמה:
`thm.local`
`uk.thm.local`
`us.thm.local`

### Forest

כמה Trees עם Namespaces שונים באותה “משפחה”
דוגמה: THM + MHT

### Trust Relationship

כדי שמשתמש מדומיין אחד יקבל גישה למשאב בדומיין אחר.

**חשוב:** Trust לא נותן גישה אוטומטית לכל דבר – רק מאפשר לאשר גישה אם הוגדר.

---

# ✅ תשובות (לשילוב בסוף README / Q&A)

* Credentials stored in central repository: **Active Directory**
* Server running AD services: **Domain Controller**
* Group that administrates all computers/resources: **Domain Admins**
* Machine account for TOM-PC: **TOM-PC$**
* Container for consistent policies by department: **Organizational Units (OUs)**
* Flag on Sophie’s desktop: **THM{thanks_for_contacting_support}**
* Granting privileges over an OU: **Delegation**
* Devices in Workstations OU after organizing: **7**
* Separate OUs for Servers/Workstations recommended: **yay**
* Network share for GPO distribution: **SYSVOL**
* Can GPO apply to users and computers?: **yay**
* NetNTLM preferred by default today?: **nay**
* Ticket to request TGS: **Ticket Granting Ticket (TGT)**
* Is password transmitted in NetNTLM?: **nay**
* Group of domains with same namespace: **Tree**
* Needed for cross-domain resource access: **Trust Relationship**

---

## 🧩 איך זה מתחבר ל-PT / SOC בפועל (חשוב ל-GitHub)

* **AD הוא יעד עיקרי לתוקפים** כי ה-DC מחזיק Hashים של כל הארגון.
* טעויות כמו:

  * Delegation מוגזם
  * GPO לא נכון
  * סיסמאות חלשות
  * OU מבולגן
    → מאפשרות תקיפה ושדרוג הרשאות (Privilege Escalation).

---

אם תרצה, אני יכול גם:

1. לכתוב לך את זה בפורמט **README.md מוכן** כולל כותרות, טבלת תוכן ותגיות.
2. להוסיף “Key Takeaways” ו-“Blue Team Detection Notes” כדי שזה ייראה עוד יותר מקצועי ב-GitHub.

.
