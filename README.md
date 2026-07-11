# Chipilito – Modern Local AI Chatbot with User Authentication

![Chipilito logo](https://github.com/noeljoan/chipilito/blob/main/public/logo_small.png)

A lightweight, **offline-first** AI chatbot that runs on your machine. Features:

- Chat with **Ollama** or **Google Gemini** models
- **File analysis**: PDF, DOCX, XLSX, images
- **User accounts** with password authentication & API keys
- **Real math calculations** (no LLM guesswork)

---

## 🛠️ System Requirements (Minimum)

| Component | Requirement |
|-----------|------------|
| **OS** | Windows 10+, macOS 12+, or Linux (Ubuntu 20.04+) |
| **CPU** | Dual-core x64 (2 GHz+) |
| **RAM** | 4 GB minimum (8 GB recommended for local LLMs) |
| **Disk** | 500 MB free space (code + dependencies) |
| **Node.js** | 18.0.0+ (tested on v20) |
| **npm** | 9.0.0+ |

> **Note**: If you only use the Gemini cloud API, you don't need Ollama or extra disk space. Local LLMs (gemma2, llama3) need model files (~1–8 GB each).

---

## 🚀 Quick Start

`ash
git clone https://github.com/noeljoan/chipilito.git
cd chipilito
npm ci
cp .env.example .env  # edit optional vars
npm start
`

Server runs on http://localhost:3000.

---

## 🔐 User Authentication

| Endpoint | Method | Purpose |
|----------|--------|---------|
| /api/auth/register | POST {name, password} | Create account → returns pi_key |
| /api/auth/login | POST {name, password} | Returns pi_key |
| /api/auth/profile | GET x-api-key | Get profile |
| /api/auth/profile | PUT x-api-key + {"name":"new"} | Update display name |

---

## ✅ What's Working

- User registration & login
- Password hashing (cryptjs)
- API key generation
- Profile management
- File parsing (PDF, DOCX, XLSX)

---

MIT License – fork away! 🚀
