# 🌾 AmaderKrishi AI

[![Python](https://img.shields.io/badge/Python-3-blue?style=flat-square&logo=python)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.115-009688?style=flat-square&logo=fastapi)](https://fastapi.tiangolo.com/)
[![Google Gemini](https://img.shields.io/badge/Google_Gemini-AI-4285F4?style=flat-square&logo=google)](https://aistudio.google.com/)
[![MySQL](https://img.shields.io/badge/MySQL-8-4479A1?style=flat-square&logo=mysql)](https://www.mysql.com/)

An AI-powered Bengali-language agricultural assistant for farmers in Bangladesh. Ask farming questions through **text, images, and voice** — get short, practical advice in Bangla, detect crop diseases from a photo, and keep every conversation in a chat history like ChatGPT.

Built with a **FastAPI** backend, **Google Gemini AI**, **MySQL**, and a vanilla **HTML/CSS/JS** frontend. No frontend framework, no build step — run the backend and the UI is served at the same address.

## 📸 Screenshots

| Chat — Bangla farming advice | Chat — follow-up conversation |
|---|---|
| ![Chat](Sample/chat1.png) | ![Chat](Sample/chat2.png) |

| Login | Sign up |
|---|---|
| ![Login](Sample/login_page.png) | ![Sign up](Sample/signup_page.png) |

## 📑 Table of Contents

- [Features](#-features)
- [How it works](#-how-it-works)
- [Tech Stack](#-tech-stack)
- [Project Structure](#-project-structure)
- [Quick Start](#-quick-start)
- [Configuration](#-configuration)
- [API Overview](#-api-overview)
- [Troubleshooting](#-troubleshooting)
- [Limitations](#-limitations)
- [Team](#-team)
- [License](#-license)

## ✨ Features

| Feature | Description |
|---|---|
| 💬 Text chat | Ask agriculture questions in Bengali, get short, clear answers in Bangla |
| 📷 Image upload | Upload a crop photo — Gemini vision identifies the disease and suggests cause, treatment, and prevention |
| 🎤 Voice chat | Speak in Bengali (Google Chrome required — uses the Web Speech API) |
| 📚 Chat history | Every conversation is saved per session, with titles auto-generated from the first message |
| 👤 Multi-user | Register / log in — each user sees only their own sessions and history |

## 🔄 How it works

1. The farmer writes (or speaks) a question in Bangla, or uploads a crop photo.
2. The backend builds a prompt from a Bangla system prompt: answer in simple Bangla, keep it short and actionable, name local pesticides and fertilizers, and for disease questions always cover cause + solution + prevention.
3. **Text** goes to the Gemini text model; **images** go to the Gemini vision model with a structured prompt (disease name, cause, solution, prevention).
4. The question, the AI answer, and the image (when sent) are saved to MySQL under the user's chat session, so the history survives across visits.

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Backend | Python 3, FastAPI 0.115, Uvicorn |
| AI | Google Gemini (`google-generativeai` SDK) — text + vision |
| Database | MySQL 8 (via `mysql-connector-python`), auto-created on first run |
| Frontend | Vanilla HTML/CSS/JS (single `index.html`), Web Speech API for voice input |
| Auth | Email + password, passwords stored as SHA-256 hashes (see [Limitations](#-limitations)) |

## 📁 Project Structure

```text
.
├── main.py            # FastAPI app — all routes, Gemini setup, prompts
├── database.py        # MySQL connection, tables, users/sessions/messages
├── requirements.txt   # Python packages
├── .env.example       # Environment variable template
├── frontend/
│   └── index.html     # Complete UI (served by the backend at /)
└── Sample/            # UI screenshots used in this README
    ├── chat1.png
    ├── chat2.png
    ├── login_page.png
    └── signup_page.png
```

## ⚡ Quick Start

Prerequisites: **Python 3.10+** and a running **MySQL 8** server.

### 1. Clone and install dependencies

```bash
git clone https://github.com/marufHtuhin/AmaderKrishi_AI.git
cd AmaderKrishi_AI
pip install -r requirements.txt
```

### 2. Get a Gemini API key (free)

1. Visit [Google AI Studio](https://aistudio.google.com/apikey)
2. Sign in with your Google account
3. Click **"Get API Key"** and copy it

### 3. Configure environment variables

```bash
cp .env.example .env
```

Then edit `.env` and fill in your values:

```env
GEMINI_API_KEY=your-gemini-api-key-here
DB_HOST=127.0.0.1
DB_PORT=3306
DB_USER=root
DB_PASSWORD=your-mysql-password
DB_NAME=amaderkrishi
```

> ⚠️ Never commit the `.env` file — it is already excluded via `.gitignore`.

You do **not** need to create the database by hand: on the first run the app creates the `amaderkrishi` database and the `users`, `sessions`, and `messages` tables automatically.

### 4. Start the backend

```bash
uvicorn main:app --reload
```

If successful, you should see:

```text
✅ আমাদের কৃষি AI চালু হয়েছে!
```

### 5. Open the app

Open **http://localhost:8000** in your browser — the backend serves the full UI at `/`. (You can also open `frontend/index.html` directly, but then it talks to the API at `localhost:8000` only when the backend is running.)

> **Important:** use Google Chrome for voice input, and allow microphone permission when asked.

## ⚙️ Configuration

All settings live in `.env` (copy from `.env.example`).

| Variable | Required | Default | Notes |
|---|---|---|---|
| `GEMINI_API_KEY` | Yes | — | Free key from Google AI Studio. **The app refuses to start without it.** |
| `DB_HOST` | No | `127.0.0.1` | MySQL host |
| `DB_PORT` | No | `3306` | MySQL port |
| `DB_USER` | No | `root` | MySQL user |
| `DB_PASSWORD` | No | _(empty)_ | MySQL password |
| `DB_NAME` | No | `amaderkrishi` | Database name (auto-created on first run) |

The Gemini model is set in `main.py` (`text_model` / `vision_model`). See [Limitations](#-limitations) — the current default model retires on 1 Dec 2026 and needs updating.

## 🔌 API Overview

Base URL (dev): `http://localhost:8000`. Interactive docs at `/docs`.

| Method | Path | Description |
|---|---|---|
| `GET` | `/` | Serves the frontend UI |
| `POST` | `/api/register` | Register a new user (`first_name`, `last_name`, `email`, `phone`, `password` ≥ 6 chars) |
| `POST` | `/api/login` | Log in, returns `{user_id, username, email}` |
| `POST` | `/api/sessions` | Create a new chat session |
| `GET` | `/api/sessions/{user_id}` | List a user's sessions |
| `GET` | `/api/history/{session_id}` | Full chat history of a session |
| `POST` | `/api/chat` | Text chat — returns the AI reply, saves both messages |
| `POST` | `/api/chat-image` | Image + optional message — Gemini vision analysis, saves the image with the chat |
| `DELETE` | `/api/sessions/{session_id}` | Delete a chat session (and its messages) |

Requests identify the user by `user_id` — there is no token auth (see [Limitations](#-limitations)).

## 🔧 Troubleshooting

| Symptom | Cause / Fix |
|---|---|
| App fails to start: `GEMINI_API_KEY environment variable is not set` | Copy `.env.example` to `.env` and add a real key from Google AI Studio. |
| `Database connection problem` at startup | Make sure MySQL is running and `DB_HOST` / `DB_PORT` / `DB_USER` / `DB_PASSWORD` in `.env` are correct. |
| `CORS error` in the browser | Check the backend is running at `http://localhost:8000`. |
| `AI সমস্যা` / `ছবি বিশ্লেষণে সমস্যা` on chat | The Gemini call failed — wrong/expired key, a retired model name, or no network. Check the backend log for the exact error. |
| Voice input not working | Use Google Chrome and allow microphone permission. Other browsers may not support the Web Speech API. |
| Empty / generic answers | The model may be falling back or the key may be rate-limited — check the backend log. |

## ⚠️ Limitations

- **The Gemini model in the code retires on 1 Dec 2026.** `main.py` uses `gemini-2.5-flash` for both text and vision; Google deprecated the 2.5 series and shuts it down on 1 Dec 2026. After that date every chat returns an error until the model is changed — update `text_model` and `vision_model` in `main.py` to a current model (for example `gemini-3.5-flash-lite`, the free-tier replacement) and restart.
- **Auth is basic, by design for a demo.** There is no JWT or token — requests identify users by `user_id`, so anyone who knows a `user_id` can read that user's sessions. Passwords are stored as unsalted SHA-256 hashes. Do not use this auth as-is for a production service.
- **Disease detection is Gemini vision analysis, not a trained model.** The AI describes what it sees in the photo and suggests a likely disease and treatment — always verify the advice with a local agriculture officer before spraying or applying anything.
- **Voice input needs Chrome** (Web Speech API) and a microphone.
- **MySQL is required** — there is no SQLite fallback.
- **No automated tests and no CI** — verify changes by running the app locally.

## 👥 Team

Built by [Md. Maruf Hasan Tuhin](https://marufhtuhin.vercel.app/) ([GitHub](https://github.com/marufHtuhin)).

## 📄 License

Open for educational use.
