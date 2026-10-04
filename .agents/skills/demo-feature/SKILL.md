---
name: demo-feature
description: Act as a product manager to produce a demo script, narrated screen recording, and natural voice audio that demonstrates the value and correctness of a newly delivered feature. Use when the user asks to "demo", "record a demo", "show off the feature", or "create a product demo".
---

# Demo Feature — PM-Narrated Screen Recording with Voice

Produce a polished product demo for a newly delivered feature: write a PM-quality demo script, drive a browser-based screen recording, and synthesize natural-sounding narration audio — all in one automated workflow.

---

## When to Use

- User says: *"demo the feature"*, *"record a demo"*, *"show this off"*, *"create a product demo"*, *"make a demo video"*
- After a feature is implemented and the user wants to validate or share it

---

## Phase 1 — Write the Demo Script (Product Manager Voice)

Adopt the perspective of an enthusiastic, customer-focused Product Manager. The script has three acts:

### Act 1 — Hook (30 s)
- Open with the **pain point** the feature solves
- State the business or user value in one sentence
- Example opener: *"Before today, every time a user wanted to X, they had to Y — wasting minutes and breaking their flow."*

### Act 2 — Live Demo (2–3 min)
Break into clearly named **beats**, each with:
| Field | Content |
|-------|---------|
| **Action** | What to click/type/navigate |
| **Narration** | What to say during this action |
| **Value callout** | The benefit to highlight |

Cover the **happy path** first, then at least one **edge case or error recovery** that shows robustness.

### Act 3 — Close (30 s)
- Summarise the 2-3 key takeaways
- State the call-to-action (try it, ship it, share it)

---

## Phase 2 — Record the Screen

Use browser automation tools to execute each beat from the script:

```
1. openBrowserPage  → navigate to the running app / feature URL
2. For each beat in Act 2:
   a. Execute the described action (click, type, navigate, hover)
   b. screenshotPage → capture a frame for the beat
   c. Pause briefly to simulate human pacing
3. Assemble screenshots into a slide-by-slide storyboard (ordered list of images + captions)
```

**Tips:**
- Resize viewport to 1280×720 for a clean 16:9 aspect ratio before recording
- Use `readPage` to confirm the expected UI state before each action
- If a step fails, note it in the script and adapt — do not silently skip

---

## Phase 3 — Synthesize Natural Voice Narration

For each beat, produce an audio clip. Use the options below in priority order — Option A produces the most natural result and should be the default.

### Option A — edge_tts / Microsoft Neural Voice ⭐ (preferred)

`edge_tts` streams from Microsoft's neural TTS service (same engine as Azure Cognitive Services) with no API key required. It produces significantly more natural audio than the Web Speech API or OS TTS.

```bash
pip install edge-tts
```

```python
import asyncio
import edge_tts
from pathlib import Path

async def save_beat(text: str, output_path: Path, voice: str = "en-US-JennyNeural") -> None:
    """Save one narration beat to an MP3 file.

    Recommended voices:
      en-US-JennyNeural   — warm, conversational female (default)
      en-US-GuyNeural     — clear, professional male
      en-US-AriaNeural    — expressive, enthusiastic female
    """
    communicate = edge_tts.Communicate(text, voice, rate="-5%")
    await communicate.save(str(output_path))

# Usage (one beat):
asyncio.run(save_beat("Your narration text here.", Path("beat-01.mp3")))

# Usage (all beats in a loop):
beats = [
    ("beat-01", "Before Oscillation, every time you gave an AI agent..."),
    ("beat-02", "With Oscillation, you start by creating an operator profile..."),
]
async def save_all():
    for name, text in beats:
        await save_beat(text, Path(f"{name}.mp3"))
asyncio.run(save_all())
```

**Why it is better:**
- Neural voice quality — not robotic, not monotone
- No API key or Azure subscription required
- Saves directly to MP3; works headless
- Consistent cadence across all beats

---

### Option B — Web Speech API (zero-dependency, browser only)
```javascript
// Run via runPlaywrightCode inside an open browser page
const synth = window.speechSynthesis;
const utt = new SpeechSynthesisUtterance("Your narration text here.");
utt.rate = 0.95;   // slightly slower than default — sounds more natural
utt.pitch = 1.05;
utt.voice = synth.getVoices().find(v => v.lang === 'en-US' && v.localService);
synth.speak(utt);
```

Use Option B only when a browser page is already open and `edge_tts` is unavailable.

---

### Option C — OS TTS (PowerShell, Windows, last resort)
```powershell
Add-Type -AssemblyName System.Speech
$synth = New-Object System.Speech.Synthesis.SpeechSynthesizer
$synth.SelectVoiceByHints('Female', 'Adult')
$synth.Rate = 1          # -10 (slowest) to 10 (fastest); 0–2 is natural
$synth.SetOutputToWaveFile("beat-01.wav")
$synth.Speak("Your narration text here.")
$synth.SetOutputToDefaultAudioDevice()
```

Use Option C only as a last resort — it produces robotic audio and is noticeably lower quality than Options A or B.

---

**Selection guide:**

| Situation | Use |
|-----------|-----|
| Default — best audio quality | Option A (`edge_tts`) |
| Browser already open, no pip access | Option B (Web Speech API) |
| Headless, no internet, no pip | Option C (OS TTS) |

---

## Phase 4 — Deliver the Artefacts

Present the following to the user:

| Artefact | Format |
|----------|--------|
| Demo script | Markdown with Act/Beat structure |
| Storyboard | Ordered list of screenshots with captions matching script beats |
| Voice narration | MP3 files via `edge_tts` (one per beat), or WAV from OS TTS fallback |
| Summary slide | Single screenshot of the most impressive moment + one-sentence value statement |

Save all artefacts to the session `files/` folder so they persist across checkpoints:
```
files/demo/
  script.md
  storyboard/
    beat-01.png
    beat-02.png
    ...
  audio/
    beat-01.mp3
    beat-02.mp3
    ...
```

---

## Quality Checklist

Before handing off, verify:
- [ ] Script covers happy path **and** at least one edge/error case
- [ ] Every screenshot matches its corresponding script beat
- [ ] Narration audio plays without clipping or awkward pauses
- [ ] The demo can be understood with audio only (accessibility)
- [ ] The demo can be understood with screenshots only (silent mode)
- [ ] Total runtime fits within the requested length (default: ~3–4 min)

---

## Example Invocation

> "Demo the new CSV export feature. The app is running at http://localhost:3000."

**Expected output:**
1. A `script.md` with Hook → 5 beats → Close
2. Screenshots: landing page → trigger export → progress indicator → downloaded file confirmation → edge case (empty dataset handled gracefully)
3. MP3 narration files for each beat (via `edge_tts`)
4. A one-sentence summary: *"Users can now export any filtered dataset to CSV in two clicks, with clear feedback at every step."*
