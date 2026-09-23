# BASANTI

The AI Oracle & Zero-UI Terminal for the Modern Kirana Store

A Python-based retail operations assistant built for small grocery and kirana stores. Basanti lets store owners interact with their business through natural language in Telegram, automatically updates inventory and sales records, and exposes a live dashboard for operational insights.

## 1. Overview

Basanti is designed around a key real-world problem: kirana store owners are busy, do not want to fill forms or use complex dashboards, and prefer communicating in the same place they already use daily — Telegram.

Instead of requiring manual POS entry, the system interprets simple natural-language messages such as:

- "Sold 2 milk packets"
- "Supplier arrived, add 50 Maggi and 20 eggs"
- "What is selling poorly?"
- "Add ₹500 to sales"
- "/advice"

The app converts those messages into structured actions, updates a SQLite database, and visualizes business health on a web dashboard.

This project is not a generic demo app. It is a small business automation system with the following goals:

- reduce manual data entry
- make inventory tracking easier
- capture sales in natural language
- provide analytics and alerts
- allow voice and image-based logging
- keep the experience lightweight and conversational

---

## 2. Problem it solves

Kirana stores run on speed, trust, and quick decisions. Most owners do not have time to learn a POS system or maintain spreadsheets, which creates several business problems:

- stockouts without warning
- dead inventory that does not move
- lost sales without proper tracking
- poor understanding of daily revenue
- difficult restocking decisions
- no real-time operations overview

Basanti solves that by turning communication into operational intelligence.

---

## 3. Core concept

Basanti follows a zero-UI retail workflow:

1. User sends a message or media file to a Telegram bot.
2. The app receives the Telegram webhook payload.
3. AI interprets the message as a structured business action.
4. The database is updated with inventory or sales changes.
5. The dashboard refreshes with live metrics.
6. The system optionally sends strategic advice and alerts.

This turns messaging into a lightweight operational system.

---

## 4. System architecture

```text
Store Owner
    │
    │ Text / Voice / Photo / CSV / Excel
    ▼
Telegram Bot
    │
    │ Webhook POST
    ▼
FastAPI Backend (main.py)
    ├── Receives request
    ├── Logs user messages
    ├── Dispatches background task
    ├── Sends outbound Telegram replies
    └── Exposes dashboard and APIs
    │
    ▼
AI Intent Engine (ai_core.py)
    ├── Text classification
    ├── Voice transcription
    ├── Receipt parsing
    └── JSON action extraction
    │
    ▼
SQLite Database (database.py)
    ├── sales
    ├── inventory
    ├── chat_logs
    └── business context generation
    │
    ├── Dashboard Queries
    ├── Health Metrics
    └── Live Inventory Data
    │
    ▼
Web Dashboard (templates/index.html)
    ├── Revenue overview
    ├── Inventory table
    ├── 7-day performance chart
    ├── Popular items
    └── Message log
```

### Architectural idea
The app separates concerns into distinct layers:

- ingestion: Telegram and webhook handling
- reasoning: AI intent extraction
- persistence: SQLite writes and reads
- visualization: web dashboard
- advisory: market/weather strategy layer

This keeps the project modular and easy to reason about.

---

## 5. Features

### 5.1 Natural-language sales logging
Users can send messages such as:

- "2 Maggi sold"
- "Sold 3 cumin packets"
- "Add 500 rupees to sales"

The AI identifies the sales intent and the database records the transaction.

### 5.2 Inventory restocking
Messages such as:

- "Supplier came, add 50 rice bags"
- "Bought 30 eggs"
- "Received 20 milk packs"

are interpreted as restock actions and the inventory quantity increases.

### 5.3 Voice note transcription
If the user sends a voice note, the app downloads the audio file from Telegram, transcribes it using the Groq Whisper integration, and then processes it as a normal text input.

### 5.4 Receipt image parsing
If a receipt or bill image is uploaded, Basanti attempts to read the total and log it as a sale.

