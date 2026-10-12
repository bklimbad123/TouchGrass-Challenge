# Deploy TouchGrass AI to Render

This guide deploys TouchGrass as a Render **Node.js web service**. Use a web service rather than a static site because the app includes the server-side `/api/quests` route.

## Before you start

- Push this project to a GitHub repository that your Render account can access.
- Have a Render account.
- Decide whether quest generation should use a hosted Ollama-compatible API or safe built-in fallback quests. A Render service cannot access Ollama running at `127.0.0.1` on your personal computer.

The app requires Node.js 20.9 or newer. These instructions pin Node 22.22.0 for a predictable deployment.

## 1. Create a Render web service

1. Open the [Render Dashboard](https://dashboard.render.com/) and choose **New > Web Service**.
2. Connect GitHub if needed, then select the repository containing TouchGrass.
3. Configure the service:

   | Setting | Value |
   | --- | --- |
   | Name | Any available service name, such as `touchgrass-ai` |
   | Branch | The branch you want Render to deploy, usually `main` |
   | Root Directory | Leave blank for this repository, or use `.` if Render requires a value |
   | Language | `Node` |
   | Build Command | `npm ci && npm run build` |
   | Start Command | `npm start` |
   | Health Check Path | `/` |

4. Choose a compute plan. The Free plan is useful for a demo; see the limitations below.
5. Add the environment variables from the next section before creating the service.
6. Choose **Create Web Service**. Watch the **Events** or **Logs** view until the first deploy succeeds.

Render provides the deployed site at an HTTPS `onrender.com` URL. Open that URL to test the app.

## 2. Configure environment variables

Add these in the Render service's **Environment** settings. Do not upload `.env.local` or commit API keys to the repository.

### Option A: Hosted Ollama-compatible API

Use this when you want AI-generated quests on the deployed site:

| Key | Value |
| --- | --- |
| `NODE_VERSION` | `22.22.0` |
| `OLLAMA_ENABLED` | `true` |
| `OLLAMA_BASE_URL` | `https://ollama.com` or your hosted Ollama-compatible service URL |
| `OLLAMA_MODEL` | A model ID enabled for that service, for example `gemma4:31b-cloud` |
| `OLLAMA_API_KEY` | Your provider key; mark it secret in Render |

The app calls the Ollama native `/api/chat` endpoint. Set the base URL without a trailing slash and use a model ID available to that provider. The API key is read by the server only; never name it with a `NEXT_PUBLIC_` prefix. Check your provider's pricing and usage limits before enabling a public demo.

### Option B: Fallback quests only

If you do not have a hosted model endpoint, set:

| Key | Value |
| --- | --- |
| `NODE_VERSION` | `22.22.0` |
| `OLLAMA_ENABLED` | `false` |

With `OLLAMA_ENABLED=false`, the app skips Ollama and supplies a ready-made quest. You can omit the other `OLLAMA_*` values. This is the simplest configuration for getting the site online, but it does not demonstrate AI quest generation.

> Do not leave `OLLAMA_BASE_URL` as `http://127.0.0.1:11434` for a Render deployment. That address refers to the Render service's own container, not your computer. To use your own Ollama machine, it would need a secure, publicly reachable endpoint; do not expose an unauthenticated Ollama server to the internet.

## 3. Deploy and check logs

Render builds the selected branch and starts the app with `npm start`. After the deploy reports **Live**:

1. Open the HTTPS URL shown in the Render dashboard.
2. Confirm the home page and `/create` page load.
3. Create a quest. With hosted Ollama configured, confirm the generated quest is labeled as AI-generated; otherwise, confirm a fallback quest is returned.
4. Save the quest and reload the page. Confirm it remains in the same browser.
5. Review Render's deploy and runtime logs if the build or API request fails.

The API route returns a safe fallback quest if Ollama is unavailable or returns an invalid response. Model generation is server-side, so the API key should not appear in browser requests or client code.

## 4. Test offline behavior

The service worker is enabled for production deployments and disabled by `npm run dev`.

1. Open the deployed home page while online and let it finish loading.
2. Reload the page once so the service worker can take control.
3. Create and save a quest while online.
4. Use your browser's developer tools to switch to offline mode, reload the home page, and open the saved quest from the home page.

The cached home page, its cached static assets, and data already stored in the browser should be available offline. Offline navigation is limited to the home page; routes such as `/create` and quest generation are unavailable without a connection. Prepare and save a quest before going offline.

## Data and Render plan notes

- Quests, progress, photos, XP, and streaks are stored in browser IndexedDB, not on the Render server. Data belongs to that browser profile and site origin. Data from `localhost` does not automatically transfer to the Render domain.
- The app does not need a Render disk or database for its current local-first data model. Render's filesystem is not where user quests are saved.
- Render Free web services spin down after 15 minutes without inbound traffic. The next visit can take about a minute while the service starts again.
- Free services have an ephemeral filesystem and limited resources. Do not add server-side file storage expecting it to persist across restarts or deploys.
- Render runs the Next.js server and API route, but it does not run an open-weight model on the Free web-service instance. Use an external hosted Ollama-compatible endpoint for AI on the deployed site, or use fallback-only mode.

## Troubleshooting

- **Build fails with an unsupported Node version:** Confirm `NODE_VERSION` is set to `22.22.0` in Render and trigger a new deploy.
- **The site loads but quest generation falls back:** Verify `OLLAMA_ENABLED=true`, the hosted base URL, model ID, and API key. Check that the provider supports Ollama's `/api/chat` endpoint.
- **The deployed service cannot connect to local Ollama:** Replace the loopback URL with a hosted endpoint reachable from Render. `127.0.0.1` inside Render is not your laptop.
- **The first request is slow:** A Free service may have spun down after inactivity; allow time for it to start.
- **Local quests are not visible on the deployed site:** Browser storage is isolated by origin. Create or import data separately for the deployed origin; the project currently has no cloud sync.

## References

- [Render: Deploy a Next.js app](https://render.com/docs/deploy-nextjs-app)
- [Render: Web services](https://render.com/docs/web-services)
- [Render: Set your Node.js version](https://render.com/docs/node-version)
- [Render: Free instance limitations](https://render.com/docs/free)
- [Next.js: System requirements](https://nextjs.org/docs/app/getting-started/installation#system-requirements)
