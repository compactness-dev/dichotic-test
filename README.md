# Dichotic Listening Test

A self-contained single-file webapp implementing a standard dichotic listening paradigm for assessing auditory laterality (hemisphere dominance for language processing).

## Usage

Open `index.html` in a modern browser with stereo headphones. Alternatively:

```bash
cd dichotic-test && python3 -m http.server 8080
# open http://localhost:8080
```

## How It Works

Two different auditory stimuli are played simultaneously — one to each ear — using the Web Audio API's `StereoPannerNode` (hard-panned left at -1, right at +1). The participant identifies what they heard in each ear. Accuracy per ear reveals which hemisphere dominates for phonological processing:

- **Right ear advantage (REA)** → left-hemisphere language dominance (typical, ~85% of population)
- **Left ear advantage (LEA)** → right-hemisphere language dominance (atypical, ~5% of population)

This works because contralateral auditory pathways (left ear → right hemisphere, right ear → left hemisphere) dominate under dichotic competition.

## Stimulus Types

### Digits (Standard DDT)
Digits 1–9. Easy; useful for screening. Prone to ceiling effects in healthy adults.

### CV Syllables
Six stop-consonant + vowel syllables: /ba/, /da/, /ga/, /pa/, /ta/, /ka/. Based on the classic Kimura paradigm. Harder to discriminate under competition, producing clearer laterality effects.

## Test Modes

| Mode | Description | Use case |
|---|---|---|
| **Free Recall** | Report both ears | Standard laterality measure |
| **Directed Left** | Attend left ear only | Tests attentional control / perceptual dominance |
| **Directed Right** | Attend right ear only | Tests attentional control / perceptual dominance |

Directed modes are critical for distinguishing genuine perceptual asymmetry from attentional bias.

## Trial Structure

1. Fixation cross (800ms)
2. Simultaneous dichotic presentation
3. Response delay (700ms)
4. Response grid (order randomized per trial to counterbalance serial report bias)
5. Submit → next trial

- **3 practice trials** with corrective feedback
- **20 / 30 / 60 test trials** (configurable) without feedback

## Results

- **Per-ear accuracy** (% correct)
- **Laterality Index**: `(Right - Left) / (Right + Left) × 100`. Positive = REA, negative = LEA.
- **Intrusion Analysis** (directed modes): counts errors that exactly match the unattended ear's stimulus, compared to chance rate
- **Response Order Breakdown** (free recall): accuracy split by which grid appeared first
- **Trial-by-trial detail**: both stimuli shown, correct/wrong marked, intrusions labeled

## Diagnostic Panel

Accessible from the welcome screen or after any test run. Synthesizes all session data into a laterality conclusion.

### Edinburgh Handedness Inventory (abbreviated)
10-item handedness questionnaire (writing, drawing, throwing, scissors, etc.). Produces a score from -100 (strongly left-handed) to +100 (strongly right-handed). Used for concordance analysis with ear advantage.

### Session History
Accumulates all test runs within a browser session. Tracks stimulus type, mode, trial count, per-ear accuracy, laterality index, and intrusion rate across runs.

### Laterality Conclusion
Determines language hemisphere dominance (left, right, or bilateral) with a confidence score (0–95%) based on:
- Number and type of test runs (CV syllables weighted higher than digits)
- Strength of laterality index
- Directed attention data availability
- Intrusion rate analysis
- Handedness concordance
- Total trial count

### Population Visualization
Visual bar chart showing the population distribution of handedness × language laterality combinations, with "YOU" marker highlighting the user's group:
- Right-handed + LH language: ~87%
- Left-handed + LH language: ~7%
- Right-handed + RH language: ~4%
- Left-handed + RH language: ~1.5%
- Bilateral: ~0.5%

### Recommendations Engine
Suggests next steps based on what data has been collected (e.g., "run CV syllables," "try directed attention," "complete handedness questionnaire").

### Export
Copies a plain-text diagnostic report to clipboard.

## Audio

Generated via Google Text-to-Speech (`gTTS`), processed with ffmpeg:
- Silence-trimmed (start and end)
- Volume-normalized to -16 LUFS
- Mono, 22050 Hz WAV
- ~0.4s per CV syllable, ~0.5s per digit
- Base64-encoded and embedded in the HTML (no external dependencies)

## Technical Details

- **Single file**: `index.html` (~440KB, mostly embedded audio)
- **No dependencies**: vanilla HTML/CSS/JS, runs offline
- **Browser requirements**: Web Audio API, `StereoPannerNode` (all modern browsers)
- **Audio decoding**: base64 → ArrayBuffer → `AudioContext.decodeAudioData()`

## Limitations

- Synthetic TTS audio, not standardized clinical recordings
- Not validated against clinical DDT norms
- No test-retest reliability data
- Headphone isolation depends on user's hardware
- For research/educational use; not a clinical diagnostic tool

## References

- Kimura, D. (1961). Cerebral dominance and the perception of verbal stimuli. *Canadian Journal of Psychology*, 15(3), 166–171.
- Hugdahl, K. (2011). Fifty years of dichotic listening research. *Brain and Cognition*, 76(2), 211–232.
- Hugdahl, K., & Westerhausen, R. (2016). Speech processing asymmetry revealed by dichotic listening and functional brain imaging. *Neuropsychologia*, 93, 466–481.
