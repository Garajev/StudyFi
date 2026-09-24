# CLAUDE.md

Ground rules for AI-assisted development in this repo. Read `ARCHITECTURE.md` and `DECISIONS.md` before making structural changes — this file is constraints, those are context.

## Hard constraints

- **Deployment target: macOS 26.0.** Don't write code that falls back to older Speech/AVFoundation APIs "for compatibility" unless explicitly asked.
- **Swift 6, strict concurrency.** No `@unchecked Sendable` as a shortcut — fix the actual isolation issue.
- **Never create or edit `.pbxproj`, `.xcworkspace`, or other Xcode project files.** These are edited by hand in Xcode only. If a change requires touching one (new target, new file group, new capability), stop and say so instead of attempting it.
- **No third-party Swift packages without asking first.** Foundation, SwiftUI, AppKit, ScreenCaptureKit, AVFoundation, Vision, and Speech cover most of this app. If something genuinely needs a dependency, name it and why before adding it.
- **All networking goes through a single client abstraction**, not scattered `URLSession` calls. Currently: `APIClient` (client) and `VoiceAgentClient` (backend, see `ARCHITECTURE.md` §3.3). New network calls extend these, they don't bypass them.
- **One type per file.** File name matches the type name.
- **No API keys, tokens, or secrets in client code, ever** — not even "temporarily for testing." Use a local `.env`-style gitignored file for the vertical-slice milestone only (see `ROADMAP.md` §1), and delete that path once the backend proxy exists.

## Data handling

- Never write image, video, or audio data to disk, `UserDefaults`, or any persistence layer — on-device or server-side. Only derived text (topic labels, summaries, timestamps) may be persisted. This is a product principle, not a preference — see `ARCHITECTURE.md` §4.
- Camera frames used for presence/gaze detection must never be sent over the network. Only explicit book/camera Q&A frames leave the device, and only for that one request.

## Workflow

- Compile after every meaningful change:
  ```bash
  xcodebuild -scheme StudyCompanion -destination 'platform=macOS' build 2>&1 | tail -40
  ```
  Read the actual errors before proposing a fix — don't guess.
- When testing permission flows, macOS caches grants per bundle ID. Reset with:
  ```bash
  tccutil reset ScreenCapture com.yourname.studycompanion
  tccutil reset Camera com.yourname.studycompanion
  tccutil reset Microphone com.yourname.studycompanion
  ```
- If a task touches the voice pipeline, re-read `ARCHITECTURE.md` §3.3 first — the tool-calling contract (`look_at_screen` / `look_at_camera`) is the intended integration point, not a keyword heuristic on transcribed text.
- If something in `DECISIONS.md` is marked **Open** and your task depends on it, flag that instead of silently picking an answer.

## Style

- Prefer clarity over cleverness — this is a small team (possibly one person) maintaining this long-term.
- Match existing patterns in a file/module before introducing a new one; don't refactor unrelated code while implementing a feature.
