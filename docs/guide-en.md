# Socratic Verse — Usage Guide

> The AI that questions you, not answers you.

---

## What is this?

Socratic Verse is an **open learning system**. Drop in study material (markdown / pdf / txt). The AI doesn't lecture you — it leads you through a chain of questions until you discover everything through your own reasoning.

2,400 years of cognitive science proves: **one retrieval beats three re-readings.** (Karpicke & Roediger, *Science*, 2008)

---

## What you need

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview) (recommended) or any AI assistant that supports custom prompts

---

## Quick start (60 seconds)

```bash
# 1. Clone
git clone git@github.com:guocongcongcong/socratic-verse.git
cd socratic-verse

# 2. Drop your study material in materials/
cp ~/your-textbook.md materials/

# 3. Launch Claude Code
claude
```

Then say in Claude Code:

```
Please read teacher/system.md first, then use V7 Socratic Seminar mode to teach me materials/<your-file>.
```

---

## Available methods (V1–V8)

| Version | Method | One-liner | Best when |
|:---:|------|------|------|
| V1 | Cornell Notes | AI generates Cornell-formatted notes; you cover right side and recall | Building retrieval habit |
| V2 | Spaced Repetition (Anki-style) | AI pushes reasoning-based cards daily | Long-term memory, micro-moments |
| V3 | Random Dice | Roll a die, answer that question — zero decision cost | Don't know what to study today |
| V4 | Closed-Book Recall | AI guides you to recall everything you just read | Just finished a chapter |
| V5 | Feynman Technique | AI plays a clueless high schooler, forces plain-English explanations | Testing understanding depth |
| V6 | Interleaved Practice | AI mixes questions from different subjects | Finding concept boundaries |
| V7 | Socratic Seminar | AI questions relentlessly, never gives answers | Deep learning (default) |
| V8 | Peer Teaching | AI plays a peer; you teach each other and challenge reasoning | Social-style solo learning |

Tell AI the version number:

```
Use V4 Closed-Book Recall mode. I just finished reading materials/machine-learning.md chapter 3.
```

---

## Three AI teachers

| Teacher | Style | Best for |
|------|------|------|
| **March 7** | Energetic, bubbly — "Yes yes! That's it! But what if..." | People who need encouragement |
| **Ganyu** | Gentle math prodigy, step-by-step guidance | People who need patience |
| **Keqing** | Strict, efficient, blunt | People who need pushing |

Specify a teacher:

```
Use V7 Socratic mode with teacher Keqing.
```

---

## Advanced usage

### Custom teachers

Create a new `.md` file in `teacher/` following the existing format, then reference it in your prompt.

### Contribute materials

Drop markdown files in `materials/` and submit a Pull Request. Any format works; chapter headings recommended.

### One-Click Citation Mode

Add this to the end of your prompt:

```
After each round of questioning, automatically list the original passages you referenced and explain your reasoning for citing them.
```

The AI will additionally output:
- **Cited Passage**: the original text your reasoning drew from
- **Citation Reason**: why this passage supports your conclusion

**Ready-to-use prompts with citations built in:**

**Quick Start + Citations:**
```
Please read teacher/system.md first.
Use V7 Socratic Seminar mode with teacher March 7.
After each round of questioning, list the original passages you drew from and explain why each passage applies.
Teach me materials/<your-file>.
```

**Feynman + Citations:**
```
Please read teacher/system.md first.
Use V5 Feynman Technique mode. Play a high school student.
After each explanation I give, list: (1) the original text passages my explanation draws from, (2) which parts I simplified correctly, and (3) which parts I over-simplified or missed.
Teach me materials/<your-file>.
```

**Closed-Book Recall + Citations:**
```
Please read teacher/system.md first.
Use V4 Closed-Book Recall mode.
After I finish recalling, show me what I missed AND cite the exact passages from the original text that I should have remembered, explaining why each passage is important.
Teach me materials/<your-file>.
```

---

## FAQ

**Q: I don't have Claude Code. What now?**
A: Copy `teacher/system.md` and the prompts from `prompts/` into ChatGPT / Hermes Agent, and manually paste your study material. Similar experience.

**Q: What formats are supported?**
A: Plain text (.md / .txt) works best. PDFs need OCR first.

**Q: Can I use local models?**
A: Yes. Run Ollama with a reasoning-capable model. The prompt system is model-agnostic.

**Q: When is voice mode coming?**
A: Phase 3, expected Q4 2026. Walk-and-learn, hands-free.

---

MIT License · [GitHub](https://github.com/guocongcongcong/socratic-verse)
