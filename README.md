# 📧 Gmail AI Reply

An intelligent email assistant built on **n8n** that automatically reads incoming Gmail messages, classifies them with labels, drafts polished HTML replies using GPT-4.1-mini — all without you lifting a finger.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Setup & Installation](#setup--installation)
- [Workflow Nodes](#workflow-nodes)
- [Label Classification](#label-classification)
- [Draft Output Format](#draft-output-format)
- [Usage Example](#usage-example)
- [Configuration](#configuration)
- [Tech Stack](#tech-stack)

---

## Overview

This n8n workflow monitors your Gmail inbox every minute. When a new email arrives, an AI agent reads the subject and body, picks the right label (course, doubts, or sponsorship), applies it to the email, and creates a ready-to-send HTML draft reply — all automatically saved in your Gmail Drafts folder.

---

## ✨ Features

- 📬 **Automatic inbox polling** — Checks for new emails every minute
- 🏷️ **Smart label classification** — Categorizes emails as `course`, `doubts`, or `sponsorship`
- 🤖 **AI-drafted replies** — GPT-4.1-mini writes respectful, context-aware responses
- 🧱 **HTML-formatted emails** — Replies are structured and easy to read
- 📝 **Draft creation** — Saves replies as Gmail Drafts (you stay in control before sending)
- 🔗 **Thread-aware** — Drafts are created within the correct email thread

---

## 🏗️ Architecture

```
Gmail Trigger (every minute)
        │
        ▼
  Get Full Message
        │
        ▼
    AI Agent  ◄── GPT-4.1-mini
        │      ◄── Get Gmail Labels Tool
        │      ◄── Structured Output Parser
        │
        ▼
 Add Label to Email
        │
        ▼
  Create Draft Reply
```

---

## ✅ Prerequisites

Before importing this workflow, ensure you have:

- An active **n8n** instance (self-hosted or cloud)
- A **Gmail account** with OAuth2 credentials configured in n8n
- An **OpenAI API key** (for GPT-4.1-mini)
- Gmail labels already created: `course`, `doubts`, `sponsorship`

---

## 🚀 Setup & Installation

### 1. Import the Workflow

- Go to **n8n → Workflows → Import from File**
- Select `Gmail_AI_Reply.json`

### 2. Configure Credentials

| Credential | Used By |
|---|---|
| Gmail OAuth2 | Gmail Trigger, Get Message, Add Label, Get Labels, Create Draft |
| OpenAI API | GPT-4.1-mini Chat Model |

### 3. Create Gmail Labels

Make sure these three labels exist in your Gmail account before activating:

- `course`
- `doubts`
- `sponsorship`

The AI agent fetches label IDs dynamically at runtime via the **Get many labels** tool, so no manual ID entry is needed.

### 4. Activate the Workflow

Toggle the workflow to **Active**. It will now poll your inbox every minute automatically.

---

## 🔧 Workflow Nodes

| Node | Type | Purpose |
|---|---|---|
| **check for new Mail** | Gmail Trigger | Polls inbox every minute for new emails |
| **Get a message** | Gmail | Fetches the full message content by ID |
| **AI Agent** | LangChain Agent | Classifies the email and drafts a reply |
| **OpenAI Chat Model** | LLM | GPT-4.1-mini as the AI brain |
| **Get many labels in Gmail** | Gmail Tool | Retrieves all Gmail labels and their IDs |
| **Structured Output Parser** | Output Parser | Enforces structured JSON output from the agent |
| **Add label to message** | Gmail | Applies the AI-chosen label to the email |
| **Create a draft** | Gmail | Saves the AI-generated reply as a Gmail Draft |

---

## 🏷️ Label Classification

The AI agent automatically assigns one of these labels based on email content:

| Label | When It's Used |
|---|---|
| `course` | Emails related to courses, lessons, or learning materials |
| `doubts` | Student questions, clarifications, or technical queries |
| `sponsorship` | Sponsorship requests or partnership inquiries |

---

## 📄 Draft Output Format

The AI agent returns a structured JSON response that drives the rest of the workflow:

```json
{
  "label": "doubts",
  "label_id": "Label_9122968551863008153",
  "subject": "Re: Doubts",
  "body": "<html><body><p>Dear Student,</p>...</body></html>"
}
```

- **`label`** — Human-readable label name
- **`label_id`** — Gmail label ID (fetched dynamically by the agent)
- **`subject`** — Reply subject line (auto-prefixed with `Re:`)
- **`body`** — Full HTML reply body

---

## 💡 Usage Example

**Incoming email:**
> **Subject:** Doubts
> **Body:** Please help in mastering n8n

**What the workflow does:**
1. Reads the email and identifies it as a `doubts` type
2. Applies the `doubts` label in Gmail
3. Generates an HTML reply explaining n8n basics with tips and examples
4. Saves the response as a Draft in the same thread, addressed back to the sender

**Generated draft (excerpt):**
```html
<p>Dear Student,</p>
<p>Thank you for reaching out with your query regarding mastering n8n...</p>
<ul>
  <li>Understand the basics of workflow automation...</li>
  <li>Explore the n8n documentation and tutorials...</li>
  <li>Practice by creating simple automation workflows...</li>
</ul>
```

---

## ⚙️ Configuration

### Polling Frequency

The trigger currently checks every minute. To change it, edit the **check for new Mail** node:

```
mode: "everyMinute"  →  change to "everyHour", "everyDay", etc.
```

### Adding More Label Categories

To add new labels (e.g., `billing`, `feedback`):
1. Create the label in Gmail
2. Update the AI Agent's system prompt to include the new label name in the classification list

### Switching to Auto-Send

Currently, the workflow creates **Drafts** for human review. To auto-send instead, replace the **Create a draft** node with a **Send** operation in the Gmail node — use this carefully!

### Model

Currently uses `gpt-4.1-mini`. To use a more powerful model, update the **OpenAI Chat Model** node.

---

## 🛠️ Tech Stack

- **[n8n](https://n8n.io/)** — Workflow automation platform
- **[Gmail API](https://developers.google.com/gmail/api)** — Email reading, labeling, and draft creation
- **[OpenAI GPT-4.1-mini](https://openai.com/)** — Email classification and reply generation

---

## ⚠️ Notes

- The workflow creates **Drafts only** — it never sends emails automatically. You always review before hitting Send.
- The AI reads only the email `snippet` (first ~100 characters), not the full body. For longer email processing, consider fetching the full decoded body from the payload.
- Pinned test data is included in the JSON for development and testing purposes.

---

## 📄 License

This project is open for personal use. Feel free to fork and extend it with additional labels, integrations, or auto-send logic!