### 5.5 Report generation and business analysis
The AI can answer questions such as:

- What should I stop restocking?
- Which products are moving?
- What is my sales today?
- How much stock is left?

It will read from the live database context and answer using that data.

### 5.6 Strategic business advice
When the user sends `/advice`, the app calls the advisor module, which fetches:

- weather data
- market/news headlines
- current inventory snapshot
- top sellers

and returns a short business strategy brief.

### 5.7 Proactive low-stock alerting
A background loop checks inventory every 30 seconds. If an item has quantity below 10 and above 0, it sends an alert to the first tracked chat session.

### 5.8 Dashboard with live operational data
The main dashboard includes:

- total revenue today
- recent sales log
- inventory table
- 7-day revenue chart
- top-selling items
- communication log
- health status modal

### 5.9 PDF export
The dashboard includes a client-side PDF exporting function with jsPDF.

### 5.10 Database reset endpoint
The app exposes an admin endpoint to wipe the database and reset the operational state.

---

## 6. Technology stack

| Layer | Technology | Purpose |
|---|---|---|
| Backend | Python | Core logic and application code |
| Web framework | FastAPI | HTTP API, dashboard routes, webhook handling |
| Bot platform | Telegram Bot API | User interaction and message events |
| AI provider | Groq | LLM-based intent extraction and transcription |
| Database | SQLite | Local persistence for sales, inventory, logs |
| Frontend | Jinja2 + Tailwind CSS + Chart.js | Live dashboard UI |
| Data retrieval | httpx | HTTP requests to Telegram, weather, and news APIs |
| Environment config | python-dotenv | Loads environment variables |
| Data processing | pandas | Inventory CSV/Excel parsing and database extraction |
| Extra dashboard | Streamlit | Alternative business terminal UI |

---

## 7. Project structure

```text
Basanti-main/
├── README.md
├── main.py
├── ai_core.py
├── advisor.py
├── config.py
├── database.py
├── inventory.py
├── memory.py
├── dashboard.py
├── dashboard_data.py
├── dashboard_styles.py
├── requirements.txt
├── sessions.json
├── kirana_store.db
├── templates/
│   └── index.html
└── .env (optional, for secrets)
```

### File-by-file explanation

| File | Purpose |
|---|---|
| main.py | Main FastAPI app, webhook route, alert loop, Telegram messaging logic |
| ai_core.py | LLM routing, text parsing, voice transcription, receipt vision processing |
| database.py | SQLite schema, data mutation, query generation, dashboard payload |
| memory.py | Chat memory persistence and recent-context tracking |
| inventory.py | Inventory file upload processing from Telegram |
| advisor.py | Weather and market brief generation |
| config.py | Loads environment variables and defines service API URLs |
| dashboard.py | Streamlit dashboard for alternative business visualization |
| dashboard_data.py | SQL fetching logic for Streamlit dashboard |
| dashboard_styles.py | Neo-brutalist styling for Streamlit |
| templates/index.html | Main dashboard UI and JS logic |
| requirements.txt | Python dependency list |
| sessions.json | Saved chat memory |
| kirana_store.db | SQLite database file |

---

## 8. Environment configuration

The app expects environment variables in a `.env` file. The actual code loads them using `python-dotenv` in [config.py](config.py).

### Example .env file

```env
GROQ_API_KEY=your_groq_key_here
TELEGRAM_TOKEN=your_telegram_bot_token_here
```

### Notes
- The project expects `GROQ_API_KEY` and `TELEGRAM_TOKEN` to be present.
- If either is missing, [config.py](config.py) raises a `ValueError`.
- The code also defines `OPENWEATHER_API_KEY` and `NEWS_API_KEY` in [config.py](config.py), but they are currently hardcoded in the file rather than loaded from `.env`.

### Important production note
For production use, it is strongly recommended to move all API keys to `.env` and never commit them to source control.

---

## 9. Installation and setup

### 9.1 Prerequisites

