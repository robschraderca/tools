# WCE AI Sales Training Tool — Architecture Plan

## The Vision
Turn your existing sales script helper into an AI-powered training platform where reps can **practice calls against realistic AI prospects** that push back with real objections, and get **feedback on their tone, pacing, and script adherence**.

---

## Phased Approach (Start Slow, Scale Up)

### Phase 1 — Text-Based AI Role-Play (Week 1-2, ~$5-15/mo)
**What it does:** Rep types their pitch, AI responds as a prospect persona with realistic objections pulled from your existing rebuttal library.

**Why start here:**
- Zero audio complexity — just API calls
- Lets you nail the personas and scenario logic before adding voice
- Reps can practice on their phones during downtime
- Costs almost nothing at 4 users

**Tech stack:**
- **AI Model:** OpenAI GPT-4o-mini — $0.15/1M input tokens, $0.60/1M output — a full role-play session costs ~$0.01-0.03
- **Frontend:** Add a new "Practice Mode" tab to your existing HTML script helper
- **Hosting:** Static HTML + direct API calls (or a simple Node.js backend for API key security)

**Estimated cost for 4 reps:** $5-15/month depending on usage

---

### Phase 2 — Voice Role-Play (Week 3-6, ~$30-80/mo)
**What it does:** Rep speaks out loud, AI listens and responds with a voice — feels like a real phone call. AI evaluates tone confidence, hesitation, filler words.

**Two options here:**

#### Option A: DIY Pipeline (Cheapest, ~$30-50/mo for 4 reps)
Stitch together best-in-class pieces yourself:

| Component | Service | Cost |
|-----------|---------|------|
| Speech-to-Text | Groq Whisper Large v3 Turbo | $0.04/hr of audio |
| AI Brain | OpenAI GPT-4o-mini | ~$0.02/session |
| Text-to-Speech | ElevenLabs Flash | $0.06/1K chars (~$0.30/session) |
| Tone Analysis | OpenAI (analyze transcript) | ~$0.01/session |

**Per practice session (10-15 min):** ~$0.35-0.50
**4 reps × 3 sessions/week × 4 weeks:** ~$20-30/mo in API costs
**Add $10/mo for a small server (Railway, Fly.io, or Render)**

**Latency:** 1.5-3 seconds between rep finishing a sentence and AI responding (transcribe → think → speak). Not instant, but workable for practice.

#### Option B: OpenAI Realtime API (Better experience, ~$50-80/mo)
One API handles everything — listen, think, respond — with sub-second latency.

| Component | Service | Cost |
|-----------|---------|------|
| Everything | OpenAI Realtime (gpt-realtime) | ~$0.04/min with caching |

**Per practice session (10-15 min):** ~$0.40-0.60
**4 reps × 3 sessions/week × 4 weeks:** ~$25-40/mo in API costs
**Add $10/mo for server**

**Latency:** Sub-second — feels like a real conversation. This is the better experience but slightly harder to add custom tone analysis.

---

### Phase 3 — Scorecard & Analytics (Week 7-10, same cost)
**What it does:** After each practice session, AI generates a scorecard:
- Script adherence (did they hit all phases?)
- Tone confidence score (hesitation, filler words, energy)
- Objection handling rating
- Specific coaching notes ("You rushed through the pitch — slow down before the commitment question")

**Tech:** This is just post-processing the transcript from Phase 2 through GPT-4o-mini with your script as the rubric. Adds ~$0.02-0.05 per session.

---

### Phase 4 — Scale to 8-10 Reps + Live Call Assist (Month 3+)
**What it does:** Real-time whisper coaching during actual calls — AI listens to the live call and surfaces the right rebuttal card when a prospect objects.

**This is the expensive one** and should only be built after Phases 1-3 are solid. Requires Deepgram streaming STT ($0.0077/min) running continuously during calls.

---

## Cost Summary

