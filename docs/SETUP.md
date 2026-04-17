# Approach 1: Tools-Only — Setup Guide

Step-by-step instructions to set up the Control Tower agent on
HappyRobot using the tools-only approach.

---

## Step 1: Create the workflow

1. Go to `platform.happyrobot.ai` → **Workflows** → **New Workflow**
2. Name it "Control Tower — Supply Chain Monitor"
3. Note the auto-generated slug (you'll need it for API triggers)

## Step 2: Choose your trigger

Pick one depending on your demo channel:

| Channel | Trigger type | Notes |
|---|---|---|
| **Voice (inbound)** | Inbound Voice Agent | Needs a phone number in Assets > Telephony |
| **SMS** | Inbound Text Agent (SMS) | Needs a Twilio number |
| **Web chatbot** | Inbound Text Agent (Chatbot) | Just register your domain — easiest for demo |
| **API-triggered** | Incoming Hook | Test via curl, no channel needed |

**For the builder night demo**, chatbot or API hook is fastest (no
telephony setup).

## Step 3: Store environment variables

Go to **Settings > Environment Variables** and add:

| Variable | Value |
|---|---|
| `CT_API_KEY` | `16042026` |
| `CT_HAPPYROBOT_TOKEN` | `789456777cdqde` |

These will be referenced in the tool headers.

## Step 4: Create the tools

Go to **Tools > Creating Tools** and create 3 HTTP tools. Copy the
definitions from [TOOLS.md](TOOLS.md).

| Tool name | Method | Endpoint |
|---|---|---|
| `get_raw_shipments` | GET | `https://plan.supply-science.com/api/v1/monitoring/tools/raw-shipments` |
| `get_recent_events` | GET | `https://plan.supply-science.com/api/v1/monitoring/tools/recent-events` |
| `post_flash_report` | POST | `https://plan.supply-science.com/api/v1/monitoring/reports` |

## Step 5: Configure the agent node

1. Add a **Voice Agent** or **Text Agent** node after the trigger
2. Paste the system prompt from [PROMPT.md](PROMPT.md)
3. Attach the 3 tools created in Step 4
4. Configure LLM model (e.g., GPT-4.1 or Claude Sonnet)
5. Set initial message (voice):
   ```
   Hello, this is the Supply Chain Control Tower assistant. Which city
   are you calling about, and how can I help you today?
   ```
   Or for chatbot:
   ```
   Welcome to the Supply Chain Control Tower. Which city would you like
   to check, and what would you like to know?
   ```

## Step 6: Generate a test record

Send a test payload to populate the `@variable` picker:

```bash
curl -X POST https://platform.happyrobot.ai/hooks/<SLUG>/draft \
  -H "Content-Type: application/json" \
  -d '{"caller_city": "Shanghai", "question": "Why are my shipments late?"}'
```

## Step 7: Test

- **Voice**: use the platform's **Test Call** button or call the number
- **Chatbot**: open the widget URL and type a question
- **API**: trigger via the hook URL

Try these test scenarios:
1. "I'm the logistics manager for Shanghai. Why are my shipments always
   late?" → [EXAMPLE_1](../EXAMPLE_1.md)
2. "Are there any orders en route to Shanghai right now? Any behind
   schedule?" → [EXAMPLES_2](../EXAMPLES_2.md)
3. "I'm in Riyadh. The stores say all deliveries were late the last 2
   days." → [EXAMPLES_3](../EXAMPLES_3.md)

## Step 8: Publish

Once tested, publish the workflow version and switch to production
environment.
