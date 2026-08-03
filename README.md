# Facebook Comment-to-DM AI Automation

An n8n workflow that automatically detects Facebook comments asking for a resource, replies publicly, and sends a personalized DM — powered by an AI agent for intent detection.

## 🎯 Overview

When someone comments on a Facebook Page post:
1. An **AI Agent** analyzes the comment to understand the user's intent.
2. If the user is genuinely asking for a resource/link, they automatically receive a **personalized Facebook DM**.
3. A **public reply** is also posted under their comment.
4. If the comment is irrelevant, spam, or negative, the bot does nothing.

## 🧩 Tech Stack

- **[n8n](https://n8n.io/)** — workflow automation engine
- **Facebook Graph API** — webhooks, messaging, comment replies
- **Google Sheets** — lightweight campaign database
- **AI Agent (LangChain node in n8n)** — intent detection, powered by an OpenRouter-compatible LLM
- **Structured Output Parser** — enforces predictable JSON from the AI

## 🗺️ Workflow Diagram

```
Facebook Page Comment
        │
        ▼
   Webhook (POST)
        │
        ▼
If: event is "comment" on "feed"? ──false──► (ignored)
        │ true
        ▼
  Extract Comment Data
        │
        ▼
 Google Sheets: Read Campaigns
        │
        ▼
   AI Agent (Intent Detection)
        │
        ▼
   Structured Output Parser
        │
        ▼
     If: isValid? ──false──► (ignored)
        │ true
        ├──► Send DM (Graph API)
        └──► Send Public Reply (Graph API)
```

A second, parallel branch (Webhook GET) handles Facebook's one-time webhook **verification handshake** (`hub.mode`, `hub.verify_token`, `hub.challenge`).

## 📋 Prerequisites

- n8n instance (Cloud or self-hosted)
- Facebook Page (test page is fine)
- Meta Developer App with **Webhooks** and **Messenger** products configured
- An LLM API key compatible with n8n's Chat Model nodes (e.g., OpenRouter)
- Google account (for the Campaigns sheet)

## ⚙️ Setup

### 1. Import the workflow
Import `fb-comment-to-dm-automation.json` into your n8n instance (**Workflows → Import from File**).

### 2. Create the Google Sheet
Create a sheet named **Campaigns** with these columns:

| Campaign Name | Post ID | Trigger Keyword | Resource Link |
|---|---|---|---|
| Just For Fun | `{page_id}_{post_id}` | magic, wand, fun | https://example.com/resource |

> **Finding the Post ID:** Right-click the timestamp on a Facebook post → *Copy link address*. The number in the URL is the Post ID. The Graph API webhook payload typically sends it as `{page_id}_{post_id}`.

### 3. Set environment variables in n8n
| Variable | Description |
|---|---|
| `FB_VERIFY_TOKEN` | A secret string you choose, used to verify the webhook handshake |
| `FB_PAGE_ACCESS_TOKEN` | A Page Access Token generated via [Graph API Explorer](https://developers.facebook.com/tools/explorer/) with `pages_messaging`, `pages_read_engagement`, `pages_manage_metadata`, `pages_manage_posts` permissions |

### 4. Configure credentials
- Connect your **Google Sheets** account in the "Google Sheets - Read Campaigns" node.
- Connect your **OpenRouter** (or other LLM) credential in the Chat Model node.

### 5. Configure the Facebook Webhook
In your Meta App Dashboard → **Webhooks**:
- **Callback URL:** `https://YOUR_N8N_DOMAIN/webhook/fb-comment-webhook`
- **Verify Token:** must match `FB_VERIFY_TOKEN`
- Subscribe to the **`feed`** field under the **Page** product (not User).
- Subscribe your Page to the app via a `POST /{page-id}/subscribed_apps?subscribed_fields=feed` call in Graph API Explorer.

### 6. Activate the workflow
Toggle the workflow to **Active** in n8n so the production webhook URL is listening.

## 🧪 Testing

**Positive test case:** Comment on the configured post with a trigger keyword, e.g.:
> "Can you share the magic wand resource link?"

Expected: a DM is sent, and a public reply appears under the comment.

**Negative test case:** Comment something irrelevant, e.g.:
> "This page is useless."

Expected: no DM, no public reply, workflow completes without errors.

## 🤖 Role of AI in this Automation

The AI's primary responsibility is understanding user intent from natural language — determining whether a comment is genuinely requesting the campaign resource, and generating appropriate DM/public reply text. All other tasks (receiving webhook events, reading Google Sheets, evaluating conditions, calling the Graph API) are standard deterministic automation steps that don't require AI. This keeps AI usage scoped to where it adds real value: language understanding.

## ⚠️ Security Notes

- Never commit real access tokens, verify tokens, or Google Sheet IDs to version control.
- This repo uses `{{ $env.* }}` placeholders — set these in your n8n instance's environment variables, not in the workflow file itself.
- The Facebook Page Access Token used here is short-lived by default; for long-term production use, exchange it for a long-lived token.

## 📄 License

This project was built as a learning exercise. Feel free to fork and adapt.
