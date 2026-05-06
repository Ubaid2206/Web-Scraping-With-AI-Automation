# 🤖 Web Scraping + AI Automation — Make.com Scenarios

Ye project ek **2-part Make.com automation pipeline** hai jo automatically contact form responses ko process karta hai, unki websites scrape karta hai, aur AI se personalized email replies bhejta hai — bilkul human jaisi.

---

## 📋 Overview

Jab koi banda Tally form fill karta hai apni details aur website URL ke saath, toh ye system:

1. Data ko Google Sheets mein save karta hai
2. Unki website ko automatically scrape karta hai (Browse AI)
3. AI Agent se email likhta hai jo website ki details padhke personalized hoti hai
4. Calendar check karke meeting schedule karta hai
5. Gmail se reply send karta hai

---

## 🗂️ Project Structure

```
web scrping ai automation/
├── Part1.txt    → Scenario 1: Form → Sheets → Web Scraping trigger
└── Part2.txt    → Scenario 2: Scraping complete → AI Agent → Email
```

---

## 🔷 Scenario 1 — Form Intake & Web Scraping Trigger

**Make.com Scenario Name:** `Integration Tally`

### Kya karta hai?

Tally form submit hone par user ka data capture karta hai aur unki website scrape karne ka kaam Browse AI ko deta hai.

### Flow Diagram

```
Tally Form Submit
       ↓
Google Sheets mein row add karo
       ↓
BasicRouter (URL format check)
    ↙           ↘
Route 1          Route 2
(URL as-is)      (http:// prefix add karke)
    ↓                ↓
Browse AI        Browse AI
executeTask      executeTask
    ↓                ↓
Sheets update    Sheets update
(task ID save)   (task ID save)
```

### Modules Detail

| Module ID | Service | Kaam |
|-----------|---------|------|
| `1` | `tally:watchNewResponse` | Tally form ka webhook — naya response aane par trigger |
| `4` | `google-sheets:addRow` | Form data Google Sheet (`Scrap Web`) mein add karta hai |
| `5` | `builtin:BasicRouter` | URL format ke hisaab se do routes mein split |
| `6` | `browse-ai:executeTask` | Website scrape karta hai (URL as-is) |
| `7` | `browse-ai:executeTask` | Website scrape karta hai (`http://` prefix ke saath) |
| `8` | `google-sheets:updateRow` | Browse AI task ID column G mein save karta hai |
| `10` | `google-sheets:updateRow` | Browse AI task ID column G mein save karta hai |

### Tally Form Fields

Form mein ye fields hain:

| Field | Type | Sheet Column |
|-------|------|-------------|
| First Name | Text | A |
| Last Name | Text | B |
| Phone Number | Text | C |
| Email | Email | D |
| Website URL (`INPUT_LINK`) | Text | E |
| Your Question / Message | Text | F |
| Browse AI Task ID | Text | G *(auto-filled)* |

### Router Logic

BasicRouter isliye hai kyunki users kabhi kabhi URL mein `http://` likhte hain aur kabhi nahi. Dono routes same Browse AI robot use karte hain lekin ek URL ko as-is bhejta hai aur doosra `http://` prefix add karta hai.

### Setup Requirements

- ✅ Tally account + webhook configured
- ✅ Google Sheets file: **Scrap Web** (Spreadsheet ID: `1t_MVDqDSe9H6jvnZGKJqNI5lwYboUP-NNm_r0rm6PE8`)
- ✅ Browse AI robot ID: `019dce51-796a-745f-8e3b-6cb9da02bfd8`
- ✅ Google account connection in Make.com

---

## 🔶 Scenario 2 — AI Response & Meeting Scheduler

**Make.com Scenario Name:** `New scenario`

### Kya karta hai?

Jab Browse AI website scraping complete kar leta hai, ye scenario trigger hota hai. Phir AI Agent:
- Client ki website analyze karta hai
- Calendar check karke meeting book karta hai ya alternative times suggest karta hai
- Personalized email likhta hai aur Gmail se bhejta hai

### Flow Diagram

```
Browse AI Task Finished (webhook)
          ↓
Google Sheets se matching row fetch karo
(column G = task ID se match)
          ↓
AI Local Agent run karo
    ├── Task 1: Meeting evaluate & schedule
    │     ├── Spam/off-topic? → Skip scheduling
    │     ├── Time available? → Google Meet event banao (30 min)
    │     └── Time unavailable? → 2-3 alternative times suggest karo
    ├── Task 2: Personalized email likho
    │     ├── Website HTML analyze karo
    │     ├── Business details reference karo (name, service, etc.)
    │     └── Conversational, human-like tone rakho
    └── Task 3: Gmail se email bhejo (HTML formatted)
```

### Modules Detail

