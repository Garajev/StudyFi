 # StudyFi - An AI Agent for Studying 

> A macOS AI companion that watches your screen and camera while you study, keeps you on task, and answers questions about what you're looking at — spoken, hands-free.

**Status:** Pre-build. Architecture decided, voice provider pending early-access approval. See [`DECISIONS.md`](./DECISIONS.md) for what's settled and what's still open.

Placeholder name — replace throughout once a real name is picked.

---

## What it does

- Watches your screen and camera during a study/work session
- Notices when you drift off-task and gives a spoken nudge — no popups, voice only
- Answers spoken questions about your screen or a physical book, hands-free
- Remembers what you studied, session to session (text only, no dashboard yet)

## Tech stack

| Layer | Choice | Why |
|---|---|---|
| Client | Swift 6, SwiftUI + AppKit | Needs ScreenCaptureKit, Vision, Keychain, true always-on-top overlay — not reachable cleanly from Electron |
| Overlay window | `NSPanel` (non-activating, floating level) | Pure SwiftUI can't do proper always-on-top |
| Screen capture | `ScreenCaptureKit` | On-demand snapshot, primary display only (v1) |
| Camera + presence | `AVFoundation` + `Vision` / MediaPipe Face Landmarker | On-device only, never leaves the device unless a book Q&A is asked |
| Voice agent | NVIDIA Nemotron 3 VoiceChat (S2S, full-duplex) | Native tool-calling mid-conversation — used to trigger screen/camera capture. Currently gated behind early access; see `DECISIONS.md`. |
| Vision reasoning (screen/book Q&A) | Claude / Gemini (TBD) | Nemotron is audio-only; a separate vision-capable model reasons about captured frames |
| Backend | TypeScript, Node (Hono/Fastify) | Proxies all AI calls, holds API keys, never trusts the client |
| Auth | Supabase (email + Sign in with Apple) | Handles Apple sign-in without building it from scratch |
| Billing | Stripe | Subscription gate checked server-side before any AI call |
| Distribution | Notarized DMG, outside the Mac App Store | Needed to use Stripe directly instead of Apple IAP |

## Repo structure

```
studycompanion/
├── README.md           ← you are here
├── ARCHITECTURE.md      ← system design, data flow, pipelines
├── DECISIONS.md         ← decision log — what's settled, what's open, why
├── ROADMAP.md           ← build order / milestones
├── CLAUDE.md            ← ground rules for AI pair-programming this repo
├── .gitignore
├── mac/                  ← Xcode project (create manually, don't let an AI generate the .pbxproj)
└── server/               ← backend (auth, Stripe, AI proxy)
```

## Getting started

Nothing runnable yet — this repo currently holds architecture and planning docs. See [`ROADMAP.md`](./ROADMAP.md) for the build order, starting with a single vertical slice: hotkey → screen capture → voice answer.

## Docs

- [`ARCHITECTURE.md`](./ARCHITECTURE.md) — full system design, capture/voice/distraction pipelines, data & privacy model
- [`DECISIONS.md`](./DECISIONS.md) — key decisions and open questions, with reasoning
- [`ROADMAP.md`](./ROADMAP.md) — MVP build order
- [`CLAUDE.md`](./CLAUDE.md) — constraints for AI-assisted ("vibecoded") development in this repo

