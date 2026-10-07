# AI Lead Qualification (n8n)

Scores every inbound lead from a website form in seconds, alerts sales about hot leads while confirming to the lead that someone will call today, thanks warm leads automatically, and logs everything to Google Sheets.

> **Portfolio sample.** Built to match a common request on freelance marketplaces ("I need a simple AI lead qualification workflow in n8n"). It is not client work.

## The business problem

Small sales teams answer every enquiry in the order it arrives. Serious buyers wait behind spam and tyre-kickers, and the best leads go cold. This workflow decides, within seconds of a form submission, which leads a person should call first.

## How it works

```
Form POST ─► Config ─► Normalize & Pre-score ─► Valid? ──no──► 400 response
                                                  │
                                                 yes
                                                  ▼
                                   AI Qualify (OpenAI-compatible)
                                                  ▼
                                           Combine Scores ─► 200 response
                                                  ▼
                                         Log to Google Sheets
                                                  ▼
                              hot ─► email alert to sales
                                   + confirmation to the lead (optional booking link)
                              warm ─► thank-you email to the lead
                              cold ─► logged only
```

1. **Lead Webhook** receives the form (`POST /webhook/lead-intake`).
2. **Config** holds every setting in one place: business description, AI model, score thresholds, emails, sheet URL.
3. **Normalize & Pre-score** cleans the input, rejects missing name/email/message, and gives a transparent rule score (0–100):
   - business email domain: +25
   - budget ≥ 5,000: +35, or ≥ 1,000: +20
   - company size 50+: +20, or 10+: +10
   - urgent timeline: +20, or near-term: +10
4. **AI Qualify** asks an LLM for an intent score (0–100), a category (sales / support / partnership / spam / other), a one-line summary and a suggested next step. It returns strict JSON.
5. **Combine Scores** blends the two: `final = 0.4 × rule + 0.6 × AI`. Spam is forced to 0. **If the AI call fails, the workflow still runs on the rule score alone**, so no lead is lost.
6. Tiers: **hot ≥ 70**, **warm ≥ 40**, otherwise **cold**.
7. **Hot leads are never left waiting:** sales gets an alert, and at the same moment the lead gets a "someone will contact you today" email, with a booking link if `booking_url` is set. Out-of-hours enquiries still get an instant reply.
8. Every logged lead gets a unique `lead_id`.

## Design choices worth noting

- **Resilient:** AI and Google Sheets failures do not stop the workflow (`continue on error`), and the rule score is a fallback.
- **Prompt-injection guard:** the system prompt tells the model to treat lead fields as untrusted data, and the AI output is validated (score clamped to 0–100, category checked against an allow-list).
- **No data leak to the form:** the public response is only `{"status":"received"}`; scores stay internal.
- **Provider-agnostic AI:** any OpenAI-compatible API works (OpenAI, Google Gemini, Groq, OpenRouter, a local Ollama server) by changing `ai_base_url` and `ai_model`. Tested live on Gemini's free tier (`ai_base_url` = `https://generativelanguage.googleapis.com/v1beta/openai`).
- **Input limits:** every field is trimmed and length-capped before it reaches the AI, which limits cost and abuse.

## Setup (self-hosted n8n)

1. In n8n: **Workflows → Import from File** → `workflow.json`.
2. Edit the **Config** node: business name, ideal customer, `ai_model` (use a current model name from your provider), thresholds, `sales_email`, `from_email`, `sheet_url`, and optionally `booking_url` (e.g. a Calendly link; leave blank to omit it).
3. Create credentials and attach them:
   - **OpenAI-type API key** (`AI Qualify` node). For Gemini or another OpenAI-compatible provider, paste that provider's key into an OpenAI credential and change `ai_base_url` and `ai_model` in Config.
   - **Google Sheets OAuth2** (`Log to Google Sheets`). Create a sheet with a tab named `Leads`; columns are filled automatically from the field names.
   - **SMTP** (all three email nodes).
4. Activate the workflow and point your form at `https://<your-n8n>/webhook/lead-intake`.

## Test it

```bash
curl -X POST https://<your-n8n>/webhook/lead-intake \
  -H "Content-Type: application/json" \
  -d @sample-leads/hot.json
```

`sample-leads/` has a hot, a warm, a spam and an invalid example.

## Running cost

One AI call per lead. With a small model this is a fraction of a US cent per lead (check your provider's current pricing). n8n self-hosted: free Community Edition.

## Test results

See [`TESTING.md`](TESTING.md).

## Ideas to extend

- Push hot leads to a CRM (HubSpot, Pipedrive) or Slack instead of email.
- Follow up warm leads that nobody has answered after a set number of days.
- Re-score leads weekly from the sheet as more data arrives.
