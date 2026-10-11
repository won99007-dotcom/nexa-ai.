# NEXA AI backend setup

The NEXA demo now sends messages to `POST /api/chat` on the Express API server. The server calls the OpenAI Chat Completions API; the API key stays on the server and is never bundled into frontend code.

## Configure

1. Set `OPENAI_API_KEY` in the environment where `@workspace/api-server` runs.
2. Optionally set `OPENAI_MODEL` (defaults to `gpt-4.1-mini`).
3. Keep `OPENAI_BASE_URL` unset to use OpenAI, or set it to a compatible provider's full chat-completions endpoint.
4. Set `CORS_ALLOWED_ORIGINS` to a comma-separated list of exact frontend origins when the frontend and API use different origins, for example `https://nexa.example.com`. Do not use `*` for a public production API.
5. Set `VITE_API_BASE_URL` for the frontend only if the API is hosted on a different origin, for example `https://api.example.com`. Do not put secret keys in any `VITE_` variable.

## Run locally

Start the API server with `PORT=3001` and start the Vite frontend with its required `PORT` and `BASE_PATH` environment variables. If frontend and API use different origins, set `VITE_API_BASE_URL` to the API origin. The API server currently requires `PORT` at startup.

## Endpoint

`POST /api/chat`

Request JSON:

```json
{ "message": "Help me plan a project" }
```

Successful response JSON:

```json
{ "reply": "Let's define the outcome first..." }
```

The route validates input, limits messages to 8,000 characters, times out upstream calls after 45 seconds, and returns sanitized errors. The API also limits each IP address to 20 chat requests per 15-minute window and limits JSON request bodies to 16 KB. Cross-origin browser requests are allowed only for origins listed in `CORS_ALLOWED_ORIGINS`.

## Production security notes

- The included rate limiter is in-memory and applies per IP address per 15-minute window per running server process. It resets on restart and does not coordinate across multiple instances. For multi-instance production deployments, replace it with a shared store such as Redis.
- If the API runs behind a reverse proxy, configure Express `trust proxy` narrowly for your actual proxy infrastructure; do not blindly trust forwarded IP headers, because clients could evade IP limits.
- IP-based limits are a baseline, not a replacement for authentication, per-user quotas, provider spending limits, monitoring, and a privacy notice. User messages are sent to the configured AI provider.