- Python 3.10+
- pip
- Telegram bot token
- Groq API key
- Optional: OpenWeather API key and NewsAPI key

### 9.2 Clone the repository

```bash
git clone <your-repository-url>
cd Basanti-main
```

### 9.3 Create a virtual environment (recommended)

```bash
python -m venv venv
source venv/bin/activate
# Windows
# venv\Scripts\activate
```

### 9.4 Install dependencies

```bash
pip install -r requirements.txt
```

### 9.5 Configure environment variables

Create a `.env` file in the project root:

```env
GROQ_API_KEY=your_groq_key_here
TELEGRAM_TOKEN=your_telegram_bot_token_here
```

If you want to use the weather/news features, update the values in [config.py](config.py) or move them into `.env` for safer configuration.

### 9.6 Run the application

```bash
uvicorn main:app --reload
```

The app will run by default on:

```text
http://localhost:8000
```

The dashboard is available at:

```text
http://localhost:8000/
http://localhost:8000/dashboard
```

---

## 10. Telegram webhook setup

The app listens for Telegram webhook updates on the `/webhook` route.

### Flow

1. Telegram sends update payload to your public HTTPS endpoint.
2. FastAPI receives the payload.
3. It checks whether the update contains a message.
4. It schedules processing in the background.
5. The bot replies with the result.

### Local testing
For local development, tools like ngrok are usually used to expose the app publicly.

Example:

```bash
ngrok http 8000
```

Then configure the Telegram bot to send webhook updates to the public ngrok URL.

---

## 11. Main API routes

The app exposes the following routes in [main.py](main.py):

| Route | Method | Purpose |
|---|---|---|
| / | GET | Main dashboard page |
| /dashboard | GET | Alias to the dashboard |
| /test | GET | Health-check endpoint |
| /api/sales/all | GET | Returns all sales records for PDF export / client-side use |
| /api/nuke | POST | Wipes the database |
| /webhook | POST | Telegram webhook receiver |

### Example response from /test

```json
{"status": "Basanti is alive and the code updated!"}
```

---

## 12. Database design

The project uses SQLite with 3 main tables.

### 12.1 sales

| Column | Type | Description |
|---|---|---|
| id | INTEGER PRIMARY KEY | Row ID |
| item | TEXT | Item names and quantity summary string |
| amount | REAL | Total revenue for the sale entry |
| timestamp | DATE DEFAULT CURRENT_DATE | Sale date |

### 12.2 inventory

| Column | Type | Description |
|---|---|---|
| id | INTEGER PRIMARY KEY | Row ID |
| item_name | TEXT | Name of the inventory item |
| quantity | INTEGER | Current item count |
| price | REAL | Unit price |

### 12.3 chat_logs

| Column | Type | Description |
|---|---|---|
| id | INTEGER PRIMARY KEY | Row ID |
| sender | TEXT | Sender name or system label |
| message | TEXT | Log message |
| timestamp | DATETIME DEFAULT (datetime('now', 'localtime')) | Timestamp of the log |

### Database initialization
The database is created by `init_db()` inside [database.py](database.py).

### Database helper behavior
The app generates dashboard context from the database using queries like:

- total sales today
- inventory snapshot
- top sellers
- dead stock items
- recent sales log
- daily revenue over the last 7 days

---

## 13. AI behavior and prompt logic

The AI layer is implemented in [ai_core.py](ai_core.py), and this is one of the most important parts of the project.

### 13.1 Input types supported
The app processes:

- text messages
- voice notes
- photos and receipts
- uploaded CSV or Excel inventory files

### 13.2 Input processing pipeline

#### Text input
- Reads the user message
- Pulls chat history from memory
- Pulls live database context from SQLite
- Injects all this into a prompt
- Sends it to Groq LLM
- Parses JSON output
- Executes database mutation if needed

#### Voice input
- Downloads the Telegram voice file
- Calls Groq Whisper transcription
- Converts the transcribed text into a business action

