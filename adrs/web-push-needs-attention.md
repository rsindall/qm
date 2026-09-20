# Web Push for agent attention

QM already tells you in the browser when an agent is waiting, but the page has to be open. On a phone, with Slack or other chat bridges turned off, there is no OS-level ping when an agent stops or needs an answer. This ADR proposes first-party Web Push so the same "needs attention" moments can reach the device lock screen without a third-party chat bot.

## Decision

Add optional Web Push delivery for attention events that already exist in the product (agent waiting on the user, hard stop / failure that needs a human). Keep chat connectors for teams that want them; Web Push is the built-in path for operators who use the web UI as the primary surface.

## Context

- Mobile browsers (notably Chrome on Android) support the Push API and service workers; iOS Safari support exists with constraints but is improving.
- Push payloads must stay small and non-sensitive: title + short body + deep link into the relevant session. No prompts, secrets, or tool output in the notification body.
- Subscriptions are per browser / device and tied to a signed-in user. Revoking access or logging out should drop subscriptions.
- Delivery is best-effort. Failed endpoints get pruned; the in-app attention UI remains the source of truth.

## Proposed shape

1. **Service worker** in the web app registers for push and shows a notification that opens the matching session URL.
2. **Subscription store** on the core: endpoint + keys per user/device, created after an explicit "enable notifications" affordance (permission prompt is user-initiated).
3. **VAPID** keys configured on the deployment (`WEB_PUSH_VAPID_PUBLIC_KEY` / `WEB_PUSH_VAPID_PRIVATE_KEY`, or equivalent). Public key is exposed to the client for `pushManager.subscribe`; private key stays server-side.
4. **Trigger points** reuse existing attention signals (for example: agent status moves to waiting / needs input / failed). Do not invent a parallel event bus only for push.
5. **Prefs**: per-user toggle for push on/off; optional quiet hours later if operators ask. Default off until the user opts in.

## Out of scope (this ADR)

- Replacing Slack, email, or other connectors.
- Rich media, action buttons that mutate agent state from the notification, or pushing full agent transcripts.
- Guaranteeing delivery on every OS / browser combination on day one (document supported surfaces in the feature notes).

## Alternatives considered

- **Keep chat bridges only.** Works for orgs that want Slack; fails for operators who deliberately run without chat and still need a lock-screen ping.
- **Email on attention.** Higher latency and noise; poor fit for "answer now" moments.
- **Native mobile apps.** Heavier; Web Push gets most of the value inside the existing PWA / mobile web surface.

## Success criteria

- After opt-in on a supported browser, a controlled "needs attention" event produces a lock-screen (or system) notification that deep-links to the session.
- No sensitive content in the payload beyond what the in-app attention chip already shows.
- Opt-out and logout clear subscriptions; invalid endpoints are removed without operator intervention.

## Open questions for implementers

- Exact list of attention statuses that fire push (waiting vs failed vs both).
- Whether multi-device users get one notification per subscription or a single "already notified" coalescing window.
- iOS install / home-screen requirements to document for operators.