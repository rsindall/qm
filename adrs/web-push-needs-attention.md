# Web Push when an agent needs you

QM already shows "needs attention" in the browser, but that only helps if the tab is open. With chat bridges turned off, there's no lock-screen ping when an agent stops or is waiting on an answer.

Ask: add optional first-party Web Push for those existing attention moments. Opt-in only. Small payload (title, short body, deep link into the session). No prompts, secrets, or tool output in the notification. Reuse the attention signals you already have; don't invent a parallel event bus. VAPID keys as deploy config. Chat connectors stay for teams that want them; this is for people who live in the web UI.

Out of scope for now: replacing Slack/email, rich media, action buttons that mutate agent state, pushing transcripts, or promising every OS/browser on day one.

If you're aligned, happy for you to implement. Open calls on your side: which statuses fire (waiting vs failed vs both), multi-device coalescing, and what to document for iOS home-screen install.