#### Image input
- Downloads image bytes from Telegram
- Encodes image as base64
- Sends to a vision-capable model
- Asked to output JSON with the sale total

#### File upload
- Accepts `.csv`, `.xlsx`, and `.xls`
- Parses inventory data
- Validates required columns
- Upserts inventory into SQLite

### 13.3 Strict JSON output rule
The prompt is designed to enforce a strict JSON response so that the app can reliably parse AI output.

The model is instructed to return only valid JSON with fields such as:

```json
{
  "type": "sale",
  "items": [{"name": "Milk", "qty": 2, "total": 100}],
  "amount": 100,
  "advice": "Logged 2 Milk for ₹100."
}
```

### 13.4 Important AI rules encoded in the prompt
The model is explicitly told:

- use `sale` for sold items or additions to sales
- use `restock` for stock received or added
- use `report` for business questions
- use `chat` for read-only questions
- never trust conversation memory over live database stats for numbers

This is important because direct LLM output can otherwise drift or hallucinate.

---

## 14. Database context and AI grounding

The AI is intentionally grounded in database context before answering.

The function `get_db_context()` generates a formatted summary such as:

```text
[LIVE STATS] Today: ₹4200
[INVENTORY] Milk (Qty: 100, ₹40), Bread (Qty: 30, ₹30)
[TOP SELLERS] Milk (10 sold), Bread (7 sold)
[DEAD STOCK (0 Sales)] Rice, Flour
```

This reduces hallucination because the model is not making up store metrics; it is answering based on recent real data.

---

## 15. Memory system

The project uses `sessions.json` and the `memory.py` module to remember recent messages by chat ID.

### Memory behavior
- loads existing chat sessions from disk on boot
- stores each user/bot turn
- keeps the last 3 exchanges per chat
- trims long messages to avoid overly large prompts
- saves instantly after each update

### Memory functions

| Function | Description |
|---|---|
| load_sessions() | Reads memory from disk |
| save_sessions() | Writes memory back to the session file |
| get_context(chat_id) | Returns last few messages for a chat |
| update_context(chat_id, user_msg, bot_msg) | Adds a new turn and truncates history |

This gives the app conversational continuity without overwhelming the model with too much history.

---

## 16. Inventory upload processing

Inventory files can be uploaded through Telegram and processed by [inventory.py](inventory.py).

### Supported file types

- `.csv`
- `.xlsx`
- `.xls`

### Required columns

| Column | Required | Notes |
|---|---|---|
| item | Yes | Item name |
| quantity | Yes | Numeric stock count |
| price | Yes | Unit price |

### Processing behavior
- downloads file from Telegram
- normalizes column names to lowercase
- validates required columns
- converts numeric values safely
- drops invalid rows
- upserts stock into the database

### Upsert logic
If an item already exists, the app adds the uploaded quantity to the existing inventory count and updates the latest price.

If the product is new, it inserts a fresh inventory record.

---

## 17. Dashboard overview

The main dashboard is rendered from [templates/index.html](templates/index.html), using Jinja2 and data passed from `get_dashboard_payload()`.

### Dashboard sections

| Section | Purpose |
|---|---|
| Overview | Revenue summary, strategic advice, recent activity |
| Inventory | Stock table with low-stock warnings |
| Comparison | 7-day revenue chart |
| Popularity | Top-selling items and leaderboard |
| Vault | Communication log |
| Health Modal | Business diagnosis and status |

### Dashboard data model
The backend returns a payload containing:

```json
{
  "revenue": 12500,
  "sales": [...],
  "inventory": [...],
  "chat_logs": [...],
  "daily_revenue": [...],
  "popular_items": [...],
  "health": {
    "orders_today": 25,
    "aov": 500,
    "low_stock_count": 3,
    "status": "OPTIMAL"
  }
}
```

### Web UI notes
The dashboard design intentionally mimics a black-and-yellow, high-contrast terminal aesthetic.

It is meant to feel like a command center for a busy retail store rather than a polished consumer app.

