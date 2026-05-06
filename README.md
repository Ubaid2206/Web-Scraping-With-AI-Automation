# 🤖 Web Scraping + AI Automation — Make.com Scenarios

This project is a **2-part Make.com automation pipeline** that automatically processes contact form submissions, scrapes the sender's website, and sends a personalized AI-generated email reply — all without any manual effort.

---

## 📋 Overview

When someone fills out a Tally form with their details and website URL, this system:

1. Saves their data to Google Sheets
2. Automatically scrapes their website using Browse AI
3. Uses an AI Agent to craft a personalized email based on their website content
4. Checks the calendar and schedules a meeting
5. Sends the reply via Gmail

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

### What it does

Captures user data when a Tally form is submitted and hands off their website URL to Browse AI for scraping.

### Flow Diagram

```
Tally Form Submitted
        ↓
Add row to Google Sheets
        ↓
BasicRouter (URL format check)
    ↙              ↘
Route 1            Route 2
(URL as-is)        (prepend http://)
    ↓                  ↓
Browse AI          Browse AI
executeTask        executeTask
    ↓                  ↓
Update Sheet       Update Sheet
(save task ID)     (save task ID)
```

### Module Details

| Module ID | Service | Description |
|-----------|---------|-------------|
| `1` | `tally:watchNewResponse` | Webhook trigger — fires when a new Tally form response is received |
| `4` | `google-sheets:addRow` | Adds the form data as a new row in the `Scrap Web` Google Sheet |
| `5` | `builtin:BasicRouter` | Splits into two routes based on URL format |
| `6` | `browse-ai:executeTask` | Scrapes the website using the URL as-is |
| `7` | `browse-ai:executeTask` | Scrapes the website with an `http://` prefix added |
| `8` | `google-sheets:updateRow` | Saves the Browse AI task ID to column G (Route 1) |
| `10` | `google-sheets:updateRow` | Saves the Browse AI task ID to column G (Route 2) |

### Tally Form Fields

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

The BasicRouter exists because users sometimes include `http://` in their URL and sometimes don't. Both routes use the same Browse AI robot — one sends the URL as entered, the other prepends `http://` to ensure the scraper can access it.

### Setup Requirements

- ✅ Tally account with webhook configured
- ✅ Google Sheets file: **Scrap Web** (Spreadsheet ID: `1t_MVDqDSe9H6jvnZGKJqNI5lwYboUP-NNm_r0rm6PE8`)
- ✅ Browse AI robot ID: `019dce51-796a-745f-8e3b-6cb9da02bfd8`
- ✅ Google account connected in Make.com

---

## 🔶 Scenario 2 — AI Response & Meeting Scheduler

**Make.com Scenario Name:** `New scenario`

### What it does

Triggered when Browse AI finishes scraping a website. The AI Agent then analyzes the scraped content, schedules a meeting on the calendar, writes a personalized email, and sends it via Gmail.

### Flow Diagram

```
Browse AI Task Finished (webhook)
            ↓
Fetch matching row from Google Sheets
(match column G = task ID)
            ↓
Run AI Local Agent
    ├── Task 1: Evaluate & Schedule
    │     ├── Spam / off-topic? → Skip scheduling
    │     ├── Time available? → Create 30-min Google Meet event
    │     └── Time unavailable? → Suggest 2-3 alternative slots
    ├── Task 2: Write Personalized Email
    │     ├── Analyze website HTML for business details
    │     ├── Reference 1-2 specific details (name, services, etc.)
    │     └── Conversational, human-like tone
    └── Task 3: Send Email via Gmail (HTML formatted)
```

### Module Details

| Module ID | Service | Description |
|-----------|---------|-------------|
| `1` | `browse-ai:onTaskFinished` | Webhook trigger — fires when a Browse AI task completes |
| `2` | `google-sheets:filterRows` | Finds the row where column G matches the finished task ID |
| `3` | `ai-local-agent:RunLocalAIAgent` | Runs the AI Agent (model: large) |

