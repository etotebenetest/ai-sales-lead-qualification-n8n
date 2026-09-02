# AI Sales Lead Qualification & Follow-up — n8n

Production-style n8n automation for inbound sales email qualification, CRM updates, AI-assisted follow-up, manager notifications, and error handling.

## Workflow Architecture

![AI Sales Lead Qualification & Follow-up workflow](workflow-architecture.png)

## Business problem

The workflow automates the first layer of inbound sales processing: reading email, checking CRM, qualifying the lead, updating records, recommending a next action, and sending appropriate follow-up.

## Stack

- n8n
- Gmail
- Google Sheets
- Groq
- `openai/gpt-oss-20b`
- Structured Output Parser
- Telegram

## Architecture

```text
New Email Received
        |
Find Lead by Email
        |
   Lead Exists?
     /       \
   YES       NO
    |         |
Requalify   Analyze New Lead
Existing      |
Lead       Is Sales Lead?
    |          |
Update         | true
Existing       v
Lead       Create New Lead
    |          |
Is Hot?    Is Qualified?
 /   \         |
yes   no       +--> Reply to New Lead
 |     |       +--> Notify Manager
TG   Is Warm?
       |
      yes
       |
 Reply to Warm Lead
```

## Existing leads

The AI receives existing CRM context plus the latest email and returns structured data:

- `leadScore`
- `qualification`
- `summary`
- `recommendedAction`
- `suggestedResponse`

Scoring:
- 80–100 = hot
- 50–79 = warm
- 0–49 = cold

Hot leads notify the manager in Telegram. Warm leads receive an AI-drafted follow-up. Cold leads are updated in CRM without outbound follow-up.

## New leads

For a sender not found in CRM, the AI extracts available name, company and budget, scores the lead, drafts a response, and determines `isLead`.

`isLead = false` filters newsletters, promotions, automated notifications, spam, unrelated receipts/invoices, cold outreach selling to the company, and other messages without realistic buying intent.

Real leads are appended to Google Sheets. Qualified new leads (not cold) trigger both a Telegram manager notification and an email reply.

## Expected CRM columns

`Lead ID`, `Name`, `Email`, `Company`, `Budget`, `Message`, `Source`, `AI Score`, `Qualification`, `Summary`, `Status`, `Created At`, `Last Contact`, `Recommended Action`, `Suggested Response`.

Status mapping:

```text
hot  -> ReadyForDeal
warm -> InProcess
cold -> Cancelled
```

## Reliability

Important Google Sheets and AI steps use retry-on-failure behavior.

In production, a separate n8n Error Workflow can be connected:

```text
Error Trigger -> Telegram
```

It alerts the manager when the main workflow fails. Its installation-specific workflow ID is intentionally removed from this public export.

## Setup

1. Import `AI_Sales_Lead_Qualification_Followup_SANITIZED.json` into n8n.
2. Connect Gmail, Google Sheets, Groq and Telegram credentials.
3. Create a Google Sheet with the expected CRM columns.
4. Replace `YOUR_GOOGLE_SHEET_ID`.
5. Replace `YOUR_TELEGRAM_CHAT_ID`.
6. Replace `your-email@example.com` in the Gmail trigger filter.
7. Set the workflow timezone to the actual business timezone.
8. Optionally create and connect a separate Error Workflow.
9. Publish/activate and test with controlled emails.

## Test cases

1. Existing lead shows strong buying intent -> hot -> Telegram alert.
2. Existing lead asks for more information -> warm -> follow-up email.
3. Existing lead declines -> cold -> no outbound follow-up.
4. New qualified sender -> new CRM row + Telegram + email.
5. Newsletter/automated notification -> `isLead = false` -> no CRM row.
6. Safe forced production failure -> Error Workflow -> Telegram alert.

## Security

The public export removes/replaces credential IDs, workflow/instance IDs, personal Telegram Chat IDs, personal email addresses, Google Sheet IDs, cached Sheet URLs, and the installation-specific Error Workflow ID.

Never commit OAuth tokens, API keys, real customer data, or private CRM screenshots. Always review an n8n export before publishing it.

## Portfolio skills demonstrated

Event-driven automation, Gmail integration, CRM lookup/update, branching, LLM classification and extraction, structured AI output, automated follow-up, Telegram notifications, retry handling, production error handling, and filtering of non-sales inbound email.
