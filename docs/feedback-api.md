# Feedback & Analytics API

The app can send **feedback** (bug reports) and **minimal analytics** to a backend you host. Set the base URL in `KnowledgeCache/Feedback/FeedbackConfig.swift` (e.g. `https://your-server.com`).

## 401 when sending (Vercel)

If the app shows **"Send failed: Server returned 401"**, the deployment URL is likely a **preview** URL protected by Vercel Deployment Protection. Fix it by either:

- Using your **production** URL in `FeedbackConfig.baseURL` (e.g. `https://feedback-server-<your-team>.vercel.app` from the Vercel project overview), or  
- In Vercel: **Project → Settings → Deployment Protection** → set protection to **"Only Preview Deployments"** so production is public.

## Dashboard UI (Vercel deployment)

If you use the **feedback-server** deployed on Vercel:

- Open your deployment URL in a browser (e.g. **https://feedback-server-xxx.vercel.app**).
- You’ll see a **dashboard** with:
  - **Analytics** – last 200 events (time, event, app version, saves count).
  - **Feedback / bug reports** – last 200 reports (time, type, message, optional email).
- Data is stored in **Vercel Blob**. In the Vercel project: **Storage** → **Blob** → create a store, then redeploy so the API can persist and display data.

## When offline

- **Feedback**: Stored locally in `Application Support/KnowledgeCache/pending_feedback.json`. When the device is back online, the app automatically POSTs each pending item to your server and removes it from the queue.

## Endpoints

### POST /api/feedback

Receives bug reports and feedback (one per request).

**Request body (JSON):**

```json
{
  "id": "uuid",
  "message": "User's message",
  "email": "optional@email.com",
  "type": "bug",
  "app_version": "1.0 (2)",
  "os_version": "14.2.1",
  "timestamp": "2025-02-10T12:00:00.000Z"
}
```

- `type`: `"bug"` or `"feedback"`
- Respond with **200** so the app removes the item from the pending queue. Any other status leaves it queued for retry.

### POST /api/analytics

Receives minimal usage data (opt-in, throttled to about once per day per install).

**Request body (JSON):**

```json
{
  "event": "session",
  "app_version": "1.0 (2)",
  "saves_count": 5,
  "timestamp": "2025-02-10T12:00:00.000Z"
}
```

- Respond with **200** so the app records the send and throttles the next one.

## Hosting

You can host a small backend anywhere (e.g. Vercel serverless, Cloudflare Worker, or a minimal Express/Flask app) that:

1. Accepts `POST /api/feedback` and stores the body (e.g. to a DB or file) and/or forwards it to your email.
2. Accepts `POST /api/analytics` and stores or aggregates the body.

Example (Node/Express) stub:

```js
app.post('/api/feedback', express.json(), (req, res) => {
  const { id, message, email, type, app_version, os_version, timestamp } = req.body;
  // save to DB or send email
  res.status(200).end();
});
app.post('/api/analytics', express.json(), (req, res) => {
  const { event, app_version, saves_count, timestamp } = req.body;
  // store or aggregate
  res.status(200).end();
});
```

Ensure CORS allows requests from your app if needed (macOS app uses URLSession; same-origin doesn’t apply, but if you add a web dashboard, allow your domain).