| Module ID | Service | Kaam |
|-----------|---------|------|
| `1` | `browse-ai:onTaskFinished` | Browse AI task complete hone par trigger |
| `2` | `google-sheets:filterRows` | Column G mein task ID dhundh ke row nikalo |
| `3` | `ai-local-agent:RunLocalAIAgent` | AI Agent run karta hai (model: large) |

### AI Agent — Full Task Breakdown

AI Agent ko ye data milta hai:

```
First Name    → Sheet column A
Last Name     → Sheet column B
Phone Number  → Sheet column C
Email         → Sheet column D
Website HTML  → Sheet column E (scraped content)
Message       → Sheet column F
```

**Task 1 — Evaluate & Schedule:**
- Pehle check karo: kya ye message spam hai ya meeting ki zaroorat hai?
- Agar meeting appropriate hai:
  - Availability: **Monday–Friday, 10:00 AM – 4:00 PM US Central Time**
  - Agar time available: 30-minute Google Meet event create karo + agenda add karo
  - Agar time unavailable: agli 5 working days mein 2-3 alternatives suggest karo

**Task 2 — Personalized Email:**
- Website HTML se business samjho (naam, services, about page)
- Reply mein 1-2 specific website details reference karo
- Meeting confirm ya alternatives mention karo
- 3-5 short paragraphs, conversational tone
- Sign off: `Best wishes, Joe`

**Task 3 — Send Email:**
- Gmail se HTML email bhejo
- Simple, human-like formatting

### AI Agent Rules (Hard Constraints)

```
❌ "I hope this email finds you well" — bilkul nahi likhna
❌ Bullet points, numbered lists, ya em dashes use mat karo
❌ Over-compliment ya sycophantic mat bano
❌ Ye mat batao ke tum AI ho ya HTML analyze kar rahe ho
❌ Availability window ke bahar kabhi schedule mat karo
```

### Setup Requirements

- ✅ Browse AI webhook (`onTaskFinished`) configured
- ✅ Same Google Sheet se connected (`1t_MVDqDSe9H6jvnZGKJqNI5lwYboUP-NNm_r0rm6PE8`)
- ✅ AI Local Agent (large model) configured in Make.com
- ✅ Google Calendar access (AI Agent ke liye)
- ✅ Gmail access (AI Agent ke liye)

---

## 🔗 How Both Scenarios Connect

```
[Scenario 1]                          [Scenario 2]
Tally Form                            Browse AI
Submit ──→ Sheet Row ──→ Scrape ──→   Webhook ──→ Match Row ──→ AI Agent ──→ Email Sent
                          Task ID               by Task ID
                          saved in
                          Column G
```

Dono scenarios **Column G (Browse AI Task ID)** ke zariye linked hain. Scenario 1 task ID save karta hai, Scenario 2 us ID se matching row dhundh ke AI ko deta hai.

---

## 🛠️ Tools & Services Used

| Tool | Purpose |
|------|---------|
| [Make.com](https://make.com) | Automation platform |
| [Tally](https://tally.so) | Form builder (webhook trigger) |
| [Google Sheets](https://sheets.google.com) | Data storage |
| [Browse AI](https://browse.ai) | Website scraping |
| [Make AI Local Agent](https://make.com) | AI task execution |
| Google Calendar | Meeting scheduling |
| Gmail | Email sending |

---

## 🚀 How to Import & Setup

1. **Scenario 1 import karein:**
   - Make.com open karein → New Scenario → Import Blueprint
   - `Part1.txt` file upload karein
   - Apne connections set karein: Tally, Google Sheets, Browse AI

2. **Scenario 2 import karein:**
   - Same steps → `Part2.txt` file upload karein
   - Connections set karein: Browse AI, Google Sheets, AI Agent, Gmail, Google Calendar

3. **Google Sheet setup karein:**
   - Sheet name: `Sheet1`
   - Column headers: `First Name | Last Name | Phone Number | Email | URL | Message | Task ID`

4. **Browse AI Robot configure karein:**
   - Robot ID `019dce51-796a-745f-8e3b-6cb9da02bfd8` apne account mein import/recreate karein
   - Website scraping task set karein (page HTML capture karna hai)

5. **Dono scenarios activate karein** aur Tally form se test submission karein.

---

## ⚠️ Important Notes

- Scenario 1 aur Scenario 2 **alag-alag** Make.com scenarios hain — dono ka apna trigger hai
- Browse AI task ID hi inhe ek doosre se connect karta hai (Column G)
- AI Agent ka model `large` set hai — agar cost concern hai toh `small` try kar sakte hain lekin quality affect hogi
- Email response mein AI apne aap ko AI reveal nahi karta — ye by-design hai

---

*Built with Make.com | Browse AI | Tally | Google Workspace*
