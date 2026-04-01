---
name: slack-message
description: Craft and send beautifully formatted Slack messages using the Slack MCP. Use this skill whenever the user wants to send a Slack message of any kind — replying to a technical question in a thread, notifying a teammate about PRs or code changes, explaining how a system works, or any communication that should look polished and professional. If the user mentions Slack, a colleague they want to message, a thread they want to reply to, or a PR they want to share — use this skill. Don't wait for the user to ask explicitly for "Slack formatting" — if they're composing a message for Slack, this skill applies.
---

# Slack Messenger

You are helping an engineering lead craft Slack messages that are genuinely pleasant to read. The goal is not just to convey information — it's to make colleagues feel like they're getting a well-considered, thoughtfully structured message rather than a wall of text or a lazy one-liner.

The two main patterns are:
1. **Technical Q&A replies** — answering questions about how a service or system works
2. **PR notifications** — alerting a teammate to PRs that need their attention

Both should make heavy use of the Slack Blocks API for structure, and emojis to add warmth and visual anchoring.

---

## Core Principles

**Lead with the answer.** Engineers are busy. Put the most important thing first, then support it with detail. A TL;DR at the top of a technical reply is always welcome.

**Use visual hierarchy.** Headers set the topic, dividers separate sections, context blocks add metadata. Don't make readers parse a wall of mrkdwn — use blocks to give the message shape.

**Emojis as signposts, not decoration.** A well-placed emoji tells the reader what kind of content follows before they read it. Use them consistently: 🔍 for explanations, 💡 for tips, ⚠️ for caveats, 🔗 for links, ✅ for completed items, 📋 for lists, 🔔 for notifications, 👀 for review requests. Don't overdo it — one emoji per section/point is the sweet spot.

**Keep it tight.** Long messages get skimmed. If there's a lot to say, use a bulleted list or numbered steps rather than prose. If it's really long, consider whether a thread reply + a link to docs is better than one giant message.

---

## Slack Blocks Cheat Sheet

Blocks are passed as a JSON array to the `blocks` parameter of `slack_send_message`. Always include a `text` parameter too (used as the notification preview / fallback).

### Header — for the title of the message
```json
{
  "type": "header",
  "text": { "type": "plain_text", "text": "🔔 PR Review Request", "emoji": true }
}
```

### Section — for main body text (supports mrkdwn)
```json
{
  "type": "section",
  "text": { "type": "mrkdwn", "text": "Hey *@username*! 👋 Got a couple of PRs for your eyes..." }
}
```

### Section with button accessory — for PR links
```json
{
  "type": "section",
  "text": {
    "type": "mrkdwn",
    "text": "*:pr: Fix auth token refresh* 🔐\n`feature/auth-refresh` → `main`\nAdds silent token refresh so users don't get logged out mid-session."
  },
  "accessory": {
    "type": "button",
    "text": { "type": "plain_text", "text": "Open PR", "emoji": true },
    "url": "https://github.com/org/repo/pull/123",
    "style": "primary"
  }
}
```

### Divider — visual separator between sections
```json
{ "type": "divider" }
```

### Context — small grey helper text (for metadata, timestamps, links)
```json
{
  "type": "context",
  "elements": [
    { "type": "mrkdwn", "text": "⏱️ No rush — but would love a review by EOD if possible" }
  ]
}
```

### Rich Text — most powerful, for technical explanations
```json
{
  "type": "rich_text",
  "elements": [
    {
      "type": "rich_text_section",
      "elements": [{ "type": "text", "text": "Here's how the auth flow works:" }]
    },
    {
      "type": "rich_text_list",
      "style": "ordered",
      "elements": [
        { "type": "rich_text_section", "elements": [{ "type": "text", "text": "User submits credentials" }] },
        { "type": "rich_text_section", "elements": [{ "type": "text", "text": "Server validates and issues a JWT" }] },
        { "type": "rich_text_section", "elements": [
          { "type": "text", "text": "Token is stored in " },
          { "type": "text", "text": "httpOnly", "style": { "code": true } },
          { "type": "text", "text": " cookie — not localStorage" }
        ]}
      ]
    },
    {
      "type": "rich_text_preformatted",
      "elements": [{ "type": "text", "text": "const token = jwt.sign({ userId }, secret, { expiresIn: '15m' });" }]
    }
  ]
}
```

### mrkdwn formatting reference
| Intent | Syntax |
|---|---|
| Bold | `*text*` |
| Italic | `_text_` |
| Inline code | `` `code` `` |
| Code block | ` ```code``` ` |
| Link | `<https://url|display text>` |
| User mention | `<@U012AB3CD>` |
| @here | `<!here>` |

---

## Template: Technical Q&A Reply

When someone asks how a service or system works, structure the reply to be immediately useful.

**Structure:**
1. Header with topic + emoji
2. TL;DR section (one sentence bottom line)
3. Divider
4. Rich text explanation (numbered steps if it's a flow, bullets if it's a list of things)
5. Code example if relevant (rich_text_preformatted)
6. Context block with related links if any

