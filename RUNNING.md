# Run TouchGrass AI Locally

This guide covers starting the app on your computer, enabling AI quest generation with Ollama, and testing the offline/PWA behavior.

## Prerequisites

- Node.js and npm
- A modern browser such as Chrome, Edge, or Firefox
- Ollama only if you want AI-generated quests; the app can run without it and will use a ready-made fallback quest when generation is unavailable

Check that Node.js and npm are installed:

```bash
node --version
npm --version
```

## 1. Open the project directory

If you have not cloned the repository yet, clone it and enter its directory:

```bash
git clone <repository-url>
cd <repository-directory>
```

If the project is already on your computer, open a terminal at the project root, where `package.json` is located.

## 2. Install dependencies

Run this from the project root:

```bash
npm ci
```

## 3. (Optional) Set up Ollama for AI quests

Install Ollama by following the instructions for your operating system at [ollama.com/download](https://ollama.com/download). Then download the model configured for this project:

```bash
ollama pull qwen2.5-coder:7b
```

Make sure the Ollama service is running. On systems where it does not start automatically, start it in a separate terminal:

```bash
ollama serve
```

The app's defaults point to `http://127.0.0.1:11434` and use `qwen2.5-coder:7b`. To customize these settings, copy `.env.example` to `.env.local` and edit the values:

```bash
cp .env.example .env.local
```

Available server-side settings are `OLLAMA_BASE_URL`, `OLLAMA_MODEL`, `OLLAMA_ENABLED`, and the optional `OLLAMA_API_KEY` for a hosted Ollama API or protected proxy. Do not add the API key to a `NEXT_PUBLIC_` variable or commit `.env.local`. Set `OLLAMA_ENABLED=false` to use fallback quests without contacting Ollama.

## 4. Start the development server

From the project root, run:

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) in your browser. Leave the terminal running while using the app; press `Ctrl+C` to stop the server.

You can create and save quests from the app. Quests, progress, and evidence are stored in that browser's local IndexedDB, so they are local to the browser and origin. If Ollama is not running or a model request fails, quest generation uses a ready-made fallback.

## 5. Test offline and PWA behavior (optional)

The service worker is disabled in development. To test offline behavior, stop the development server with `Ctrl+C`, then build and run the production version:

```bash
npm run build
npm start
```

Open [http://localhost:3000](http://localhost:3000) while online and allow the page to finish loading. Reload once so the service worker can take control, then create or open a saved quest. Use the browser's developer tools to switch to offline mode and reload the app. The app shell and data already saved in IndexedDB should remain available. AI quest generation requires Ollama connectivity; save a quest before going offline to continue it without a network connection.

Press `Ctrl+C` in the terminal to stop the production server.

## Common issues

- **Port 3000 is already in use:** Next.js will report another available port in the terminal; open the URL it prints.
- **Quest generation returns a fallback:** Check that Ollama is running, the model was pulled, and `OLLAMA_BASE_URL` and `OLLAMA_MODEL` match your setup. The app remains usable with fallback quests.
- **Changes to `.env.local` do not take effect:** Stop and restart the Next.js server after changing environment variables.
- **Saved data is missing:** Browser data is separated by origin and browser profile. Data stored on `localhost` is not shared with a deployed site or another browser.

For hosted deployment steps, see [DEPLOYMENT.md](DEPLOYMENT.md).