| Phase | Timeline | Monthly Cost (4 reps) | Monthly Cost (10 reps) |
|-------|----------|----------------------|------------------------|
| 1. Text Role-Play | Week 1-2 | $5-15 | $15-40 |
| 2A. Voice (DIY) | Week 3-6 | $30-50 | $70-120 |
| 2B. Voice (Realtime) | Week 3-6 | $50-80 | $100-180 |
| 3. Scorecards | Week 7-10 | +$5-10 | +$10-25 |
| 4. Live Coaching | Month 3+ | +$40-80 | +$80-180 |

---

## Recommended Path

**Start with Phase 1 this week.** Here's why:

1. It costs almost nothing and you can have it running in days
2. Your existing rebuttal library becomes the AI's playbook — we feed your exact scripts into the system prompt
3. You'll immediately learn what persona types your reps struggle with most
4. Everything you build in Phase 1 (personas, scoring logic, the UI) carries directly into Phase 2

**For Phase 2, I recommend Option B (OpenAI Realtime)** if budget allows. The sub-second response time makes practice feel real, and OpenAI handles all the audio plumbing. The DIY route saves $20-30/mo but adds engineering complexity and latency.

---

## What I Can Build For You Right Now

If you want to move forward, the first deliverable would be:

**An upgraded version of your sales-script-helper.html** with a new "AI Practice" tab where:
- Rep picks a scenario (e.g., "Prospect has $35K in credit card debt, skeptical, been burned before")
- AI plays the prospect using your actual script phases as the framework
- AI throws objections from your rebuttal library at realistic moments
- After the session, AI gives a quick score and coaching notes

This would be a single HTML file with a text chat interface — no server needed initially (API key stored locally). We'd add voice in the next iteration.

---

## Tech Architecture (For Reference)

```
┌─────────────────────────────────────────────┐
│           WCE Sales Script Helper           │
│         (Your Existing HTML App)            │
├─────────────────────────────────────────────┤
│                                             │
│  [Script Tab] [Rebuttals] [AI Practice]     │
│                    ▲            │            │
│                    │            ▼            │
│              ┌─────────────────────┐        │
│              │   Practice Engine    │        │
│              │                     │        │
│              │  • Persona Manager  │        │
│              │  • Scenario Builder │        │
│              │  • Score Engine     │        │
│              └────────┬────────────┘        │
│                       │                     │
└───────────────────────┼─────────────────────┘
                        │
            ┌───────────┼───────────┐
            │           │           │
     Phase 1 & 3    Phase 2A    Phase 2B
    ┌───────────┐ ┌───────────┐ ┌───────────┐
    │ GPT-4o-   │ │ Groq STT  │ │ OpenAI    │
    │ mini      │ │ GPT-4o-   │ │ Realtime  │
    │ (text)    │ │ mini      │ │ API       │
    │           │ │ ElevenLabs│ │ (all-in-  │
    │ $0.01/    │ │ TTS       │ │  one)     │
    │ session   │ │ $0.35/    │ │ $0.50/    │
    │           │ │ session   │ │ session   │
    └───────────┘ └───────────┘ └───────────┘
```

---

## AI Prospect Personas (Built From Your Script)

These would be pre-configured and tied to your call phases:

1. **"Defensive Dave"** — Immediately pushes back: "You don't pay off my debt?" / "Sounds too good to be true"
2. **"Worried Wendy"** — Anxious about consequences: "Will I be sued?" / "What about my credit score?"
3. **"Skeptical Steve"** — Needs proof: "How do I know this will work?" / "I just need a loan"
4. **"Emotional Emma"** — Carries guilt: "I don't feel right about not paying" / "I'm scared to fall behind"
5. **"Busy Brian"** — Wants to get off the phone: "Not interested" / "I don't have time for this"
6. **"Custom"** — Rep sets the debt amount, attitude, and specific objections

Each persona follows a realistic arc through your 14 call phases, throwing objections at the moments they'd naturally come up.
