# 🌱 TouchGrass AI
> **AI that gives you a reason to leave the screen — not another reason to stare at it.**

**TouchGrass AI** is an offline-first outdoor quest companion originally built for the **Hacktoberfest 2026 DEV Challenge — Week 1: “Touch Grass.”** It can use an **open-weight model through Ollama** to generate short, personalized outdoor quests. If AI generation is unavailable, a safe built-in quest is used instead. Once a quest is prepared, the user can save it locally, put their phone away, go outside, and complete the experience without depending on the internet.

**Project guides:** [Run locally](RUNNING.md) · [Deploy to Vercel](DEPLOYMENT.md) · [Deploy to Render](RENDER_DEPLOYMENT.md) · [Challenge submission draft](HACKTOBERFEST_SUBMISSION.md)

The core idea is simple:

```text
        User preferences
               ↓
     Open-weight local AI
               ↓
      Personalized quest
               ↓
         Save offline
               ↓
       📵 Go outside
               ↓
       Complete tasks
               ↓
        🌱 Earn XP
```

---

## 🏆 Hacktoberfest 2026 — Week 1 Project

### Challenge: **Touch Grass**

The Week 1 prompt asked builders to create a **new project with open-source AI at its core** that gets people **off the screen and into the real world**. The submission window ran October 5–11, 2026 and has ended.

TouchGrass is designed specifically around that idea:
- AI generates the **real-world activity**, not more screen content.
- The AI can run locally through **Ollama** using an open-weight model.
- Quests can be prepared before leaving home.
- Saved quests, progress, XP, streaks, and photo evidence are stored locally.
- The outdoor session can continue after the device loses internet access.
- The interface intentionally minimizes screen interaction once the quest begins.
- The home screen now uses a field-guide layout with a bundled woodland photo, and the service worker caches that image for offline viewing.

**Official challenge:**  
https://dev.to/challenges/hacktoberfest-week1-2026-10-05

**Official Week 1 rules:**  
https://dev.to/page/hacktoberfest-week1-2026-10-05-contest-rules

---

# 💡 Why TouchGrass?

Most AI products encourage people to stay inside an application:
- Chat with an AI
- Generate content
- Ask questions
- Consume recommendations
- Keep scrolling

TouchGrass reverses that loop.

Instead of:

```text
AI → more screen time
```

the product is designed around:

```text
AI → outdoor activity → screen away
```

The screen is only needed to **prepare the experience**.

The actual product experience happens outside.

---

# ✨ What It Does

## 1. Generate an Outdoor Quest

The user selects:
- ⏱️ Time: **15 / 30 / 60 minutes**
- 🚶 Activity:
  - Walking
  - Exploring
  - Nature
  - Photography
- 🎯 Difficulty:
  - Easy
  - Medium
  - Hard

The app sends these structured preferences to the local AI model.

Example:

```json
{
  "duration": 30,
  "activity": "exploring",
  "difficulty": "easy"
}
```

The model returns a structured quest containing a title and 3–5 tasks.

Example:

```text
🌳 Urban Explorer

30 minutes · Easy

□ Find something naturally occurring that is red.
□ Find a tree you have never noticed before.
□ Walk somewhere you have never walked before.
□ Sit somewhere for five minutes without using your phone.
```

---

# 🤖 Open-Weight AI at the Core

TouchGrass does not require a closed AI API for its local AI experience.

The application uses:

```text
TouchGrass
    ↓
Next.js API route
    ↓
Ollama
    ↓
Open-weight local model
    ↓
Structured outdoor quest
```

The current local development configuration uses:

```text
Ollama
Model: qwen2.5-coder:7b
```

The model can be changed through environment configuration.

### Why open AI matters here

The open/local approach is important because it can provide:

### 🔒 Privacy

Quest preferences do not have to be sent to a third-party AI provider.

### 📡 Offline-friendly preparation

The AI can run on the user's own machine, allowing quests to be prepared without relying on a remote AI service.

### 💰 Cost control

Local inference does not require a per-request hosted AI API for the local experience.

### 🔧 Model freedom

The model can be replaced or experimented with without redesigning the product around a single closed provider.

This is a central part of the project's Hacktoberfest story — the open model is not just an optional add-on; it is what generates the personalized quest.

---

# 📵 Offline-First Design

The most important product requirement is:

> **Prepare online. Go outside offline.**

