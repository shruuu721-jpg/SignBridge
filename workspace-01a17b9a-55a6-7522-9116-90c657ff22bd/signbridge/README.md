# SignBridge — frontend

A single-file, offline-capable frontend for the AI SignBridge two-way sign-language
communicator described in section 4 of the project document.

**Open `index.html` in any modern browser.** No build step, no dependencies, no
network calls — everything (layout, animation engine, speech, state) lives in one
68 KB HTML file.

---

## What the interface does

| Side | Input | Output |
|---|---|---|
| **Sign → Text & Speech** | Signing (simulated camera feed with a 21-point hand-landmark overlay) | Live typewriter caption, confidence score, spoken audio via the Web Speech API, and a line in the conversation transcript |
| **Voice → Text & Sign** | Microphone (Web Speech recognition), typed text, or a quick phrase | Text transcript plus an animated signing avatar with a gloss caption explaining each sign as it is performed |

Plus a shared **Conversation** log with “Signed / Spoken” attribution, timestamps,
and a copy-to-clipboard transcript — so the two directions read as one dialogue, not
two separate tools.

## Feature checklist

- **Two-way, one screen** — “Two-way”, “Sign → Voice” and “Voice → Sign” modes.
- **Animated signer** — SVG character driven by a keyframe player; arms are solved
  with 2-bone inverse kinematics so limbs never stretch, and each sign is a real
  short motion phrase (wave, chin touch, chest circle, rising sun, fingerspelling).
- **16 signs** + fingerspelling fallback for unknown words:
  Hello, Thank you, Please, Yes, No, Help, Sorry, Name, Water, How are you?,
  Nice to meet you, Wait, Slow, Good morning, Goodbye, I understand, Friend.
- **Text → sign pipeline** — phrase matching → word glossing → fingerspelling,
  with per-sign gloss captions.
- **Accessibility controls** — text size S/M/L/XL, signing speed 0.7×/1×/1.4×,
  spoken-output toggle, gloss toggle, reduce-motion, high contrast, focus-visible
  rings, `aria-live` captions, `aria-pressed`/`aria-selected` state, and keyboard
  shortcuts (`Space` mic, `1`–`6` demo signs, `Esc` close).
- **Responsive** — two-column desktop layout collapses to a single column with a
  compact segmented mode switch on phones.
- **Graceful degradation** — if speech synthesis or recognition is unavailable
  (sandboxed iframe, no mic permission), the app falls back to a simulated speech
  stream and tells you so instead of failing silently.

## Where the mock ends (integration points)

Everything user-facing is real; two data sources are simulated so the UI can be
demoed without a trained model:

1. **Sign recognition.** `showSign(key)` is the seam. In production a MediaPipe
   Holistic / hand-landmark stream would classify the sequence and call
   `showSign(predictedClass, { conf: probability })`. The landmark overlay in the
   left panel is already drawn from the same 21-point template, so the visual
   contract with a real model is in place.
2. **Speech recognition.** `startListening()` uses `SpeechRecognition` when
   available and `recognition.lang` is wired to the “Speech input” selector
   (English India/US/UK, Hindi beta). Swap in a Whisper/streaming ASR endpoint by
   replacing the `recognition.onresult` handler.

To extend the vocabulary, add an entry to `SIGNS` — a label, a gloss note, and a
handful of `restPose({ rw: [...], lw: [...], rhand: 'flat' | 'fist' | 'point' | 'cup' })`
keyframes. The IK solver and player handle the rest.

## Files

```
signbridge/
├── index.html          # complete frontend (markup + styles + engine + app logic)
├── README.md
└── screenshots/
    ├── gallery.html    # all shots on one scrollable page
    ├── 01-two-way.png          # default two-way workspace
    ├── 02-sign-to-text.png     # sign recognised -> caption + confidence + speech
    ├── 03-voice-to-sign.png    # spoken/typed English -> animated signing
    ├── 04-conversation.png     # shared Signed/Spoken transcript
    ├── 05-accessibility.png    # text size, speed, motion, contrast controls
    ├── 06-voice-mode.png       # single-direction mode
    ├── 07-mobile.png           # 390 px layout
    └── 08-mobile-voice.png     # mobile signing panel
```

## Verified

Rendered and exercised in headless Chromium at 1440 px and 390 px widths:
19/19 interaction checks pass (sign recognition, logging, vocabulary, mic flow,
replay, speed, accessibility settings, mode switching, demo script) with zero
console errors.
