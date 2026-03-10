# Development Log: Dichotic Listening Test

Built in a single session with Claude Code (Opus 4.6). The entire webapp is one self-contained HTML file — no frameworks, no build tools, no server.

## How it started

A one-sentence prompt: *"Can you build a small dichotic listening test webapp? Fetch audio files from a reliable source, and follow standard test practices."* Here's how the project evolved from there.

## Phase 1: Getting audio into the browser

The first challenge was getting speech audio into a self-contained HTML file. The standard Dichotic Digits Test uses spoken digits (1–9), and `espeak` was available on the system, so I generated WAV files, base64-encoded them, and embedded them directly in the HTML. No external files, no server — just open it in a browser.

**Technical aside:** The Web Audio API's `StereoPannerNode` routes each stimulus to a specific ear (pan = -1 for left, +1 for right). Both sounds play simultaneously. That's the entire dichotic trick.

## Phase 2: Digits were too easy

The digit test produced near-perfect scores — a classic ceiling effect. Not hard enough to reveal laterality differences. The user asked for something harder, so I added CV (consonant-vowel) syllables: /ba/, /da/, /ga/, /pa/, /ta/, /ka/ — the standard set from Shankweiler & Studdert-Kennedy (1967), used in the Bergen Dichotic Listening Test. These are much harder to discriminate under dichotic competition.

## Phase 3: The espeak problem

CV syllables with `espeak` were a disaster — near-chance accuracy, and the user kept hearing /ma/ and /na/, which weren't even in the stimulus set. The issue was `espeak`'s formant synthesizer: its stop consonant bursts were too weak, so the brief closures distinguishing /b/ from /m/ and /d/ from /n/ were getting lost in dichotic competition.

I switched to Google TTS (`gTTS`), trimmed silence with `ffmpeg`, normalized volume to -16 LUFS, and re-embedded the audio. The consonants became crisp, and accuracy patterns became meaningful.

## Phase 4: Laterality emerges

With better audio, the test started producing clear laterality effects. This is where dichotic listening gets interesting: the ear advantage reveals which hemisphere dominates for phonological processing, because contralateral auditory pathways (left ear → right hemisphere, right ear → left hemisphere) are stronger under dichotic competition.

## Phase 5: Directed attention

A key question in dichotic listening research: is an ear advantage genuine perceptual asymmetry, or just attentional bias? I implemented **Directed Attention** modes (attend left only, attend right only) to test this. If a user can freely recover accuracy on the disadvantaged ear just by trying harder, it's likely attention. If intrusions from the unattended ear persist — if the dominant ear's stimulus keeps overriding conscious effort — that's evidence for genuine perceptual dominance.

The **intrusion analysis** tracks exactly this: what fraction of errors on the attended ear match what was actually playing in the unattended ear, compared to the chance rate (17% for 6 syllables).

## Phase 6: Grid order and practice effects

An unexpected UX problem emerged: when the response grid order was randomized (left grid vs right grid shown first), users could lose track of which ear heard what, producing noisy data. I added a **Fixed/Randomized toggle** so users can choose their preferred response order.

Practice effects also became apparent across runs — users can meta-learn the task, consciously compensating for a known asymmetry. This is documented in the literature; the first naive run tends to be the most diagnostic.

## Phase 7: The diagnostic panel

The user asked for a diagnostic summary with confidence scoring. I built a panel that synthesizes all session data:

- **Edinburgh Handedness Inventory** — 10-item questionnaire for handedness scoring (-100 to +100)
- **Session history** — tracks every run across the session
- **Laterality conclusion** with confidence score (0–95%) based on weighted evidence factors
- **Intrusion analysis** — flags when errors match the unattended ear
- **Population visualization** — shows where the user sits among the 5 handedness × laterality groups
- **Recommendations engine** — suggests what to run next based on gaps in collected data

## Phase 8: Polish

I redesigned the UI with a light theme, refined typography (DM Sans + JetBrains Mono), and added sample results so new users can see what the output looks like before running a test.

## Design decisions

1. **Single file, no dependencies.** The entire app is one HTML file (~440KB, mostly embedded audio). No React, no npm, no build step. Open it in a browser — that's the deployment.
2. **Audio is the hard part.** Browser `SpeechSynthesis` can't be routed through the Web Audio API for stereo panning. The solution: generate audio externally (espeak/gTTS), base64-encode it, embed it, decode with `AudioContext.decodeAudioData()`.
3. **Domain knowledge drives features.** The 6 CV syllables, the laterality index formula (`(R - L) / (R + L) × 100`), the intrusion analysis, the Edinburgh Handedness Inventory — these are established research tools. Each feature was added when real testing revealed the need for it: ceiling effects → harder stimuli, ambiguous results → directed attention, raw scores → diagnostic synthesis.
4. **Iterative development.** Every feature came from the user running the test and noticing something: too easy → add syllables. Bad audio → switch TTS. Confusing grid order → add toggle. The conversation was the development process.

## References

- Kimura, D. (1961). Cerebral dominance and the perception of verbal stimuli. *Canadian Journal of Psychology*, 15(3), 166–171.
- Shankweiler, D. & Studdert-Kennedy, M. (1967). Identification of consonants and vowels presented to left and right ears. *Quarterly Journal of Experimental Psychology*, 19(1), 59–63.
- Hugdahl, K. (2003). Dichotic listening in the study of auditory laterality. In K. Hugdahl & R.J. Davidson (Eds.), *The Asymmetrical Brain*. MIT Press.
- Hugdahl, K. (2011). Fifty years of dichotic listening research. *Brain and Cognition*, 76(2), 211–232.
- Oldfield, R.C. (1971). The assessment and analysis of handedness: The Edinburgh inventory. *Neuropsychologia*, 9(1), 97–113.

---

*Developed with Claude Code (Opus 4.6). Not academically validated. For educational and exploratory purposes only.*
