# Roadmap

Milestone order matters here more than usual: the goal is to de-risk the scary OS-level integrations early, not to build outward from a polished shell.

## 0. Before any code

- [ ] Apply for Nemotron 3 VoiceChat early access on `build.nvidia.com` (critical path — this can take a while, start it first)
- [ ] Read the NVIDIA API Trial Service Terms of Use / Software and Model Evaluation License for production-use restrictions
- [ ] Create the Xcode project by hand (File → New → macOS App, SwiftUI, deployment target macOS 26.0). Never let an AI generate or edit the `.pbxproj`.
- [ ] Add entitlements + `Info.plist` usage strings (`NSCameraUsageDescription`, `NSMicrophoneUsageDescription`) manually

## 1. Vertical slice (prove the risky stuff works, in one path)

**Goal:** press hotkey → capture screen → send to a vision model → hear the answer spoken. Nothing else — no auth, no menu bar polish, no distraction detection yet.

- [ ] Global hotkey listener
- [ ] `ScreenCaptureKit` snapshot on trigger
- [ ] Direct call to a vision model (hardcoded local-only API key, gitignored — not the real backend yet)
- [ ] `AVSpeechSynthesizer` speaks the response

If this works end to end, the hardest parts of the app — permissions, capture, networking, TTS — are proven. If it doesn't, better to find out in days 1–3 than week 3.

## 2. Backend skeleton

- [ ] Auth (Supabase, email + Sign in with Apple)
- [ ] Stripe subscription gating
- [ ] AI proxy endpoint — move the API key server-side, client calls the backend instead of the vision provider directly

## 3. Local distraction signals

- [ ] Camera presence via Vision/MediaPipe (face present, facing screen, absent 15+s)
- [ ] Frontmost-app / window-title pre-filter before any cloud call
- [ ] Canned spoken nudge on trigger (no AI yet — "face away for 15s" → speak a fixed line)

## 4. Voice agent integration (Nemotron 3 VoiceChat)

- [ ] Backend: `VoiceAgentClient` interface (connect / send audio / receive audio / register tools / handle interruption) — build against the interface, not the SDK directly, so the provider can be swapped
- [ ] Register `look_at_screen()` / `look_at_camera()` tools with on-hold spoken messages
- [ ] Wire tool calls to the client's capture pipeline
- [ ] Route captured frames to the vision model, return text to the voice session, hear it spoken back
- [ ] Handle the ~2 min context window: re-inject session topic/summary periodically

## 5. Screen-content relevance check

- [ ] Wire the ~20–30s relevance check (cheap model) behind the local pre-filter from step 3
- [ ] Nudge rate limiting (default: 1 per 90s)
- [ ] Interrupt any playing nudge immediately when the user activates voice Q&A

## 6. Session lifecycle

- [ ] Start Session flow (topic entry or auto-detect from first screenshot)
- [ ] End Session flow (manual + idle auto-end, default 10 min)
- [ ] Server-side session log: topic, start/end time, one-paragraph AI summary — text only

## 7. Polish and ship

- [ ] Manual "pause monitoring" control in the overlay (don't defer this — see `DECISIONS.md`)
- [ ] Permission-request UX (Screen Recording, Camera, Microphone, Notifications)
- [ ] Trial → subscription conversion flow
- [ ] Notarization + DMG packaging

## Deferred to v2

- Wake-word activation
- Posture detection
- Visual session dashboard
- Multi-monitor support
- Group study / multiplayer
