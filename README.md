# n8n Automation Portfolio

Business automations built with [n8n](https://n8n.io) and AI, each modelled on real requests posted by small businesses on freelance marketplaces. Every workflow is importable, documented, and tested on self-hosted n8n.

> These are portfolio samples, not client deliverables.

| # | Workflow | Business problem it solves | Key skills | Status |
|---|---|---|---|---|
| 01 | [AI Lead Qualification](01-ai-lead-qualification/) | Sales teams waste time on spam and low-intent enquiries while hot leads go cold | Webhooks, LLM scoring, fallback logic, Google Sheets, email routing | Tested live: Gemini, Google Sheets, Gmail (n8n 2.41.7) |
| 02 | [AI Invoice & Receipt Extractor](02-ai-invoice-extractor/) | Supplier bills are typed in by hand, mistyped, paid twice or lost in an inbox | Gmail trigger, upload form, AI document reading (PDF + photo), validation rules, duplicate detection, human review | Tested live: Gemini (PDF + photo), Google Sheets, SMTP (n8n 2.41.7) |
| 03 | [AI Bakery Order Assistant](03-ai-bakery-order-assistant/) | A home baker loses orders and evenings to repetitive questions, missed details and overbooked days | Web chat + Telegram, AI understanding with rule-based pricing and capacity checks, conversation memory in Sheets, owner hand-off | Tested live with Claude: order taken, saved to Google Sheets, owner emailed (n8n 2.41.7) |

## What every workflow includes

- `workflow.json`: import directly into n8n (no credentials included)
- `README.md`: the problem, how it works, setup steps, running cost
- `TESTING.md`: what was tested, how, and what was not
- Sample input data

## Standards I build to

- **Fails safely:** if an AI or third-party service is down, the workflow degrades instead of losing data.
- **Treats user input as untrusted:** validation, length limits, and prompt-injection guards before anything reaches an AI model.
- **One place to configure:** all business settings live in a single `Config` node.
- **AI where it helps, rules where it matters:** the AI reads and understands; prices, totals and approvals are decided by code.

## Contact

Open to freelance automation work (n8n, Zapier, AI integrations). Reach me via my Upwork or Fiverr profile.