After a quest has been prepared and saved, the outdoor experience does not need the AI API.

The app stores local state using **IndexedDB/Dexie**.

### Works offline

- ✅ Open saved quests
- ✅ Start and continue a quest
- ✅ View tasks
- ✅ Mark tasks complete
- ✅ Track progress
- ✅ Capture photo evidence
- ✅ Store evidence locally
- ✅ Calculate XP
- ✅ Update streaks
- ✅ View local history

### Local data

The following are stored locally:

```text
IndexedDB
├── Prepared quests
├── Quest results
├── User statistics
└── Photo evidence
```

Photos are stored as local blobs and are not intentionally uploaded to a cloud database.

The service worker caches the application shell and static assets; quest records and photos remain in IndexedDB rather than Cache Storage.

---

# 📸 Photo Evidence

Some generated tasks can require photo evidence.

Example:

```text
Task:
Find something naturally occurring that is red.

Requires photo:
YES
```

The user can:

1. Open the camera/photo picker.
2. Capture or select evidence.
3. Save it locally.
4. Review or remove it.
5. Complete the task once required evidence exists.

### Privacy by design

The current MVP does **not** send task photos to a remote vision API.

A future version can add local/open-weight vision verification:

```text
Photo
  ↓
Local vision model
  ↓
Evidence verification
  ↓
Task completed
```

This is intentionally a stretch goal rather than a dependency for the MVP.

---

# 🌱 Quest Completion

When all required tasks are completed:

```text
🌱 YOU TOUCHED GRASS

30:42 OUTSIDE

████████████████████ 100%

+120 GRASS XP

🔥 4 DAY STREAK
```

The app records the completion locally.

Tracked information includes:

- Quest
- Completion time
- Outdoor duration
- Completed tasks
- XP
- Streak information

---

# 🎮 Gamification

TouchGrass turns outdoor activity into lightweight progress.

### Grass XP

Example scoring:

| Activity | XP |
|---|---:|
| Easy quest | 50 |
| Medium quest | 100 |
| Hard quest | 150 |
| Optional bonus | +25 |
| Streak milestone | +50 |

The exact scoring can evolve as the project develops.

### Streaks

Completing outdoor quests contributes to a local streak.

The goal is not to create another addictive feed.

The goal is to make the user **close the app and go outside**.

---

# 🧠 Safety Constraints

Generated quests are validated before they are saved.

The AI is instructed not to suggest:

- Trespassing
- Traffic-related risks
- Climbing
- Water activities
- Approaching wildlife
- Restricted areas
- Entering buildings
- Unsafe physical activities

Photo tasks also avoid requesting photographs of people or private spaces.

The generated JSON is validated against an application schema before it becomes a saved quest.

If local AI generation fails or returns invalid output, TouchGrass falls back to a safe ready-made quest.

---

# 🏗️ Architecture

```text
┌──────────────────────────────────────────┐
│              TouchGrass PWA              │
│                                          │
│  Quest Builder → Quest Session → Stats   │
│         │              │                 │
│         └──────┬───────┘                 │
│                ↓                         │
│          IndexedDB / Dexie               │
│       quests · results · photos          │
└────────────────┬─────────────────────────┘
                 │
                 │ Quest generation only
                 ↓
        ┌───────────────────┐
        │ Next.js API Route │
        │   /api/quests     │
        └─────────┬─────────┘
                  ↓
             Ollama
                  ↓
       Open-weight local model
```

### Important separation

```text
AI generation
     ↓
Requires Ollama

Outdoor quest execution
     ↓
Does NOT require Ollama
     ↓
Does NOT require internet
```

This separation is what makes the offline-first experience possible.

---

# 🛠️ Tech Stack

| Technology | Purpose |
|---|---|
| Next.js | Application framework |
| React | UI |
| TypeScript | Type-safe application code |
| Tailwind CSS | Styling |
| Ollama | Local model runtime |
| Open-weight model | Quest generation |
| Dexie | IndexedDB persistence |
| IndexedDB | Local-first data |
| Service Worker | Offline application shell |
| PWA Manifest | Installable mobile experience |

---

# 🚀 Run Locally

## Prerequisites

- Node.js 20.9 or newer
- npm
- Ollama (optional; without it, quest generation uses a ready-made fallback)

Install Ollama:

https://ollama.com/download

Check that Ollama is available:

```bash
ollama --version
```

Pull the configured model:

```bash
ollama pull qwen2.5-coder:7b
```

---

## 1. Clone the repository

```bash
git clone https://github.com/Shashank0701-byte/TouchGrass.git
cd TouchGrass
```

## 2. Install dependencies

```bash
npm ci
```

## 3. Configure environment

Copy:

```bash
.env.example
```

to:

```bash
.env.local
```

The local setup can use the default Ollama endpoint:

```text
http://127.0.0.1:11434
```

---

## 4. Start the development server

```bash
npm run dev
```

Open:

```text
http://localhost:3000
```

### Testing offline behavior

The service worker is intentionally disabled in development.

For the offline/PWA behavior, build and run the production server:

```bash
npm run build
npm start
```

Then:
1. Open the application.
2. Generate a quest with Ollama.
3. Save the quest.
4. Disconnect Wi-Fi/mobile data.
5. Reopen the saved quest.
6. Complete the outdoor session.

---

# 🌐 Deployment

See:

- [DEPLOYMENT.md](DEPLOYMENT.md) for Vercel
- [RENDER_DEPLOYMENT.md](RENDER_DEPLOYMENT.md) for Render
- [RUNNING.md](RUNNING.md) for detailed local setup and offline testing

The hosted demo can use a hosted Ollama-compatible API configuration, while the strongest privacy/offline demonstration is the local setup where Ollama runs on the user's device.

**For the Hacktoberfest submission, clearly demonstrate the local/open-weight AI path rather than presenting a fallback-only hosted demo as the AI implementation.**

---

# 🧪 Project Status

### Current MVP

- [x] Outdoor quest generation
- [x] Open-weight local AI through Ollama
- [x] Structured AI output validation
- [x] Safe fallback quest generation
- [x] IndexedDB/Dexie local persistence
- [x] Offline quest execution
- [x] Quest progress tracking
- [x] XP and streak tracking
- [x] PWA/service-worker shell
- [x] Photo evidence
- [x] Local photo storage
- [x] Mobile-oriented experience

### Future ideas

- [ ] Local vision-model evidence verification
- [ ] Better location-aware quests
- [ ] Weather-aware quest generation
- [ ] Accessibility-aware activity generation
- [ ] More outdoor activity types
- [ ] Optional group quests
- [ ] More advanced local model support

---

# 🎥 Recommended Hacktoberfest Demo

A strong demo should show the entire product idea in a few minutes:

```text
1. Open TouchGrass
        ↓
2. Select:
   30 min + Exploring + Easy
        ↓
3. Generate with local open-weight AI
        ↓
4. Show the generated quest
        ↓
5. Save it for offline use
        ↓
6. Turn off internet
        ↓
7. Reopen the saved quest
        ↓
8. Complete tasks / add photo evidence
        ↓
9. Finish the quest
        ↓
10. Show XP + streak
```

The most important moment:
> **Turn the internet off and keep using the saved quest.**

That demonstrates the project's core idea better than a long code walkthrough.


# 🤝 Open Source & Attribution

If you build on existing open-source libraries, models, code, or ideas, preserve their licenses and give appropriate attribution.
Current major dependencies include:
- Next.js
- React
- Dexie
- Tailwind CSS
- Ollama
- Qwen2.5-Coder

Check the licenses of all dependencies/models before redistribution or deployment.
Do not present third-party code or ideas as original work.

---

# 📄 Project Documentation

Additional project documentation:
- [`docs/PRD (5).md`](docs/PRD%20(5).md) — product requirements and scope
- [`RUNNING.md`](RUNNING.md) — local setup and run instructions
- [`DEPLOYMENT.md`](DEPLOYMENT.md) — Vercel deployment guidance
- [`RENDER_DEPLOYMENT.md`](RENDER_DEPLOYMENT.md) — Render deployment guidance
- [`HACKTOBERFEST_SUBMISSION.md`](HACKTOBERFEST_SUBMISSION.md) — unpublished challenge post draft
- [`AGENTS.md`](AGENTS.md) — agent/development guidance
- [`CLAUDE.md`](CLAUDE.md) — project instructions

---


## Hacktoberfest 2026

Built for:
**Hacktoberfest Open-Source AI Challenge — Week 1: Touch Grass**
Challenge window: **October 5–11, 2026**

Official challenge:  
https://dev.to/challenges/hacktoberfest-week1-2026-10-05

---

**Built with open-source AI. Designed to make the screen less important.**