### AI Agent — Full Task Breakdown

The AI Agent receives the following data from the matched sheet row:

```
First Name    → Sheet column A
Last Name     → Sheet column B
Phone Number  → Sheet column C
Email         → Sheet column D
Website HTML  → Sheet column E (scraped content)
Message       → Sheet column F
```

**Task 1 — Evaluate & Schedule:**
- First, determine whether the message warrants a consultation. Skip scheduling if it's spam or off-topic.
- If a meeting is appropriate:
  - Availability window: **Monday–Friday, 10:00 AM – 4:00 PM US Central Time**
  - If the requested time is available → Create a 30-minute Google Meet event with the contact as an invitee, including a brief agenda based on their message
  - If unavailable → Propose 2–3 alternative times within the next 5 business days

**Task 2 — Personalized Email:**
- Analyze the website HTML to understand the business (name, services, about page, personal details)
- Reference 1–2 specific details from their site to make the email feel tailored
- Confirm the meeting time or propose alternatives
- Keep it to 3–5 short paragraphs with a conversational, human tone
- Sign off: `Best wishes, Joe`

**Task 3 — Send Email:**
- Send an HTML-formatted email via Gmail
- Keep formatting simple — should feel like a quick human reply

### AI Agent Hard Constraints

```
❌ Never open with "I hope this email finds you well" or similar clichés
❌ No bullet points, numbered lists, or em dashes in the reply
❌ Do not over-compliment or sound sycophantic
❌ Never reveal that you are an AI or mention analyzing their HTML
❌ Never schedule outside the defined availability window
```

### Setup Requirements

- ✅ Browse AI webhook (`onTaskFinished`) configured
- ✅ Same Google Sheet connected (`1t_MVDqDSe9H6jvnZGKJqNI5lwYboUP-NNm_r0rm6PE8`)
- ✅ AI Local Agent (large model) configured in Make.com
- ✅ Google Calendar access granted to the AI Agent
- ✅ Gmail access granted to the AI Agent

---

## 🔗 How Both Scenarios Connect

```
[Scenario 1]                               [Scenario 2]
Tally Form                                 Browse AI
Submit ──→ Sheet Row ──→ Scrape ──→        Webhook ──→ Match Row ──→ AI Agent ──→ Email Sent
                          Task ID                    by Task ID
                          saved in
                          Column G
```

Both scenarios are linked via **Column G (Browse AI Task ID)**. Scenario 1 saves the task ID after triggering the scrape. Scenario 2 uses that same ID to find the correct row and pass all the data to the AI Agent.

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

## 🚀 How to Import & Set Up

1. **Import Scenario 1:**
   - Open Make.com → Create New Scenario → Import Blueprint
   - Upload `Part1.txt`
   - Connect your accounts: Tally, Google Sheets, Browse AI

2. **Import Scenario 2:**
   - Same steps → Upload `Part2.txt`
   - Connect: Browse AI, Google Sheets, AI Agent, Gmail, Google Calendar

3. **Set up the Google Sheet:**
   - Sheet name: `Sheet1`
   - Column headers: `First Name | Last Name | Phone Number | Email | URL | Message | Task ID`

4. **Configure the Browse AI Robot:**
   - Recreate or import robot ID `019dce51-796a-745f-8e3b-6cb9da02bfd8` in your Browse AI account
   - Set the task to capture full page HTML from the provided URL

5. **Activate both scenarios** and submit a test entry via the Tally form to verify the full pipeline.

---

## ⚠️ Important Notes

- Scenario 1 and Scenario 2 are **separate Make.com scenarios** — each has its own trigger
- The Browse AI Task ID in Column G is what links the two scenarios together
- The AI Agent model is set to `large` — switching to `small` may reduce costs but could affect reply quality
- The AI is intentionally designed to never reveal it is an AI in the email response

---

*Built with Make.com | Browse AI | Tally | Google Workspace*
