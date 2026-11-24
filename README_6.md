# AI Appointment Booking Agent (n8n + Groq + Google Calendar + Gmail)

A conversational booking assistant for **Braid's by Pee**, a hair braiding studio. Clients chat with the assistant, **Priscilla**, to book an appointment. The agent checks the studio's Google Calendar for open time, books the slot, and sends the client a confirmation email through Gmail.

Built with [n8n](https://n8n.io) using an AI Agent with tool calling.

## What it does

- Understands natural requests like *"I'd like medium knotless braids this Saturday at 10am"*
- Converts relative dates ("tomorrow", "this Saturday") into exact dates and times
- Sizes each appointment to the service length (1 to 6 hours)
- Checks Google Calendar before offering or booking any time
- Offers alternative times when a slot is taken
- Confirms details with the client before booking
- Creates the calendar event and sends a confirmation email
- Enforces business rules: opening hours, closed days, maximum duration, and 24 hours' notice

## Workflow

```
Chat Trigger ──► AI Agent
                   ├── Chat Model:  Groq (openai/gpt-oss-120b)
                   ├── Memory:      Simple Memory
                   └── Tools:
                        ├── check_calendar_availability   (Google Calendar: Availability)
                        ├── create_calendar_appointment   (Google Calendar: Create Event)
                        └── send_email_tool               (Gmail: Send Message)
```

| Node | Purpose |
|---|---|
| Chat Trigger | Receives the client's chat messages |
| AI Agent | Runs the conversation and decides which tool to call |
| Groq Chat Model | LLM powering the agent (`openai/gpt-oss-120b`) |
| Simple Memory | Keeps conversation context across messages |
| `check_calendar_availability` | Checks that the full appointment window is free |
| `create_calendar_appointment` | Books the appointment on Google Calendar |
| `send_email_tool` | Emails the client a confirmation |

Tool inputs are filled by the model at run time using n8n's `$fromAI()` function.

## Prompt design

The system prompt ([system-prompt.md](system-prompt.md)) is structured in five parts:

1. **Role** – assistant persona, business name, hours, and time zone
2. **Context** – services, durations, prices, and sample conversations
3. **Instructions** – a 10-step booking flow
4. **Tools** – when and how to call each tool
5. **Guardrails** – duration limits, business hours, no double-booking, explicit confirmation, tone, and privacy

## Example conversations

**Successful booking**
> **Client:** Hi, I'd like medium knotless box braids this Saturday at 10am
> **Priscilla:** Checks the calendar, collects name, email, and phone, confirms the details, books 10:00 AM–2:00 PM, and sends a confirmation email.

**Unsuccessful booking**
> **Client:** Can I book cornrows on Monday at 10am?
> **Priscilla:** We're closed on Mondays, but we're open Tuesday–Saturday from 9 AM–8 PM. Would a Tuesday work for you?

See the [screenshots](screenshots/) folder for the full runs.

## Setup

1. **Run n8n** (locally with `npm install -g n8n`, then `n8n`, or on n8n Cloud). Node.js 22 is recommended.
2. **Import the workflow:** in n8n, create a workflow, open the **⋯** menu, choose **Import from File**, and select `workflow.json`.
3. **Groq:** create a free API key at [console.groq.com/keys](https://console.groq.com/keys) and add it to the Groq Chat Model credential.
4. **Google Cloud:**
   - Create a project and enable the **Google Calendar API** and **Gmail API**.
   - Configure the OAuth consent screen and add your Gmail address as a test user.
   - Create an OAuth client ID (Web application) with the redirect URI
     `http://localhost:5678/rest/oauth2-credential/callback` (or your n8n Cloud URL).
   - Paste the Client ID and Secret into the Google Calendar and Gmail credentials in n8n, then click **Sign in with Google**.
5. Select your calendar in both Google Calendar tool nodes, save, and click **Open chat** to test.

> **Note:** Groq's free tier allows about 8,000 tokens per minute. Because each booking makes several model calls, wait a moment between messages, or keep the memory window small, to avoid rate-limit errors.

## Tech stack

n8n · Groq (`openai/gpt-oss-120b`) · Google Calendar API · Gmail API · OAuth 2.0 · Prompt engineering

## Author

**Priscilla Anim**