**Example blocks structure:**
```json
[
  {
    "type": "header",
    "text": { "type": "plain_text", "text": "🔍 How the event pipeline works", "emoji": true }
  },
  {
    "type": "section",
    "text": { "type": "mrkdwn", "text": "💡 *TL;DR:* Events are published to SQS, consumed by a Lambda worker, and written to DynamoDB — there's no direct DB writes from the API." }
  },
  { "type": "divider" },
  {
    "type": "rich_text",
    "elements": [
      {
        "type": "rich_text_section",
        "elements": [{ "type": "text", "text": "📋 Here's the full flow:", "style": { "bold": true } }]
      },
      {
        "type": "rich_text_list",
        "style": "ordered",
        "elements": [
          { "type": "rich_text_section", "elements": [{ "type": "text", "text": "API receives the request and publishes an event to SQS" }] },
          { "type": "rich_text_section", "elements": [{ "type": "text", "text": "Lambda worker polls SQS and processes the message" }] },
          { "type": "rich_text_section", "elements": [{ "type": "text", "text": "Processed data is written to DynamoDB" }] }
        ]
      }
    ]
  },
  {
    "type": "context",
    "elements": [
      { "type": "mrkdwn", "text": "🔗 <https://internal.docs/event-pipeline|Event Pipeline Docs>  •  ⚠️ Note: failed events are retried 3x then dead-lettered" }
    ]
  }
]
```

---

## Template: PR Notification

When alerting a colleague about PRs they need to look at, be specific about what you need from them and why.

**Structure:**
1. Header — e.g. "👀 Got a couple of PRs for you"
2. Section — brief intro with context about the work. If there's a linked ticket (Jira, Linear, GitHub issue), **include it inline here** as a natural part of the description — e.g. "These PRs are for <https://linear.app/...|ENG-123>". Don't bury it in the footer.
3. Divider
4. One section-with-button per PR, including: PR title + link, branch info, brief description, any specific areas to focus on
5. Context block — urgency/deadline, and any secondary reference links

**Tips:**
- If there are multiple PRs, say whether they're related and whether order matters — it helps the reviewer not waste time
- Be specific about what you want: "quick sanity check", "need approval to merge", "just FYI — no action needed"
- If there's a specific file or tricky bit of logic to focus on, call it out
- **Always ask for a ticket/issue link if the user hasn't provided one** — it's almost always relevant and saves the reviewer having to hunt for context

**Example blocks structure:**
```json
[
  {
    "type": "header",
    "text": { "type": "plain_text", "text": "👀 PR Review Request", "emoji": true }
  },
  {
    "type": "section",
    "text": { "type": "mrkdwn", "text": "Hey *@sarah* 👋 These two PRs are for the new checkout flow (<https://linear.app/team/issue/ENG-456|ENG-456>). They build on each other so ideally review in order." }
  },
  { "type": "divider" },
  {
    "type": "section",
    "text": {
      "type": "mrkdwn",
      "text": "*1️⃣ Add cart validation middleware* ✅\n`feature/cart-validation` → `main`\nValidates cart contents before checkout proceeds. Pay attention to the edge case handling around out-of-stock items (line 47 of `cartMiddleware.ts`)."
    },
    "accessory": {
      "type": "button",
      "text": { "type": "plain_text", "text": "Open PR", "emoji": true },
      "url": "https://github.com/org/repo/pull/201",
      "style": "primary"
    }
  },
  {
    "type": "section",
    "text": {
      "type": "mrkdwn",
      "text": "*2️⃣ Checkout summary page* 🛒\n`feature/checkout-summary` → `feature/cart-validation`\nDependency on the first PR — just needs a quick sanity check on the layout."
    },
    "accessory": {
      "type": "button",
      "text": { "type": "plain_text", "text": "Open PR", "emoji": true },
      "url": "https://github.com/org/repo/pull/202"
    }
  },
  {
    "type": "context",
    "elements": [
      { "type": "mrkdwn", "text": "⏱️ Targeting merge by Thursday — no rush today though 🙏" }
    ]
  }
]
```

---

## Sending the Message

Use `slack_send_message` with:
- `channel_id` — the channel or DM to send to
- `text` — a plain-text fallback (used in notifications, link previews, and accessibility). Make it meaningful, not just "see blocks below"
- `blocks` — the blocks array as JSON

If you're replying to a thread, include `thread_ts` with the timestamp of the parent message.

**Note on draft mode:** `slack_send_message_draft` may only accept a plain markdown `message` string rather than a `blocks` array. If the user wants to preview before sending, try the draft tool — but if it doesn't support blocks, fall back to composing the blocks in `slack_send_message` directly. You can always show the user the full blocks JSON and ask them to confirm before you send.

### Finding recipients
- Use `slack_search_users` to find a user by name if you don't have their ID. If the search fails or the user can't be found, ask the user to provide the channel ID or DM ID directly rather than guessing.
- Use `slack_search_channels` to find a channel
- Use `slack_read_thread` to read a thread before composing a reply (so you can reference what was asked)

### Gathering context
Before sending, think about what you know and what you might need:
- If replying to a question, read the thread first with `slack_read_thread` to understand the full context
- If the user mentions a PR, ask for the URL if they haven't provided it
- If the user hasn't specified a recipient, ask who they want to message
