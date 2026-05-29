# 🏠 Idealista Multi-User Monitoring Bot

> An intelligent, event-driven Telegram bot built with **n8n** that monitors real estate listings on Idealista and delivers personalized alerts to multiple users in real time.

![n8n](https://img.shields.io/badge/n8n-workflow-orange?logo=n8n)
![Telegram](https://img.shields.io/badge/Telegram-Bot-2CA5E0?logo=telegram)
![Google Sheets](https://img.shields.io/badge/Google%20Sheets-Database-34A853?logo=googlesheets)
![JavaScript](https://img.shields.io/badge/JavaScript-Logic-F7DF1E?logo=javascript)
![License](https://img.shields.io/badge/license-MIT-green)

---

## 📌 Overview

This project automates the discovery and delivery of new real estate listings from [Idealista](https://www.idealista.com) — Spain's largest property portal. Users interact with a Telegram bot to register and receive instant, filtered notifications whenever new listings matching their criteria appear.

Built as part of the **Especialista en Inteligencia Artificial** program at EDE Fundazioa / Lanbide (Basque Government), it demonstrates practical skills in workflow automation, serverless architecture, web scraping, and multi-user data management.

---

## ✨ Key Features

- **Multi-user support** — unlimited users via Telegram, each with a personalized experience
- **On-demand activation** — workflow triggers only when a user sends a message (event-driven, no idle resource consumption)
- **Smart time parser** — converts natural-language timestamps ("3 horas", "Publicado ayer", "15 may") into minutes for precise filtering
- **Duplicate prevention** — validates each user by `chat_id` before registering, eliminating duplicate entries
- **Serverless database** — uses Google Sheets as a lightweight, zero-infrastructure user backend
- **Conditional routing** — intelligently branches between "new listings found" and "no new listings" notification flows
- **Structured notifications** — each alert includes title, price, features, direct link, and publication time

---

## 🔧 Tech Stack

| Layer | Technology |
|---|---|
| Workflow Automation | n8n |
| Messaging | Telegram Bot API |
| Web Scraping | n8n HTTP Request + HTML Extract nodes |
| Database | Google Sheets (OAuth2) |
| Logic / Data Processing | JavaScript (n8n Code nodes) |
| Hosting | n8n Cloud / Self-hosted |

---

## 🗺️ Workflow Architecture

The workflow is composed of **18 interconnected nodes** across three main flows:

```
┌─────────────────────────────────────────────────────────┐
│  FLOW 1 — User Registration                              │
│                                                          │
│  Telegram Trigger → Edit Fields → Merge                  │
│       → Verify Duplicate → Save to Google Sheets        │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│  FLOW 2 — Listing Monitoring                             │
│                                                          │
│  HTTP Request (Idealista) → Extract HTML Data            │
│       → Clean & Parse Data → Filter by Time (<6h)       │
└─────────────────────────────────────────────────────────┘
                          │
             ┌────────────┴────────────┐
             ▼                         ▼
┌────────────────────┐     ┌───────────────────────────┐
│  New listings found│     │  No new listings           │
│  Format messages   │     │  Send "no results" message │
│  → Send to Telegram│     │  → Send to Telegram        │
└────────────────────┘     └───────────────────────────┘
```

### Node Summary

| # | Node | Function |
|---|---|---|
| 1 | Telegram Trigger | Captures user messages via webhook |
| 2 | Edit Fields | Extracts `chat_id`, `username`, `text`, `date` |
| 3 | Leer Google Sheets | Reads existing user database |
| 4 | Merge | Combines new user data with existing records |
| 5 | Verificar Duplicado | Prevents duplicate registrations by `chat_id` |
| 6 | Guardar Google Sheets | Persists new users to the database |
| 7 | HTTP Request Idealista | Scrapes Bilbao rental listings page |
| 8 | Extraer Datos | Parses HTML with CSS selectors |
| 9 | Code limpiar datos | Normalizes data, converts timestamps to minutes |
| 10 | ¿Hay anuncios nuevos? | Switch node — routes based on listing age (<360 min) |
| 11–14 | Code_con / Code_sin / Counters | Filters, counts, and formats results |
| 15–18 | Telegram Send Nodes | Delivers personalized messages to each user |

---

## 🧠 Technical Highlights

### Smart Timestamp Parser
Converts Spanish natural-language publication times to numeric minutes:
```javascript
function convertirATiempoEnMinutos(texto) {
  if (texto === 'publicado ayer') return 1440;         // yesterday = 1440 min
  if (match = texto.match(/(\d+)\s*hora/)) return match[1] * 60;
  if (match = texto.match(/(\d+)\s*min/))  return match[1];
  if (match = texto.match(/^(\d+)\s+(ene|feb|mar...)/)) {
    // Calculate minutes since specific date
  }
}
```

### Duplicate Prevention Logic
```javascript
const nuevoUsuario = items[0].json;
const existentes   = items.slice(1).map(item => item.json);
const yaExiste     = existentes.some(row => row.chat_id === nuevoUsuario.chat_id);
return yaExiste ? [] : [{ json: nuevoUsuario }];
```

### Conditional Routing
```javascript
// Routes to branch 0 (new listings) or 1 (no listings)
$input.all().some(item =>
  typeof item.json.Minutos === 'number' && item.json.Minutos < 360
) ? 0 : 1
```

---

## 📋 Google Sheets Schema

The user database uses a simple 5-column structure:

| A | B | C | D | E |
|---|---|---|---|---|
| chat_id | username | text | date | status |

---

## 🚀 Setup & Installation

### Prerequisites
- [n8n](https://n8n.io) account (Cloud or self-hosted)
- Telegram Bot token (via [@BotFather](https://t.me/BotFather))
- Google account with Sheets API enabled

### Steps

**1. Import the workflow**
```
n8n Dashboard → Workflows → Import → select idealista_todo.json
```

**2. Configure Telegram credential**
- Open `Telegram Trigger` node → Credentials → Create New
- Paste your Bot Token from BotFather
- Apply the same credential to all Telegram send nodes

**3. Configure Google Sheets credential**
- Open `Leer Google Sheets` node → Credentials → Google Sheets OAuth2
- Authorize with your Google account
- Create a new Sheet with headers: `chat_id | username | text | date | status`
- Replace the Spreadsheet ID in both Google Sheets nodes with your Sheet's ID

**4. Activate the workflow**
- Click **Activate** (top-right toggle) in n8n
- Send any message to your Telegram bot to test

---

## 📊 Performance Metrics

| Metric | Value |
|---|---|
| Concurrent users | Unlimited (serverless) |
| Response time | ~2–4 seconds per user |
| Filtering precision | >98% (listings <6 hours) |
| Duplicate prevention | 100% (chat_id validation) |
| Google Sheets API calls | 2 per active user |
| Bandwidth per execution | ~75 KB |

---

## 📁 Repository Structure

```
idealista-telegram-bot/
│
├── README.md
├── workflow/
│   └── Idealista con n8n.json       # n8n workflow export
└── docs/
    └── Idealista con n8n_en.pdf  # Full technical english report
    └── Idealista con n8n_es.pdf  # Full technical spanish report
└── assets/
    └── n8n_flowchart.png  # image of the n8n flowchart
    └── n8n_telegram.png  # image of telegram bot
```

---

## ⚠️ Known Limitation

Idealista uses **DataDome** bot protection, which blocks automated HTTP requests from cloud IPs. For production use, one of the following is recommended:

- **Idealista Official API** — request access at [developers.idealista.com](https://developers.idealista.com) (free, stable, recommended)
- **Self-hosted n8n** on your local machine — shares your browser's IP, works with session cookies
- **Premium scraping proxy** (e.g. ScraperAPI ultra_premium, ZenRows with JS rendering)

---

## 🔮 Potential Improvements

- [ ] Multi-city support (Madrid, Barcelona, Valencia...)
- [ ] Price range and room filters per user
- [ ] Daily digest summary messages
- [ ] Integration with Fotocasa and Habitaclia
- [ ] Analytics dashboard (clicks, conversion tracking)
- [ ] Scheduled automatic checks (every 6h via cron trigger)

---

## 👤 Author

**Arman Ghaziaskari Naeini**
Especialista en Inteligencia Artificial — EDE Fundazioa / Lanbide

🌐 Portfolio: [armanghazi.github.io/portfolio](https://armanghazi.github.io/portfolio)

---

## 📄 License

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.

---

> *Built with ❤️ in Bilbao, Basque Country*
