---
title: "TouchGrass AI: An Offline-First Quest to Get Outside"
published: false
tags: devchallenge, hf26challenge
---

*This is a submission for the [Hacktoberfest Open-Source AI Challenge Week 1: Touch Grass](https://dev.to/challenges/hacktoberfest-week1-2026-10-05)*

## What I Built

I built TouchGrass AI, an outdoor quest companion for people who want a nudge away from their screens and an easy plan for spending time outside. Choose a duration, activity, and difficulty, then get a short quest with practical tasks. The app saves quests, progress, photos, and earned XP in the browser so the session can continue without a connection.

The AI is there to help someone get ready, not to keep them chatting. After the quest is prepared, the intended next step is to put the phone away and go outside.

## Demo

<!-- Replace this placeholder with a deployed app URL or a short demo video before publishing. -->

**Live demo:** [Add deployed URL]

**Video walkthrough:** [Add video URL, if available]

## Code

The source code is available on [GitHub](https://github.com/Shashank0701-byte/TouchGrass).

The local setup and run instructions are in [`RUNNING.md`](RUNNING.md). Hosted deployment details are in [`DEPLOYMENT.md`](DEPLOYMENT.md).

## How I Built It

TouchGrass is built with Next.js, React, and TypeScript. Quest generation runs through a server-side route that calls Ollama's local API with the open-weight `qwen2.5-coder:7b` model. The prompt asks for a constrained JSON quest, and the app validates the response and rejects unsafe or malformed tasks. If Ollama is unavailable or a response fails validation, a safe built-in quest is used instead.

Dexie stores quests, progress, completion history, XP, streaks, and photo evidence in browser IndexedDB. The production service worker caches the app shell and static assets, so a saved quest can be opened and completed offline. AI generation itself needs the Ollama service; completing a quest already saved on the device does not.

The project does not require an account or cloud database for its local-first flow. For a hosted deployment, the server can instead be configured with an Ollama-compatible endpoint and a server-only API key.

## Why Does Open Innovation Matter?

An outdoor companion should still be useful when a user leaves Wi-Fi behind. With an open-weight model served locally by Ollama, quest generation can happen on the user's own machine without sending their preferences to a closed, remote AI API. The model choice and prompt are inspectable and replaceable, and the saved quest remains available even when the model or network is not.

This also keeps the AI's role bounded: it produces a draft, while validation enforces the app's task shape and safety constraints. A closed hosted model could generate the text, but it would make generation depend on a third-party service and an active connection. The open local path makes privacy and offline use practical parts of the product rather than optional promises.

## My Agent Session

<!-- Optional: add a DevRelay session URL or embed your saved session here. -->

[Add DevRelay session link, if available]

## Prize Categories

I am entering the overall Week 1 challenge. I have not selected a partner-specific category.
