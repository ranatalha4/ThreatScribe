# ThreatScribe

An n8n workflow that turns a security news link posted in Slack into a formatted, client-ready threat advisory.

## How it works

| # | Node | What it does |
|---|------|--------------|
| 1 | Slack Trigger | Listens for messages in the advisory channel |
| 2 | Extract URL & Metadata | Pulls the first URL, channel, thread and user from the message |
| 3 | Fetch & Extract Content | Scrapes the article (handles redirects, bot walls, `<article>`/`<main>` extraction, meta fallbacks) |
| 4 | Build AI Prompt | Builds an 8-section CTI advisory prompt with an auto-generated advisory ID (`TADV-DDMMYYYY`) |
| 5 | Prepare Groq Request / Groq AI Analyzer | Sends the prompt to Groq (`openai/gpt-oss-20b`) |
| 6 | Format HTML Advisory | Parses the Markdown into a 4-page styled HTML report (severity, CVSS, CVEs, IOCs, references) |
| 7 | Convert to Binary | Packages the HTML as a file |
| 8 | Build Email Template | Builds an HTML email body and a plain-text Slack summary |
| 9 | Upload Advisory to Slack / Send Advisory Message | Posts the report file and summary back into the original thread |

## Advisory sections

Executive Summary · Threat Overview · Technical Details · Affected Systems · Recommendations · MSSP Detection & Response · IOCs (only when found) · References

## Setup

1. In n8n, **Import from File** → `threatscribe-workflow.json`.
2. Create credentials and assign them to the nodes:
   - **Slack API** (bot with `channels:history`, `chat:write`, `files:write`) → Slack Trigger, Upload, Send Message nodes.
   - **Groq API** → Groq AI Analyzer node.
3. Replace placeholders:

| Placeholder | Where | Replace with |
|---|---|---|
| `YOUR_SLACK_CHANNEL_ID` | Slack Trigger, Upload Advisory to Slack | Your channel ID |
| `your-advisory-channel` | Slack Trigger, Send Advisory Message | Your channel name |
| `REPLACE_WITH_YOUR_BASE64_PNG_LOGO` | Format HTML Advisory | Base64 PNG of your logo |
| `YourOrg` / `yourorg` | Prompt, report and email text | Your organisation name |

4. Activate the workflow and post an article link in the channel.

## Notes

- The workflow is imported **inactive**; activate it after setup.
- The AI output is a draft — review CVSS estimates, IOCs and references before sending to clients.
- All credentials, IDs and branding in this export have been redacted.
