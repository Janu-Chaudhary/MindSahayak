# MindSahayak – AI-Powered Mental Health Companion for Students

> An AI-driven, multilingual chatbot that provides emotional support, detects early signs of distress, and offers self-help tools tailored for students in India.

---

## 📌 Authorship Note

> This is a **forked repository** from a group project. The terminal chatbot (`terminal_chat.py`), emotion detection system (`emotion_detector.py`), and crisis detection engine (`advanced_crisis_detector.py`) were **designed and developed entirely by me**. The original code was pushed to the shared repo by a teammate on my behalf — this fork exists to properly attribute my individual contribution for portfolio and CV purposes.

---

## Problem Statement

Students in India — especially in Tier 2 and Tier 3 cities — face growing mental health challenges such as exam anxiety, chronic stress, and depression. Barriers like social stigma, language differences, and limited access to professionals mean many students suffer in silence.

---

## Objective

Build a scalable, privacy-first AI companion (web + mobile) that:

- Provides empathetic conversation and emotional support
- Detects early warning signs using sentiment & emotion analysis
- Offers self-help resources and mood tracking
- Supports multilingual interactions (English, Hindi, Hinglish)
- Optionally escalates to human therapists or peer listeners when required

---

## My Contribution — Terminal Chatbot Backend

While the group worked on the frontend (React Native) and overall architecture, I was solely responsible for building the **entire AI chatbot backend** as a terminal-based application. This includes three files:

| File | Description |
|---|---|
| `terminal_chat.py` | Core chatbot — conversation loop, session management, LLM integration, crisis escalation |
| `emotion_detector.py` | Real-time emotion classification using a RoBERTa transformer model |
| `advanced_crisis_detector.py` | Rule-based + sentiment-based crisis risk scoring engine |

---

## How the Chatbot Works

### Persona
The chatbot runs as **"Aanya"** — a culturally aware counselor persona tuned specifically for Indian students, with understanding of academic pressure, family dynamics, and the cultural stigma around mental health. Responses use a warm, sisterly tone and naturally incorporate Hindi phrases where appropriate.

### 1. Emotion Detection (`emotion_detector.py`)

Every user message is passed through a HuggingFace pipeline using `SamLowe/roberta-base-go_emotions`, which scores 27 emotion labels. These are:

- Mapped to standard categories (`grief → sadness`, `nervousness → anxiety`, etc.)
- Normalised to sum to 1.0
- Stored in a rolling per-user history (capped at 100 entries)
- Fed back into the LLM prompt as emotional context for every reply

```python
self.classifier = pipeline(
    "text-classification",
    model="SamLowe/roberta-base-go_emotions",
    top_k=5,
    device=self.device
)
```

If a crisis is detected, negative emotion scores (sadness, fear, anxiety) are boosted by 1.5× to better reflect the severity before being passed to the LLM.

### 2. Crisis Detection (`advanced_crisis_detector.py`)

A two-layer risk scoring system:

**Layer 1 — Rule-based phrase matching**
```python
crisis_phrases  = [r'kill\s*(my)?self', r'end\s*(it\s*all|my\s*life)', 'suicid']  # +0.5 risk
warning_phrases = ['hopeless', 'helpless', 'worthless', 'no one cares']           # +0.2 risk
safe_phrases    = ['struggling with', 'hard time', 'stress about']                # bypass detection
```

**Layer 2 — Sentiment analysis**
- Runs `distilbert-base-uncased-finetuned-sst-2-english` on the message
- Negative sentiment contributes up to +0.4 to the risk score

**Final risk score → response tier:**

| Score | Level | Action |
|---|---|---|
| ≥ 0.7 or crisis phrase match | High | Immediate helplines + urgent message |
| ≥ 0.4 or warning phrase | Medium | Gentle check-in + helpline mention |
| < 0.4 | Low | Normal conversation |

### 3. Conversation Manager (`terminal_chat.py`)

The `UserChatManager` class ties everything together using **Google Gemini 1.5 Flash** via LangChain:

- Persistent sessions — each user gets a UUID, history saved to JSON and reloaded across sessions
- Before every LLM call, a structured context block is injected containing:
  - Dominant emotion + intensity + trend from last 5 messages
  - Key topics detected (studies, family, relationships, career, stress, sleep, health)
  - Last 5 messages with timestamps and per-message emotion tags
- A `needs_follow_up` flag is written to the user's JSON if they were ever in a crisis session

```python
formatted_input = f"{context_str}\n\nUser: {user_input}"

response = self.conversation_chain.invoke(
    {"input": formatted_input},
    {"configurable": {"session_id": user_id}}
)
```

---

## Setup & Running the Chatbot

### Prerequisites
- Python 3.9+
- A Google Gemini API key

### Installation

```bash
git clone https://github.com/Janu-Chaudhary/MindSahayak.git
cd MindSahayak

pip install -r requirements.txt
```

Create a `.env` file:

```
GOOGLE_API_KEY=your_gemini_api_key_here
```

### Run

```bash
python terminal_chat.py
```

```
==================================================
        Welcome to Mental Health Chatbot
              Type 'exit' to quit
==================================================

Are you a new or existing user?
1. New User
2. Existing User
3. Exit
```

- New users get a UUID to reuse across sessions
- Type `emotions` anytime to see your real-time emotion breakdown
- Type `exit` to end the session and view your emotion summary

Chat histories are saved locally to `user_chat_histories/user_<uuid>.json`.

---

## Full Project Tech Stack

| Layer | Technology |
|---|---|
| Chatbot LLM | Google Gemini 2.5 Flash (via LangChain) |
| Emotion model | `SamLowe/roberta-base-go_emotions` (HuggingFace) |
| Crisis model | `distilbert-base-uncased-finetuned-sst-2-english` |
| Chatbot framework | LangChain, LangChain-Google-GenAI |
| Frontend | React Native |
| Chatbot Backend | Flask |
| Database | MongoDB |


---

## MVP Outcomes

- ✅ Functional AI chatbot with emotional conversation and persistent memory
- ✅ Real-time sentiment and emotion detection on every message
- ✅ Tiered crisis detection with automatic helpline escalation
- ⬜ Self-help dashboard with videos and mood tracking tools
- ⬜ Multilingual support (English + Hindi minimum)


---

## Crisis Helplines Referenced

The chatbot surfaces these in high-risk situations:

- **Vandrevala Foundation**: 1860-2662-345 / 1800-2333-330 (24/7, free)
- **iCall**: +91-9152987821 (Mon–Sat, 10am–8pm, WhatsApp available)
- **AASRA**: +91-9820466626 (24/7)

---

## Vision

To build a trusted, private, and empathetic AI companion that empowers students to take control of their mental well-being, encourages emotional openness, and provides accessible support regardless of location, language, or background.
