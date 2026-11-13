# ORION (Gerald-)

A lightweight, mobile-first personal assistant interface. The app is a static HTML/CSS/JavaScript page and is compatible with GitHub Pages and Cloudflare Pages.

## What the browser app does

- Responsive dark glass interface and chat composer.
- Persistent conversations in browser local storage: create, revisit, and delete chats.
- Explicit, user-approved local memories with a manager to review and delete them. The app rejects common credential-like memory requests; do not store sensitive information.
- Built-in local tools for the current time/date, jokes, and recalling saved memories.
- Optional speech input using browser SpeechRecognition and spoken replies using installed system voices. Voice support differs by browser/device; microphone access generally requires HTTPS or localhost.
- Copy buttons for assistant messages, loading states, connection/error feedback, and a stop-speech control.

## Real AI: connect a private backend

The repository is deliberately a static frontend. It does **not** contain an AI model and does not fake general intelligence with canned answers. To answer open-ended questions, deploy a serverless function or API backend and configure its same-origin URL in **Settings**. For example, a Cloudflare Pages Function can be mounted at `/api/chat`; a same-origin service is also suitable on other hosts.

The frontend sends a POST request with JSON in this shape:

```json
{
  "messages": [
    { "role": "user", "content": "Help me plan my week." }
  ],
  "memories": ["Relevant user-approved memory"],
  "user": { "displayName": "Citro Miami" }
}
```

The endpoint should return JSON such as:

```json
{ "reply": "A concise, helpful answer..." }
```

The frontend also accepts the common `choices[0].message.content` response shape. Implement provider-specific API calls **on the server**, keep API credentials in the hosting provider's secret/environment-variable manager, validate inputs, limit request size/rate, and return only the assistant text. Never put a provider API key in `index.html`, browser storage, or the endpoint URL. The endpoint should apply your chosen model's system instructions for ORION's calm, natural personality, avoid inventing facts, and clearly state uncertainty.

Settings only accepts a same-origin backend URL to avoid sending conversation and memory context to an arbitrary third-party host. A blank endpoint keeps local mode enabled. When no endpoint is configured, unsupported general questions receive a clear setup explanation; the local time/date and simple tools still work. No paid dependency or API credential is required to deploy the frontend.

## Deploy

- **GitHub Pages:** enable Pages for the branch containing `index.html`.
- **Cloudflare Pages:** deploy the repository as a static site. Add a Pages Function at `/api/chat` if you want backend AI; store model credentials as Cloudflare secrets, not frontend variables.
- Microphone recognition may be unavailable when opening a local `file://` URL. Test voice on an HTTPS deployment or localhost and grant browser microphone permission.

## Data and limitations

Chat history, settings, and approved memories are kept in this browser's `localStorage`; they are not synchronized across devices. Clearing browser site data removes them. The frontend does not execute reminder notifications, search the web, analyze documents, or provide a live model without a configured backend. Speech quality is determined by the browser and operating-system voices installed on the user's device.

## Manual smoke-test checklist

1. Load `index.html`; verify the page renders and a new conversation appears.
2. Send text in local mode: time/date, joke, and memory recall should use local tools; an open-ended request should show an honest backend setup message.
3. Configure a same-origin endpoint implementing the documented request/response contract and verify a natural-language request returns a response. Verify a failing endpoint displays an error, not a fabricated answer.
4. Create two conversations, switch between them, reload, and confirm their history persists; delete one and confirm it is removed.
5. Add a memory in the manager, reload, confirm it persists, delete it, and try a credential-like entry to confirm it is rejected.
6. Test copy, stop speech, voice selection, and microphone permission on a browser that supports those APIs.
7. Check layout on a narrow phone viewport and browser console for runtime errors.

This repository is a frontend-only prototype; manual checks requiring a configured AI backend or browser microphone must be performed in the deployment environment where those services are available.
