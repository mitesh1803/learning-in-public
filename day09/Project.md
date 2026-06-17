![Status](https://img.shields.io/badge/Status-Building-blue?style=for-the-badge&logo=github)
![Stack](https://img.shields.io/badge/Stack-Bun%20%7C%20Groq%20%7C%20Deepgram-black?style=for-the-badge)

# 🎙️ HireMind — Day 09 Devlog

## 📝 Update: Voice Pipeline is Fully Alive
**Project:** HireMind — Real-time, voice-driven AI technical interviewer

**Date:** June 13, 2026

---

## 🚀 What is HireMind?

HireMind is a real-time, voice-driven AI technical interviewer designed to conduct personalized software engineering mock interviews.

```
You submit your GitHub URL
        ↓
AI scrapes your public repos — names, languages, topics, stars
        ↓
Generates personalized interview questions around YOUR projects
        ↓
Conducts the full interview by voice in real time
        ↓
Produces a structured technical score (0–10) + feedback report
```

The entire experience runs over low-latency WebSockets — no page refreshes, no lag, no pre-scripted questions.

---

## ✅ What Shipped Today

| Feature | Status |
|---|---|
| Full STT → LLM → TTS pipeline over WebSockets | ✅ Done |
| Sentence-level audio streaming (low-latency TTS) | ✅ Done |
| Echo prevention — mic mutes while AI speaks, unmutes after queue drains | ✅ Done |
| Redis as live conversation memory — no DB reads during interview | ✅ Done |
| DB flush on disconnect — full transcript saved to PostgreSQL | ✅ Done |
| Groq evaluator — scores transcript 0–10 with structured feedback | ✅ Done |
| Silent candidate edge case — score 0, no wasted LLM call | ✅ Done |

---

## 🔄 The Full Interview Flow

```
User submits GitHub URL
        ↓
Backend scrapes public repo metadata (names, descriptions, languages, topics)
        ↓
Groq + Llama-3.1-8b generates personalized question set
        ↓
AI interviewer greets candidate by voice (Deepgram Aura TTS)
        ↓
Candidate answers out loud
        ↓
Deepgram Live STT transcribes speech in real time
        ↓
Groq generates follow-up question based on answer + context
        ↓
Response streamed back as voice — sentence by sentence
        ↓
Loop until interview complete
        ↓
Groq evaluator scores full transcript → report saved to PostgreSQL
```

---

## 🐛 Hardest Bug of the Day

**Symptom:** Deepgram was receiving audio. Returning empty transcripts. No error message. Just silence.

**Two hours of logs later — root cause found:**

```
Timeline of what was happening:

t=0ms   User audio starts streaming over WebSocket
t=0ms   First 3 audio chunks sent  ← WebM container HEADER
t=47ms  Deepgram WebSocket opens
t=48ms  Remaining audio arrives at Deepgram

Result: Deepgram never received the container header
        → can't decode any subsequent audio
        → returns empty transcripts with no error
        → silent failure ❌
```

**Fix:**

```typescript
// Buffer early chunks before Deepgram is ready
const earlyChunks: Buffer[] = [];

ws.on('message', (chunk) => {
  if (!deepgramReady) {
    earlyChunks.push(chunk);   // hold them
    return;
  }
  deepgramSocket.send(chunk);
});

deepgramSocket.on('open', () => {
  deepgramReady = true;
  earlyChunks.forEach(c => deepgramSocket.send(c));  // replay header first
  earlyChunks.length = 0;
});
```

5 lines of code. 2 hours to find.

---

## 💰 Cost Comparison

| Provider | Cost per Interview |
|---|---|
| OpenAI Realtime API | ~$3.00 |
| Groq + Deepgram | ~$0.004 |

**750x cheaper. No GPU infra. Same quality.**

---

## 🏗️ Architecture & Tech Stack

```
HireMind/
├── apps/
│   ├── frontend/    # React 19 SPA — WebSockets, AudioContext, VoiceOrb
│   └── backend/     # Express + WebSocket Server — Groq, Deepgram, Redis, Prisma
├── packages/
│   ├── ui/          # Shared React components
│   ├── eslint-config/
│   └── typescript-config/
```

| Layer | Technology |
|---|---|
| **Runtime** | Bun |
| **Monorepo** | Turborepo |
| **Frontend** | React 19, React Router v7, TailwindCSS v4, Radix UI |
| **Backend** | Express, WebSocket (ws), Axios |
| **AI — Interviewer** | Groq SDK — Llama-3.1-8b-instant |
| **STT** | Deepgram Live Speech-to-Text |
| **TTS** | Deepgram Aura (aura-asteria voice model) |
| **Database** | PostgreSQL via Prisma ORM |
| **Session State** | Redis — ephemeral conversation buffer |

---

## 🔊 Audio Pipeline Detail

### Echo Prevention

```
AI starts speaking
        ↓
Frontend mutes microphone immediately
        ↓
TTS audio chunks stream into HTML5 AudioContext queue
        ↓
Queue drains — last chunk plays
        ↓
Microphone unmutes automatically
        ↓
Candidate speaks — Deepgram picks up clean audio
```

Without this: mic picks up the AI's voice → feeds back into STT → AI interviews itself.

### Why Sentence-Level Streaming?

Streaming TTS word by word creates choppy audio. Waiting for the full response creates a lag spike. Sentence-level is the Goldilocks zone:

```
Full response generated:  "That's a great point. Can you walk me through how you handled..."
Streamed as:
  → "That's a great point."          [plays immediately]
  → "Can you walk me through..."     [plays next, seamless]
```

---

## 📊 Redis Session Architecture

```
Interview starts → session key created in Redis
        ↓
Each Q&A turn appended to Redis conversation buffer
        ↓
Groq reads full buffer for context on every response
        ↓ (no PostgreSQL reads during live interview)
Interview ends / WebSocket disconnects
        ↓
Full buffer flushed from Redis → saved to PostgreSQL
        ↓
Groq evaluator scores the saved transcript
        ↓
Score + feedback written to DB → available on dashboard
```

Redis holds state during the interview (fast). PostgreSQL stores the permanent record (durable).

---

## 📈 Next Up

**Master-Slave Multi-Agent Architecture**

```
Master Agent (Claude Haiku)
  → holds full interview state
  → decides when to move to next topic
  → tracks time and coverage

Slave Agents (Groq / Llama)
  → Question Generator     — produces next question from context
  → Answer Evaluator       — scores individual answers in real time
  → Final Scorer           — produces end-of-interview report
```

Each agent does one job. Master orchestrates. No single LLM call does everything.

---

## 🔗 Links

* [GitHub Repository](https://github.com/mitesh1803/HireMind)
* [Deepgram Live STT Docs](https://developers.deepgram.com/docs/getting-started-with-live-streaming-audio)
* [Groq SDK Docs](https://console.groq.com/docs/openai)
* [Deepgram Aura TTS](https://developers.deepgram.com/docs/tts-websocket)
* [Turborepo Docs](https://turbo.build/repo/docs)

---
*Follow the build — updates daily. ⭐ the repo if you find it useful.*