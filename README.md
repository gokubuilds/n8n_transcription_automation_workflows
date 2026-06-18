# Telegram AI Assistant with Voice Summarization

## Overview

This project is an intelligent Telegram chatbot built using n8n that can:

* Respond to text messages using GPT-4o via OpenRouter
* Maintain conversational memory per user
* Process Telegram voice/audio messages
* Convert speech to text using AssemblyAI
* Generate concise summaries of audio content
* Send AI-generated responses back to Telegram

The workflow automatically detects whether the incoming message is text or audio and routes it through the appropriate processing pipeline.

---

## Features

### Text Chat Assistant

* Receives messages from Telegram
* Uses GPT-4o through OpenRouter
* Maintains user-specific conversation memory
* Generates contextual responses
* Sends responses directly back to Telegram

### Voice Message Processing

* Detects incoming audio messages
* Downloads audio from Telegram
* Uploads audio to AssemblyAI
* Performs speech-to-text transcription
* Waits for transcription completion
* Generates AI-powered summaries
* Returns summarized content to Telegram

### Conversation Memory

The chatbot stores conversation context using memory buffers.

Benefits:

* Maintains context across messages
* Enables follow-up questions
* Improves conversational quality
* Creates a personalized experience

---

# Architecture

```text
Telegram User
      │
      ▼
Telegram Trigger
      │
      ▼
Message Type Detection
      │
 ┌────┴────┐
 │         │
 ▼         ▼
Text      Audio
 │          │
 ▼          ▼
GPT-4o   AssemblyAI
 │          │
 ▼          ▼
Reply   Transcription
 │          │
 ▼          ▼
Telegram  GPT-4o Summary
               │
               ▼
           Telegram
```

---

# Workflow Breakdown

## 1. Telegram Trigger

Receives incoming Telegram updates.

Supported events:

* Text messages
* Audio messages

---

## 2. Message Type Detection

An IF node checks:

```javascript
$json.message.audio
```

### If FALSE

Message is treated as text.

Flow:

```text
Telegram
  ↓
AI Agent
  ↓
Send Message
```

### If TRUE

Message is treated as audio.

Flow:

```text
Telegram
  ↓
Download Audio
  ↓
AssemblyAI
  ↓
Transcription
  ↓
Summary
  ↓
Telegram
```

---

# Text Chat Pipeline

## AI Agent

Uses GPT-4o through OpenRouter.

Prompt Logic:

* Understand user messages
* Answer questions
* Continue conversations naturally
* Generate contextual replies

## Memory Buffer

Stores chat history using:

```text
Telegram Chat ID
```

This ensures each user has an isolated conversation context.

## Telegram Response

Returns AI-generated output directly to the user.

---

# Audio Processing Pipeline

## Download Audio

Retrieves Telegram audio file using:

```text
file_id
```

## Upload to AssemblyAI

Audio file is uploaded to:

```text
https://api.assemblyai.com/v2/upload
```

AssemblyAI returns:

```json
{
  "upload_url": "..."
}
```

---

## Create Transcription Job

The workflow submits the uploaded audio for transcription.

Endpoint:

```text
https://api.assemblyai.com/v2/transcript
```

Returns:

```json
{
  "id": "transcript_id"
}
```

---

## Wait Node

Introduces a delay before polling the transcription status.

Purpose:

* Reduce API requests
* Allow transcription processing

---

## Poll Transcription Status

Checks:

```text
https://api.assemblyai.com/v2/transcript/{id}
```

If status:

```text
completed
```

Continue to summarization.

Otherwise:

```text
Retry transcription check
```

---

# AI Summarization

Once the transcript is available:

* GPT-4o receives the transcription text
* Generates a concise summary
* Preserves important information
* Maintains logical flow

Special Handling:

* If transcript is too short
* If content cannot be summarized meaningfully

The original content is returned without modification.

---

# Technologies Used

## Workflow Automation

* n8n

## Messaging Platform

* Telegram Bot API

## AI Models

* GPT-4o
* OpenRouter

## Speech Recognition

* AssemblyAI

## Memory

* n8n Memory Buffer Window

---

# Required Credentials

## Telegram

Create a Telegram Bot:

```text
@BotFather
```

Required:

```env
TELEGRAM_BOT_TOKEN=
```

---

## OpenRouter

Required:

```env
OPENROUTER_API_KEY=
```

Model:

```text
openai/gpt-4o
```

---

## AssemblyAI

Required:

```env
ASSEMBLYAI_API_KEY=
```

---

# Setup

## Import Workflow

1. Open n8n
2. Import workflow JSON
3. Configure credentials

## Configure Telegram

Create Telegram credentials in n8n.

## Configure OpenRouter

Add API key.

## Configure AssemblyAI

Add API key.

## Activate Workflow

Enable workflow and send a message to your Telegram bot.

---

# Example Use Cases

### Text Chat

User:

```text
What is Retrieval Augmented Generation?
```

Bot:

```text
RAG is a technique that combines retrieval systems with language models...
```

---

### Voice Summarization

User sends:

```text
5-minute meeting recording
```

Bot returns:

```text
Summary:
- Discussed project milestones
- Finalized deployment plan
- Assigned testing responsibilities
```

---

# Future Enhancements

* Multi-language transcription
* RAG integration
* PDF summarization
* Image understanding
* Calendar scheduling
* Browser automation via MCP
* Email assistant
* CRM integration
* WhatsApp support

---

# Author

Gokul Soundarapandian

AI Engineer | GenAI Enthusiast | Automation Developer