---

## 18. Proactive alert system

The function `proactive_stock_alert()` in [main.py](main.py) runs continuously in the background.

### Logic
- every 30 seconds, check inventory
- look for items with quantity less than 10 and greater than 0
- if found, send a Telegram message to the first active user session
- log the alert in the chat log
- pause to avoid spam

### Example alert

```text
🚨 *BASANTI AUTO-ALERT* 🚨

Boss, we are running critically low on:
• Milk (Only 4 left!)
• Bread (Only 3 left!)

Text me to restock when the supplier arrives!
```

---

## 19. Sending Telegram messages

The main app includes helper functions in [main.py](main.py):

- `send_telegram_message(chat_id, text_message)`
- `get_telegram_file_url(file_id)`
- `format_bot_response(analysis)`

These handle:

- outbound bot replies
- media file downloads
- message formatting for Telegram
- safe exception handling when Telegram is unavailable

---

## 20. Request processing flow

The key function `process_telegram_update(message)` does the following:

1. extracts `chat_id`
2. handles text, voice, photo, or document messages
3. logs the user input
4. checks special command `/advice`
5. calls AI processing based on media type
6. formats the result
7. logs the bot response
8. sends the reply to Telegram
9. updates memory context

This is the main operational control loop.

---

## 21. Special /advice command

This is a strategically important workflow in the app.

When the user sends `/advice`, Basanti:

- reads the live dashboard payload
- calls `generate_business_advice(payload)` from [advisor.py](advisor.py)
- combines:
  - weather
  - market headlines
  - inventory summary
  - popular items
- returns a brief recommendation
- sends it back to Telegram

This bypasses the normal `analyze_kirana_data()` flow for a focused strategy brief.

---

## 22. Advisor module details

[advisor.py](advisor.py) is the decision-support layer.

### Data sources

- OpenWeatherMap for weather
- NewsAPI for retail and commodity headlines

### Current functionality
It fetches this information and builds a summary prompt. At the moment, the code returns a deterministic mock strategy string rather than a true dynamic LLM-generated recommendation.

### Example generated advice

```text
Heavy rain is forecast for tomorrow. Move the biscuits to the front counter and check umbrella stock. Also, news indicates a potential hike in dairy prices—consider updating your milk margins.
```

This is useful for a prototype but should be upgraded in production to use the same LLM system as the main assistant.

---

## 23. Important implementation details and patterns

### 23.1 `BackgroundTasks` usage
The app uses FastAPI background tasks to process incoming Telegram messages while leaving the webhook response quick.

This keeps the bot responsive and lowers user waiting time.

### 23.2 AI prompt grounding
The model sees the real-time database context and previous conversation summary before making decisions. This is a strong design choice because it reduces errors and false statements.

### 23.3 Local DB-first architecture
The app stores everything in SQLite rather than a remote database. That makes it:

- simple
- self-contained
- easy to run locally
- good for prototype and demo environments

### 23.4 “Zero-UI” design philosophy
Basanti is intentionally built around messaging rather than app forms. This makes it highly usable for business owners who do not want to enter data manually.

---

## 24. Good components already in the project

These are the parts that make the project strong and portfolio-worthy:

| Component | Why it matters |
|---|---|
| FastAPI backend | Modern Python API framework |
| Telegram webhook integration | Real-world user interaction model |
| AI intent parsing | Demonstrates practical LLM application |
| SQLite database | Lightweight and reliable local persistence |
| Dashboard UI | Shows ability to visualize operational data |
| Voice transcription | Adds multimodal interaction |
| Receipt parsing | Useful for real-world business workflows |
| File upload support | Helps with supplier inventory management |
| Background alerts | Adds proactive operations logic |
| Memory system | Keeps interactions context-aware |

---

## 25. Weaknesses and limitations

This project is strong as a prototype, but there are real limitations that should be acknowledged.

### 25.1 Hardcoded secrets
Keys are currently hardcoded in [config.py](config.py) for weather and news APIs. This is not a secure production practice.

