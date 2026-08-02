<div align="center">

<img src="https://raw.githubusercontent.com/ks777-del/Nexus-AI-Gateway/main/frontend/public/logo.png" alt="Nexus AI Gateway" width="80" height="80" />

# Nexus AI Gateway

**The Unified AI Router for the Modern Developer**

[![Version](https://img.shields.io/badge/version-3.8.48-6366f1?style=for-the-badge)](https://github.com/ks777-del/Nexus-AI-Gateway)
[![Node](https://img.shields.io/badge/node-%3E%3D22.0.0-brightgreen?style=for-the-badge&logo=node.js)](https://nodejs.org)
[![License](https://img.shields.io/badge/license-MIT-blue?style=for-the-badge)](LICENSE)
[![Made by](https://img.shields.io/badge/Made%20by-Kshitij%20Singh-a78bfa?style=for-the-badge)](https://github.com/ks777-del)

<p align="center">
  <strong>One gateway. 160+ AI providers. Zero vendor lock-in.</strong>
</p>

<p align="center">
  <a href="#-quick-start">Quick Start</a> ·
  <a href="#-features">Features</a> ·
  <a href="#-documentation">Documentation</a> ·
  <a href="#-installation">Installation</a> ·
  <a href="https://github.com/ks777-del/Nexus-AI-Gateway">GitHub</a>
</p>

---

</div>

## 🌟 What is Nexus AI Gateway?

**Nexus AI Gateway** is a self-hosted, OpenAI-compatible AI proxy and router. It sits between your AI tools — Cursor, Cline, Claude Code, OpenAI Codex — and the underlying AI provider APIs (OpenAI, Anthropic, Google, Mistral, Groq, and 150+ more), giving you:

- **Unified access** to every major AI provider through one endpoint
- **Intelligent routing** with combo fallback chains — if one provider fails, another takes over automatically
- **Full observability** — logs, analytics, cost tracking, and audit trails in a beautiful dashboard
- **Security** — API key management, OAuth, session auth, and role-based access
- **Zero lock-in** — swap providers anytime from the dashboard without touching code

> **Drop-in replacement** for the OpenAI API. Just change your `base_url` to `http://localhost:20128/v1`.

---

## ✨ Features

| Feature | Description |
|---|---|
| 🔀 **160+ Providers** | OpenAI, Anthropic, Gemini, Mistral, Groq, Bedrock, Cohere, Together AI, and more |
| 🛡️ **OpenAI-Compatible** | Drop-in API — no client code changes required |
| ⚡ **Combo Routing** | Chain providers with automatic failover and load balancing |
| 📊 **Live Dashboard** | Real-time logs, analytics, cost tracking, and usage metrics |
| 🔑 **Key Management** | Create, rotate, revoke API keys with per-key provider scoping |
| 🗜️ **Context Compression** | RTK + Caveman compression to reduce token costs |
| 🤖 **MCP + A2A** | Model Context Protocol and Agent-to-Agent protocol support |
| 📦 **Batch API** | Async batch processing compatible with OpenAI Batch API |
| 🌍 **Multi-language UI** | Fully internationalized dashboard (20+ languages) |
| 🖥️ **Electron + PWA** | Runs as desktop app, PWA, or headless server |

---

## 📚 Documentation

| Document | Description |
|---|---|
| 📖 [Developer Guide](./Developer_Guide.md) | Architecture, setup, provider integration, environment variables, CLI reference |
| 💡 [Examples](./Examples.md) | cURL, Python, JavaScript, streaming, tool calling, AI tool configurations |
| 📡 [REST API Reference](./REST-API.md) | All endpoints, request/response schemas, error codes, headers |

---

## 🚀 Quick Start

### 1. Install

```bash
npm install -g nexus-ai-gateway
```

### 2. Run

```bash
nexus serve
```

### 3. Open Dashboard

```
http://localhost:20128
```

### 4. Connect Your Tools

Point any OpenAI-compatible client to:

```
Base URL : http://localhost:20128/v1
API Key  : sk_nexus
```

---

## 📦 Installation

### Requirements

| Requirement | Version |
|---|---|
| Node.js | `>= 22.0.0` |
| npm | `>= 10` |
| OS | Windows, macOS, Linux |

### Install Globally

```bash
npm install -g nexus-ai-gateway
```

### Verify Installation

```bash
nexus --version
# Output: 3.8.48
```

---

## 🖥️ Running the Server

### Start

```bash
nexus serve
```

### Start on a Custom Port

```bash
nexus serve --port 3000
```

### Start with Verbose Logging

```bash
nexus serve --verbose
```

### Stop

```bash
nexus stop
```

### Check Status

```bash
nexus status
```

### Reset Admin Password

```bash
npx nexus-ai-gateway reset-password
```

---

## ⚙️ Configuration

Create a `.env` file in `~/.nexus/` or `~/.omniroute/`:

```env
PORT=20128
NEXUS_SESSION_SECRET=your-secret-here
NEXUS_DATA_DIR=~/.nexus
HTTP_PROXY=          # optional outbound proxy
```

---

## 🔌 Usage Examples

### cURL

```bash
curl http://localhost:20128/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk_nexus" \
  -d '{
    "model": "gpt-4o",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

### Python

```python
from openai import OpenAI

client = OpenAI(
    api_key="sk_nexus",
    base_url="http://localhost:20128/v1",
)

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Hello!"}],
)
print(response.choices[0].message.content)
```

### JavaScript

```javascript
import OpenAI from 'openai';

const client = new OpenAI({
  apiKey: 'sk_nexus',
  baseURL: 'http://localhost:20128/v1',
});

const res = await client.chat.completions.create({
  model: 'gpt-4o',
  messages: [{ role: 'user', content: 'Hello!' }],
});
console.log(res.choices[0].message.content);
```

> 📝 See [Examples.md](./Examples.md) for streaming, tool calling, image generation, and more.

---

## 🛠️ CLI Reference

```bash
nexus serve              # Start the server
nexus stop               # Stop the server
nexus status             # Show server status
nexus logs               # View logs
nexus keys list          # List all API keys
nexus keys create        # Create a new API key
nexus keys revoke <key>  # Revoke an API key
nexus backup             # Backup the database
nexus restore <file>     # Restore from backup
nexus update             # Update to latest version
nexus --version          # Show version
```

---

## 🗺️ Architecture Overview

```
Client Tools (Cursor / Cline / Codex / Claude Code)
                    │
                    ▼  OpenAI-Compatible HTTP
         ┌──────────────────────┐
         │   Nexus AI Gateway   │
         │  ┌────────────────┐  │
         │  │  Auth + Router │  │
         │  └───────┬────────┘  │
         │  ┌───────▼────────┐  │
         │  │ Provider Layer │  │
         │  │ OpenAI·Anthropic│  │
         │  │ Gemini·Mistral │  │
         │  │ Groq·Bedrock+  │  │
         │  └───────┬────────┘  │
         │  ┌───────▼────────┐  │
         │  │  Observability │  │
         │  │ Logs·Analytics │  │
         │  └────────────────┘  │
         └──────────────────────┘
```

---

## 🤝 Contributing

Contributions are welcome! Please open an issue or pull request on [GitHub](https://github.com/ks777-del/Nexus-AI-Gateway).

1. Fork the repository
2. Create your feature branch: `git checkout -b feature/my-feature`
3. Commit your changes: `git commit -m 'Add my feature'`
4. Push to the branch: `git push origin feature/my-feature`
5. Open a pull request

---

## 📄 License

MIT License © 2026 [Kshitij Singh](https://github.com/ks777-del)

---

<div align="center">

**Made with ❤️ by [Kshitij Singh](https://github.com/ks777-del)**

⭐ Star this project on [GitHub](https://github.com/ks777-del/Nexus-AI-Gateway) if you find it useful!

</div>
