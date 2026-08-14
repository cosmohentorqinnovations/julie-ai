# J.U.L.I.E — julie.cosmohentorq.com

A live, browser-based chatbot for **J.U.L.I.E** by Cosmohentorq Innovations.
Static front-end (`index.html`) + a Cloudflare Pages Function that proxies chat
to an OpenAI-compatible backend.

## Why a proxy?

The AI **model name, backend URL and API key live only in the server-side
function** (`functions/api/chat.js`), read from environment variables. The
browser only ever calls the same-origin `/api/chat` endpoint. This means:

- the key is never shipped to the browser (no "view source" leak),
- the model name / provider / endpoint are never exposed to users,
- J.U.L.I.E's system prompt also instructs it never to disclose them.

Never hardcode the key in `index.html` or commit it to git.

## Project layout

```
index.html              # chat UI (launched state)
functions/api/chat.js   # Pages Function — POST /api/chat, streams the reply
.dev.vars.example       # template for local secrets (copy to .dev.vars)
```

## Environment variables (set in Cloudflare, not in the repo)

| Variable            | Required | Example                              |
|---------------------|----------|--------------------------------------|
| `LITELLM_BASE_URL`  | yes      | `https://<endpoint>/llmops-litellm/v1` |
| `LITELLM_API_KEY`   | yes      | `sk-…` (mark as a **secret**)        |
| `LITELLM_MODEL`     | yes      | the model name to serve              |
| `JULIE_TEMPERATURE` | no       | `0.7` (default)                      |

> The function calls `POST {LITELLM_BASE_URL}/chat/completions`. If your proxy
> serves the OpenAI route under `/v1`, include `/v1` in `LITELLM_BASE_URL`.
> If it serves it at the root, omit `/v1`.

## Deploy to Cloudflare Pages

1. Push this repo to GitHub.
2. Cloudflare dashboard → **Workers & Pages → Create → Pages → Connect to Git**,
   select this repo.
3. Build settings: **Framework preset: none**, **Build command: (empty)**,
   **Output directory: `/`**.
4. **Settings → Environment variables** → add the three variables above.
   Mark `LITELLM_API_KEY` as an encrypted **Secret**. Save and redeploy.
5. **Custom domains** → add `julie.cosmohentorq.com`. Cloudflare provisions the
   TLS cert and the `CNAME` automatically (the domain must be on this
   Cloudflare account).

## Local development

```bash
npm i -g wrangler
cp .dev.vars.example .dev.vars   # fill in real values (gitignored)
wrangler pages dev .
```

Open the printed localhost URL and chat.

## Note on the backend endpoint

The Pages Function runs on Cloudflare's public edge, so `LITELLM_BASE_URL` must
be reachable from the public internet. An internal / VPN-only endpoint will not
work from Cloudflare — expose a public gateway (with auth + rate limiting) or
host the proxy where it can reach the backend.
