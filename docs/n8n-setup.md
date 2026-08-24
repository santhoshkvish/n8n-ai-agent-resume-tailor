# n8n setup

Three independent webhook workflows live in [`/n8n`](../n8n). Each is self-contained — import them one at a time.

## 1. Import

In n8n: **Workflows → Add workflow → Import from File**, and pick each JSON in turn:

- `analyze-match.json` — scores a resume against a job description, no rewriting.
- `tailor-resume.json` — rewrites the resume to match the job description, returns structured JSON.
- `email-resume.json` — emails a generated PDF + the job description to a given address.

Re-importing a workflow JSON in n8n always wipes credential assignments and resets it to inactive — this is expected, not a bug. Do steps 2–4 below after every import.

## 2. Connect credentials

**`analyze-match` and `tailor-resume`** each have a **Gemini Chat Model** node needing a *Google Gemini(PaLM) Api* credential:

- Get a free API key at [Google AI Studio](https://aistudio.google.com/) (no credit card required, 1,500 requests/day on the free tier).
- In n8n, create a new *Google Gemini(PaLM) Api* credential with that key, then select it on the Gemini Chat Model node in both workflows.
- Both are pinned to `models/gemini-3.5-flash-lite`. If that model is deprecated or unavailable on your account by the time you read this, check [Google's model list](https://ai.google.dev/gemini-api/docs/models) and update the `modelName` parameter — don't assume the pinned string still works; Google has quietly deprecated model strings before.

**`email-resume`** has an **HTTP Request** node ("Send via Gmail API") needing a *Gmail OAuth2* credential:

- Create it in n8n's credential manager, authenticate with the Google account you want sending mail.
- During the OAuth consent screen, only grant the minimal scope needed to send mail (`gmail.compose` / `gmail.send`) — not full inbox read/delete access.

## 3. Activate

Each workflow imports as **inactive**. Open each one and toggle **Active** in the top right, then save. A deployed frontend hitting an inactive webhook gets a plain 404 — if something isn't working after deployment, check this first before assuming a CORS or code issue.

## 4. Grab your webhook URLs

Each workflow's trigger node shows a **Production URL** once active — something like:

```
https://<your-instance>.app.n8n.cloud/webhook/analyze-match
https://<your-instance>.app.n8n.cloud/webhook/tailor-resume
https://<your-instance>.app.n8n.cloud/webhook/email-resume
```

Copy these into the three URL constants near the top of `index.html`'s `<script>` block: `ANALYZE_URL`, `TAILOR_URL`, `EMAIL_URL`.

## Known gotchas (already fixed in these exports, documented so you don't reintroduce them)

- **Binary conversion:** the resume PDF arrives as base64 in the webhook body. Don't use n8n's *Move Binary Data* node to convert it — its `sourceKey` option doesn't support dot-paths like `body.resumeBase64`. Both `analyze-match` and `tailor-resume` use a **Code node** with `this.helpers.prepareBinaryData()` instead.
- **Email attachments:** n8n's built-in *Send Email (SMTP)* node and *Gmail* node both have confirmed platform bugs where attachments silently never make it into the outgoing MIME message (see n8n GitHub issues #20832 and #20831). `email-resume` avoids both — it hand-builds a raw RFC 2822 MIME message in a Code node and sends it directly via the Gmail API's `messages.send` endpoint over HTTP Request with OAuth2.
- **Third-party ESPs:** Resend's free tier only delivers to the account's own signup address. Brevo can deliver to any recipient but Gmail silently drops mail relayed by unauthenticated third-party senders claiming to be from `@gmail.com` addresses. The Gmail API direct-send approach above is the one that's actually confirmed working end-to-end with a real recipient and a real PDF attachment.
- **CORS preflight:** each webhook trigger node has `allowedOrigins: "*"` set in its options, separate from the `Access-Control-Allow-Origin` header set later in the Respond to Webhook node. Both need to be present — the trigger-level setting controls the `OPTIONS` preflight response, the Respond to Webhook header controls the actual response.

## Self-hosting note

n8n Cloud trial instances expire, which breaks the production webhook URLs above. For a production deployment you don't want to depend on a trial, self-hosting n8n via Docker is the more durable path — see [n8n's Docker docs](https://docs.n8n.io/hosting/installation/docker/).
