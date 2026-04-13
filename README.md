# ✈️ Travel Booking Agent — Powered by VAPI & n8n

> An AI-powered **voice travel booking agent** built with [n8n](https://n8n.io) and [VAPI](https://vapi.ai). It listens to a user's voice call, extracts travel intent, searches for real-time flights, resorts, and activities — then compiles a full itinerary and delivers it to the user's inbox via Gmail.

---

## 📸 Workflow Overview

![Workflow Diagram](travel_booking_agent__powered_by_VAPI_.png)

---

## 🧠 How It Works

The workflow is triggered by a **VAPI voice call webhook** and processes the conversation end-to-end through a multi-step n8n automation pipeline:

```
Voice Call (VAPI)
      │
      ▼
[1] Webhook          — Receives POST payload from VAPI after the call ends
      │
      ▼
[2] Edit Fields      — Extracts origin & destination from the voice transcript
      │
      ▼
[3] AI Agent         — Interprets travel intent using Google Gemini (with memory)
      │
      ▼
[4] Activities       — Searches for local activities via Tavily API
      │
      ▼
[5] Resort           — Searches for resorts via SerpAPI
      │
      ▼
[6] Code (JS)        — Normalizes and merges API response data
      │
      ▼
[7] Flights          — Searches for available flights via SerpAPI Google Flights
      │
      ▼
[8] AI Agent1        — Compiles a complete travel itinerary using Gemini (with memory)
      │
      ▼
[9] Gmail            — Sends the final itinerary to the user's email
```

---

## 🔧 Tech Stack & Services

| Component | Tool / Service |
|---|---|
| Workflow Automation | [n8n](https://n8n.io) |
| Voice AI Interface | [VAPI](https://vapi.ai) |
| LLM / AI Agent | [Google Gemini (PaLM API)](https://ai.google.dev/) |
| Activity Search | [Tavily API](https://tavily.com/) |
| Flight & Resort Search | [SerpAPI](https://serpapi.com/) |
| Email Delivery | [Gmail via OAuth2](https://developers.google.com/gmail/api) |
| Memory | n8n Window Buffer Memory (x2) |

---

## 🗂️ Project Structure

```
vapi-travel-booking-agent-n8n/
├── Travel_booking_agent__powered_by_VAPI.json   # n8n workflow export
├── travel_booking_agent__powered_by_VAPI_.png   # Workflow screenshot
└── README.md
```

---

## 🚀 Getting Started

### Prerequisites

- [n8n](https://docs.n8n.io/hosting/) instance (self-hosted or cloud)
- A [VAPI](https://vapi.ai) account with a configured assistant
- API keys for:
  - Google Gemini (PaLM API)
  - SerpAPI
  - Tavily API
- Gmail account with OAuth2 credentials configured in n8n

---

### 1. Import the Workflow

1. Open your n8n instance
2. Go to **Workflows → Import from File**
3. Upload `Travel_booking_agent__powered_by_VAPI.json`

---

### 2. Configure Credentials

Inside n8n, set up the following credentials:

| Credential | Used By | How to Get |
|---|---|---|
| `Google Gemini (PaLM) API` | AI Agent, AI Agent1 | [Google AI Studio](https://makersuite.google.com/app/apikey) |
| `SerpAPI Query Auth` | Resort, Flights nodes | [SerpAPI Dashboard](https://serpapi.com/dashboard) |
| `Tavily Header Auth` | Activities node | [Tavily API Keys](https://app.tavily.com/) |
| `Gmail OAuth2` | Send booking data to user | [Google Cloud Console](https://console.cloud.google.com/) |

---

### 3. Configure the VAPI Webhook

1. Copy the **Webhook URL** from the `Webhook` node in n8n
2. In your VAPI dashboard, go to your assistant's **End of Call Report** settings
3. Paste the n8n Webhook URL as the POST destination
4. Make sure the call transcription includes `artifact.messages` in the payload

---

### 4. Update the Gmail Node

In the `Send booking data to user` node:
- Replace the `sendTo` email with a dynamic field from the conversation (or a test email during development)
- Customize the subject and message body as needed

---

### 5. Activate the Workflow

Toggle the workflow to **Active** in n8n. It will now listen for incoming VAPI call webhooks.

---

## 🔍 Workflow Node Details

### Webhook
Receives the raw POST request from VAPI at the end of a voice call. The payload contains the full call transcript inside `message.artifact.messages`.

### Edit Fields
Parses the voice transcript to extract:
- `origin` — the user's departure city
- `destination` — the user's travel destination

Uses word-level confidence metadata from the VAPI transcript.

### AI Agent (1st)
Powered by Google Gemini with a sliding window memory buffer. Processes the extracted origin/destination and formulates structured search queries for the downstream API calls.

### Activities
Calls the **Tavily Search API** (POST) to find popular activities and things to do at the destination.

### Resort
Calls **SerpAPI** (GET) to search for hotels and resorts at the destination.

### Code in JavaScript
Normalizes and merges data from the Activities and Resort API responses into a unified format for the next agent.

### Flights
Calls **SerpAPI Google Flights** (GET) to find available flights between the origin and destination.

### AI Agent1 (2nd)
Powered by Google Gemini with its own memory buffer. Receives all collected data (activities, resorts, flights) and generates a clean, human-readable travel itinerary.

### Send Booking Data to User
Sends the final compiled itinerary to the user's email via **Gmail OAuth2**.

---

## 📋 Example Use Case

1. User calls the VAPI phone number
2. VAPI assistant asks: *"Where are you flying from, and where would you like to go?"*
3. User responds: *"I'm flying from New York to Bali"*
4. VAPI ends the call and sends the transcript to the n8n webhook
5. The workflow searches for Bali activities, resorts, and NY→Bali flights
6. An AI-generated itinerary is emailed to the user within seconds

---

## ⚠️ Important Notes

- **API Keys in JSON**: The exported workflow JSON may contain placeholder or real API keys inside node configurations. **Before pushing to GitHub**, audit the JSON and remove or rotate any sensitive keys. Use n8n's credential manager instead of hardcoding keys.
- The `Edit Fields` node uses hardcoded message indices (`messages[4]`, `messages[6]`) to extract origin and destination. These may need adjustment depending on your VAPI assistant's conversation structure.
- The JavaScript `Code` node currently adds a placeholder field (`myNewField`). This should be updated to properly transform and pass API data between nodes.

---

## 🛠️ Potential Improvements

- [ ] Dynamic email recipient extraction from the VAPI call payload
- [ ] Add error handling and fallback branches for failed API calls
- [ ] Replace hardcoded message indices with a more robust NLP extraction method
- [ ] Add a hotel booking confirmation step (e.g., via a booking API)
- [ ] Integrate a payment or calendar node for end-to-end booking
- [ ] Add Slack or WhatsApp notification as an alternative to email

---

## 📄 License

This project is open-source and available under the [MIT License](LICENSE).

---

## 🙌 Acknowledgements

- [n8n](https://n8n.io) — Low-code workflow automation
- [VAPI](https://vapi.ai) — Voice AI platform
- [Google Gemini](https://ai.google.dev/) — Large language model
- [SerpAPI](https://serpapi.com/) — Search engine results API
- [Tavily](https://tavily.com/) — AI-optimized search API
