# AI Agent Telegram - Workflow

An n8n workflow that turns a Telegram bot into a multi-modal AI agent:
- **Text** messages
- **Voice** notes (via Deepgram transcription + ffmpeg)
- **Images** (sent with captions)

Powered by LangChain agents and OpenRouter/OpenAI-compatible models.

---

<img width="1200" alt="n8n-workflow" src="https://github.com/user-attachments/assets/f0e27445-3b88-4487-a759-4822ea373b37" />


## Table of Contents

1. [Features](#features)
2. [Prerequisites](#prerequisites)
3. [Installation & Setup](#installation--setup)
4. [Configuration](#configuration)
5. [Workflow Structure](#workflow-structure)
6. [Usage](#usage)
7. [Customization](#customization)
8. [Troubleshooting](#troubleshooting)

---

## Features

- **Telegram Trigger**: Listens for incoming messages.
- **Switch**: Routes based on message type (text, voice, image).
- **Voice Processing**:
  - Downloads voice file via Telegram API
  - Converts to 16 kHz mono with **ffmpeg**
  - Sends to Deepgram for speech‑to‑text
- **Text & Image Handling**:
  - Text messages go directly to LangChain agent
  - Images are resized (≤ 800×800) and passed (with caption) to an image‑capable model
- **LangChain Agents**:
  - Separate agents for text-only, voice‑derived text, and image inputs
  - Custom system prompts per node
- **Output**: Sends AI’s reply back to the user in Telegram

---

## Prerequisites

- **n8n** v0.200.0 or newer (self‑hosted or desktop)
- **Node.js** (if running n8n locally)
- **ffmpeg** installed (for audio transcoding)
- **Docker** (optional, if you prefer containerized setup)
- **Telegram Bot**
- **Deepgram** account & API token
- **OpenRouter** (or other LLM API) token for LangChain nodes

---

## Installation & Setup

1. **Clone this repo**
   ```bash
   git clone https://github.com/04yashgautam/n8n-ai-agent-telegram
   cd n8n-ai‑agent‑telegram
   ```

2. **Install n8n**
   - **Locally**
     ```bash
     npm install -g n8n
     n8n start
     ```
   - **Docker Compose**
     ```yaml
     version: '3'
     services:
       n8n:
         image: n8nio/n8n
         ports:
           - '5678:5678'
         volumes:
           - ./n8n:/home/node/.n8n
         environment:
           - N8N_BASIC_AUTH_ACTIVE=true
           - N8N_BASIC_AUTH_USER=<user>
           - N8N_BASIC_AUTH_PASSWORD=<pw>
     ```
   - Start with `docker-compose up -d`

3. **Import Workflow**
   - In n8n Editor, click **Import** → **From File** → select `AI Agent Telegram.json`
   - Activate the workflow.

---

## Configuration

### Environment Variables (or n8n Credential Nodes)

| Variable                       | Description                                   |
| ------------------------------ | --------------------------------------------- |
| `TELEGRAM_BOT_TOKEN`           | Your Telegram Bot API token                   |
| `DEEPGRAM_API_TOKEN`           | Token for Deepgram transcription service      |
| `OPENROUTER_API_KEY`           | API key for your LLM (LangChain)              |
| `VOICE_FILE_PATH`              | Local folder path for temporary audio files   |
| `IMAGE_FILE_PATH`              | Local folder path for downloaded images       |

> **Sticky Notes** in the workflow highlight where to add/change tokens and file paths.

---

## Workflow Structure

1. **Telegram Trigger**
   - Listens for any incoming updates (text, voice, photo).
2. **Switch Node**
   - **Voice** → `Get voice file` → `Download Request` → `Read/Write File` → `Execute Command` (ffmpeg) → `Extract from File` → `Deepgram Transcribe` → `Voice Only` → **AI Agent**
   - **Text** → `Text Only` → **AI Agent1**
   - **Image** → `Text+Image` → `Get image file` → `Resize` → **AI Agent2**
3. **LangChain Agents**
   - **AI Agent** (voice/text): System prompt contains custom tools & instructions
   - **AI Agent1** (text): Short max‑token responses
   - **AI Agent2** (image): Passthrough binary images + caption
   - All use OpenRouter‐compatible chat models
4. **Output**
   - Strips `<think>` tags, sends response via `Output` node

---

## Usage

1. **Start** your n8n instance.
2. **Message** your Telegram bot one of:
   - **Text**: any question or prompt
   - **Voice note**: speak your query
   - **Image**: send a photo with an optional caption
3. **Reply**: the bot processes and responds within seconds.

---

## Customization

- **System Prompts**: Edit per-agent system messages to tweak behavior.
- **Model Selection**: Swap out `lmChatOpenRouter` nodes for other providers (OpenAI, Hugging Face, etc.).
- **Memory**: Enable & configure the `Simple Memory` node to maintain conversation history.
- **Audio Settings**: Adjust ffmpeg `-ar` (sample rate) and `-ac` (channels) as needed.
- **Image Resize**: Change max dimensions in the `Resize` node.

---

## Troubleshooting

- **No Response**:
  - Check **Sticky Notes** to ensure tokens are set.
  - Verify file paths exist and are writable.
- **ffmpeg Errors**:
  - Ensure ffmpeg is installed on host or in Docker container.
- **Deepgram Failures**:
  - Confirm your token and endpoint are correct.
- **n8n Logs**:
  - `n8n start --tunnel` (for remote testing)
  - Check `~/.n8n/.log` or Docker logs.

---

*Happy automating!*  
