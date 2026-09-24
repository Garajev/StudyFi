# Decisions

A running log of architecture decisions, why they were made, and what's still open. Update this as things change — it's more useful than a Slack thread nobody reads again.

Format: **Decision** — status — reasoning.

---

## Settled

**Native Swift app, not Electron/cross-platform.**
Status: Final for v1.
The app needs `ScreenCaptureKit`, camera + `Vision` framework access, a true always-on-top overlay, menu bar integration, and Keychain — all deep OS-level APIs that Electron can't reach cleanly.

**Backend proxy holds all API keys; client never calls AI providers directly.**
Status: Final.
This is a paid subscription product — any key shipped in the client binary is extractable. The backend checks subscription status before every AI call.

**Distraction detection is local-first.**
Status: Final.
Camera presence (Vision/MediaPipe) and frontmost-app checks run entirely on-device for free. Only ambiguous cases escalate to a cheap cloud classifier, roughly every 20–30s — not continuous.

**Voice provider: NVIDIA Nemotron 3 VoiceChat (S2S, full-duplex).**
Status: Chosen, blocked on access.
Reasoning: native tool-calling mid-conversation (with a spoken "on-hold" message while a tool runs) maps directly onto the screen-vs-camera routing problem — the model calls `look_at_screen()` or `look_at_camera()` itself instead of relying on a keyword heuristic on the transcribed question. Turn-taking latency ~450ms, competitive with commercial realtime APIs.
Caveats:
- Hosted access on `build.nvidia.com` is gated behind an **early access application**, not a self-serve API key. This is the critical-path blocker — applied for on [date].
- Governed by the NVIDIA API Trial Service Terms of Use and NVIDIA Software and Model Evaluation License. **Read these before building a paid product on top of the endpoint** — trial/evaluation licenses commonly restrict production/commercial use until off the trial tier. Not yet verified for this specific license.
- Nemotron is audio-only — no vision input. A separate vision-capable model (Claude or Gemini) still does the actual "look at this and explain it" reasoning.
- Reliable context window is short (~2 min); the backend must re-inject session context periodically rather than relying on the model to retain a multi-hour session.

**Vision reasoning (screen/book Q&A): Claude or Gemini, not yet finalized.**
Status: Open — see below.

**Wake word deferred to v1.1; v1 uses a global hotkey.**
Status: Final for v1.
Apple's on-device speech APIs aren't built for always-on wake-word spotting — that needs a dedicated engine (e.g. Picovoice Porcupine) with its own licensing and always-on battery/CPU cost. A push-to-talk hotkey ships in an afternoon, sidesteps the "is it always listening?" trust question, and can be upgraded later.

**Distribution: notarized DMG outside the Mac App Store.**
Status: Final.
Needed to bill via Stripe directly. App Store subscriptions require Apple IAP and a 15–30% cut.

**No local storage of images/video/audio, on-device or server-side — ever.**
Status: Final, non-negotiable product principle. Only derived text (topic, summary, timestamps) is persisted. See `ARCHITECTURE.md` §4 for the full table.

---

## Open

**Vision model for screen/book Q&A: Claude vs. Gemini.**
Needs a real test, not a leaderboard: photograph actual textbook pages at bad angles/lighting and compare. Cost also matters — this fires on every voice Q&A turn.

**Privacy framing: privacy-first vs. best-in-class cloud experience.**
The original spec's "nothing ever leaves the device" framing and a cloud full-duplex voice model are in tension. Whatever's built, the privacy table in `ARCHITECTURE.md` must describe reality, not the aspiration — especially for a studying-focused, often younger user base that will notice the gap.

**Fallback if Nemotron early access is denied or too rate-limited for production.**
Recommendation: build the backend's voice-agent integration behind a small internal interface (connect / send audio / receive audio / register tools / handle interruption) so Gemini Live or a self-hosted Nemotron NIM container can be swapped in without touching client code. Not yet built.

**Wake word phrase, and whether it needs to be user-customizable or can be hardcoded.** (Deferred to v1.1 alongside the wake-word feature itself.)

**Multi-display support.** v1 assumes primary display only. Revisit if the target user commonly studies on two screens.

**Manual "pause monitoring" control.** Recommended to build into the overlay from day one (not as later polish) — an app that flags someone as "distracted" for stepping away is a fast way to get uninstalled, and retrofitting a pause state through detection code that wasn't built with it is genuinely painful.

**Free trial length and AI cost per session.** Spec placeholder is 7 days; needs to be tuned once actual per-session AI spend (voice + vision + relevance checks) is measured.

**Should the AI stay silent on sensitive-but-unrelated screen content** rather than flagging it? Not yet decided.
