# AI Email Triage and Auto-Response Agent

An n8n workflow that watches a Gmail inbox, uses an AI model (Groq) to classify each new email, and automatically replies to the ones that are safe to answer.

## What it does

- Picks up each new email as it arrives in Gmail.
- Asks an LLM to read the email and decide its category and whether an automatic reply is appropriate.
- Routes the email by category: **Urgent Mail**, **Meeting Scheduling**, **Promotion**, or **General Mail**.
- Checks a **"Safe to Reply?"** gate. Only emails that pass it get an automatic reply, sent in the same Gmail thread.
- Emails that fail the check are left alone, so a human can handle them.

## Workflow overview

```
Gmail Trigger → Get a message → Prepare Email Context → Basic LLM Chain → Parse AI Output
                                                              ↑
                                                       Groq Chat Model
→ Switch - Email Category → IF - Safe to Reply? ──true──→ Reply to a message
                                   └──false──→ (no action)
```

| Step | Node | Purpose |
|------|------|---------|
| 1 | **Gmail Trigger** | Starts the workflow when a new email arrives. |
| 2 | **Get a message** | Fetches the full email (sender, recipient, subject, body, IDs). |
| 3 | **Prepare Email Context** | Code node that cleans the email and builds the text sent to the AI. Keeps `emailId`, `threadId`, `from`, `to` and `subject` for later steps. |
| 4 | **Basic LLM Chain** | Sends the email to the AI model with instructions to classify it and draft a reply. |
| 5 | **Groq Chat Model** | The language model used by the chain. |
| 6 | **Parse AI Output** | Code node that turns the model's response into structured fields the workflow can use. |
| 7 | **Switch - Email Category** | Routes by category using rules: Urgent Mail, Meeting Scheduling, Promotion, General Mail. |
| 8 | **IF - Safe to Reply?** | Safety gate. The true branch continues to the reply, and the false branch stops. |
| 9 | **Reply to a message** | Sends the reply in the original Gmail thread using the saved `emailId` and `threadId`. |

## Requirements

- An n8n instance (self-hosted or cloud).
- A **Gmail** account connected to n8n through a Gmail OAuth2 credential.
- A **Groq** account and API key: https://console.groq.com

## Setup

1. **Import the workflow.** In n8n, open Workflows, click the ⋯ menu, choose **Import from file**, and select the workflow JSON.
2. **Connect Gmail.** Open the **Gmail Trigger**, **Get a message** and **Reply to a message** nodes and select your Gmail OAuth2 credential.
3. **Connect Groq.** Open the **Groq Chat Model** node, create a Groq API credential and paste your key.
4. **Review the AI prompt.** Open **Basic LLM Chain** and adjust the instructions to match your tone, business and categories.
5. **Review the safety rules.** Open **IF - Safe to Reply?** and check exactly which emails are allowed to receive an automatic reply.
6. **Test it** (see below), then activate the workflow.

## Testing safely

Auto-replying to real email can go wrong, so test before turning it on:

1. Send a few test emails to the connected inbox from a second account of your own.
2. Click **Execute workflow** in n8n and check each node's output, especially the category and the **Safe to Reply?** result.
3. Make sure urgent, sensitive or unclear emails go to the false branch.
4. Consider switching the **Reply to a message** step to create a **draft** instead of sending, until you trust the results.
5. Activate the workflow only after the test emails are handled the way you expect.

## Customizing

- **Categories:** edit the rules in **Switch - Email Category** and update the AI prompt so the model uses the same category names.
- **Reply style:** change the instructions in **Basic LLM Chain** (tone, length, signature, language).
- **Safety rules:** tighten or loosen the conditions in **IF - Safe to Reply?**, for example never auto-reply to Urgent Mail or to specific senders.
- **Model:** change the model in **Groq Chat Model** if you hit rate limits or want better quality.
- **Notifications:** connect the false branch to Slack, email or a Google Sheet so a human sees every email that was not auto-replied to.

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Workflow doesn't start | Check that the workflow is active and the Gmail credential is connected. |
| Replies go out as new emails, not in the thread | Make sure `threadId` and `emailId` are passed through to the **Reply to a message** node. |
| AI output fails to parse | Tighten the prompt to require a strict output format and check the code in **Parse AI Output**. |
| Wrong category chosen | Add clearer category definitions and examples to the AI prompt. |
| Groq errors or rate limits | Re-enter the API key, try a smaller model, or reduce how often the trigger polls. |

## Privacy and safety

- Email content is sent to a third-party AI provider (Groq). Do not use this on inboxes with confidential or regulated data unless your policies allow it.
- Keep a human in the loop for anything urgent, financial, legal or sensitive.
- Auto-replies can embarrass you if the AI gets it wrong, so review the logs regularly, especially in the first weeks.

## Built with

n8n · Gmail · Groq LLM