### 25.2 Not enough schema normalization
Sales are currently stored as textual strings in the `sales.item` field. This can make analytics and reporting harder over time.

### 25.3 Partial inconsistency between modules
The Streamlit dashboard refers to a table called `khata`, but the SQLite schema in [database.py](database.py) does not create that table.

### 25.4 No real authentication or admin security
There is no built-in auth layer, so running externally without protection would be risky.

### 25.5 No automated tests
There are no tests for AI parsing, DB mutation, or API endpoints.

### 25.6 Limited production resilience
No retry logic, no strong monitoring, no health endpoints beyond `/test`, and no structured error logging.

---

## 26. Recommended next improvements

To make this project production-grade, the next upgrades would be:

1. Move all secrets into `.env` and validate them properly.
2. Normalize the database schema into dedicated transaction tables.
3. Add a proper admin authentication layer.
4. Add automated tests for AI parsing and DB logic.
5. Add robust logging and observability.
6. Replace mock advisor logic with a stronger, LLM-driven recommendation engine.
7. Add pagination and filtering to the dashboard.
8. Add error and retry handling for external API requests.
9. Add webhook verification for Telegram security.
10. Consider deployment via Docker or a cloud service.

---

## 27. Example user interaction scenarios

### Scenario 1: Sale logging
User message:

```text
Sold 2 Milk and 1 Bread
```

Expected behavior:

- AI classifies as sale
- inventory is decreased
- revenue increases
- bot replies with a sale confirmation

### Scenario 2: Restock
User message:

```text
Supplier came. Add 50 Maggi and 20 Eggs.
```

Expected behavior:

- AI classifies as restock
- stock counts increase
- dashboard updates

### Scenario 3: Query
User message:

```text
What item should I stop restocking?
```

Expected behavior:

- AI reads database context
- identifies dead or low-moving items
- returns recommendation

### Scenario 4: Advice request
User message:

```text
/advice
```

Expected behavior:

- weather and market feed are fetched
- strategy brief is generated
- user receives actionable retail advice

---

## 28. Demo flow to test manually

Use a Telegram bot account to send messages like these:

| Action | Example message |
|---|---|
| Restock | "Basanti, the supplier arrived. Add 50 Maggi and 100 Eggs." |
| Sale | "Sold 2 Maggi." |
| Manual sales entry | "Add 500 rupees to sales." |
| Report | "What is my revenue today?" |
| Advice | "/advice" |
| Inventory upload | Send a CSV or Excel file with Item, Quantity, Price columns |

The dashboard should update automatically based on the database changes.

---

## 29. Repository-specific notes

This repository is intentionally built as a compact, practical MVP for smart retail operations. It is not designed to replace enterprise POS systems, but it demonstrates a complete idea cycle:

- capture user input
- interpret it with AI
- persist to a database
- present live operational dashboards
- automate business insight delivery

That makes it a good showcase project for AI + backend + automation + business workflow design.

---

## 30. Final verdict

Basanti is a thoughtful and usable AI-powered business tool for local retail operations. It demonstrates a clear understanding of:

- natural-language interfaces
- AI-driven automation
- operational dashboards
- business logic
- Telegram integration
- database-driven workflows

It is especially strong as a portfolio project because it shows how a practical idea can be turned into a working system using a small but meaningful technical stack.

---

## 31. Quick start command summary

```bash
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
# create .env with GROQ_API_KEY and TELEGRAM_TOKEN
uvicorn main:app --reload
```

Then open:

```text
http://localhost:8000
```

and configure Telegram webhook to point to your public endpoint.

---

## 32. Conclusion

Basanti combines modern AI, workflow automation, and data-driven business operations into one practical tool tailored for the realities of a kirana store. The project is a compelling example of how AI can be applied to everyday operational tasks rather than abstract software use cases.

It is best described as a real-world business automation assistant that sits in the same communication layer as the owner and turns messages into business intelligence.
