# 🧠 Nexus AI Gateway — Developer Guide

> Built by **Kshitij Singh** | [GitHub](https://github.com/ks777-del/Nexus-AI-Gateway)

---

## Table of Contents

1. [Overview](#overview)
2. [Architecture](#architecture)
3. [Prerequisites](#prerequisites)
4. [Installation & Setup](#installation--setup)
5. [Environment Variables](#environment-variables)
6. [Running the Server](#running-the-server)
7. [Custom Provider Integration](#custom-provider-integration)
8. [Authentication Modes](#authentication-modes)
9. [API Key Management](#api-key-management)
10. [Combo Routing (Fallback Chains)](#combo-routing-fallback-chains)
11. [CLI Reference](#cli-reference)
12. [Troubleshooting](#troubleshooting)

---

## Overview

**Nexus AI Gateway** is a self-hosted, unified AI router that sits between your tools (Cursor, Cline, Codex, Claude Code) and AI provider APIs (OpenAI, Anthropic, Google, etc.). It routes requests intelligently, applies fallback logic, handles authentication, and provides a full dashboard for observability.

### Key Capabilities

| Feature | Description |
|---|---|
| 160+ Providers | OpenAI, Anthropic, Gemini, Mistral, Groq, Bedrock, and more |
| OpenAI-Compatible API | Drop-in replacement — point any OpenAI client to `localhost:20128/v1` |
| Combo Routing | Chain multiple providers with automatic fallback |
| RTK + Caveman Compression | Request/response context compression to reduce token costs |
| MCP / A2A Support | Model Context Protocol and Agent-to-Agent protocol support |
| Dashboard | Real-time monitoring, logs, analytics, and settings |
| Desktop + PWA | Runs as a local service or progressive web app |

---

## Architecture

```
┌─────────────────────────────────────────┐
│           CLIENT TOOLS                  │
│  Cursor / Cline / Codex / Claude Code   │
└────────────────┬────────────────────────┘
                 │ HTTP (OpenAI-compatible)
                 ▼
┌─────────────────────────────────────────┐
│        NEXUS AI GATEWAY                 │
│                                         │
│  ┌──────────┐  ┌──────────────────┐     │
│  │  Auth    │  │  Request Router  │     │
│  │ Service  │  │  (Combo Engine)  │     │
│  └──────────┘  └────────┬─────────┘     │
│                         │               │
│  ┌──────────────────────▼──────────┐    │
│  │         Provider Adapters       │    │
│  │  OpenAI │ Anthropic │ Gemini    │    │
│  │  Mistral│ Groq │ Bedrock │ ...  │    │
│  └──────────────────────┬──────────┘    │
│                         │               │
│  ┌──────────────────────▼──────────┐    │
│  │     Observability Layer         │    │
│  │  Logs │ Analytics │ Audit Trail │    │
│  └─────────────────────────────────┘    │
└─────────────────────────────────────────┘
```

### Key Directories

```
nexus-omni/
├── src/
│   ├── auth/           # Authentication & session management
│   ├── dashboard/      # Next.js pages for the dashboard UI
│   ├── server/         # Core API server & route handlers
│   ├── shared/         # Shared components, hooks, constants
│   │   ├── components/ # React UI components
│   │   ├── constants/  # Provider configs, sidebar items
│   │   └── services/   # API key resolver, model sync
│   ├── mitm/           # MITM proxy handlers (TPROXY)
│   ├── sse/            # Server-Sent Events handlers
│   └── types/          # TypeScript type definitions
├── bin/
│   ├── nexus.mjs       # Main CLI entry point
│   └── cli/            # CLI subcommands
├── dist/               # Compiled production build
└── frontend/           # Login page assets
```

---

## Prerequisites

- **Node.js** >= 22.0.0 (< 23 or >= 24, < 27)
- **npm** >= 10

```bash
node --version   # Should be 22.x or 24.x
npm --version    # Should be 10+
```

---

## Installation & Setup

### Global Install (Recommended)

```bash
# Install globally
npm install -g nexus-ai-gateway

# Start the server
nexus serve
```

### From Source

```bash
# Clone the repo
git clone https://github.com/ks777-del/Nexus-AI-Gateway.git
cd Nexus-AI-Gateway/nexus-omni

# Install from source (no build needed — dist is included)
npm install -g . --ignore-scripts

# Start
nexus serve
```

### First-Time Setup

1. Open **http://localhost:20128** in your browser
2. Create an admin account on the login page
3. Navigate to **Providers** and add your API keys
4. Configure **Combos** for fallback chains (optional)
5. Point your AI tools to `http://localhost:20128/v1`

---

## Environment Variables

Create a `.env` file in your data directory (`~/.nexus/` or `~/.omniroute/`):

```env
# ─── Core ────────────────────────────────────
PORT=20128                          # Server port (default: 20128)
NEXUS_DATA_DIR=~/.nexus             # Data storage directory

# ─── Auth ─────────────────────────────────────
NEXUS_SESSION_SECRET=your-secret    # Session signing secret (auto-generated if empty)
NEXUS_ADMIN_EMAIL=admin@example.com # Initial admin email (first run only)

# ─── Database ────────────────────────────────
NEXUS_DB_PATH=~/.nexus/nexus.db     # SQLite database path

# ─── Proxy ────────────────────────────────────
HTTP_PROXY=                         # Outbound HTTP proxy (optional)
HTTPS_PROXY=                        # Outbound HTTPS proxy (optional)

# ─── Build Flags ──────────────────────────────
NEXUS_BUILD_PROFILE=minimal         # Build profile: minimal | full
NEXUS_USE_TURBOPACK=1               # Use Turbopack bundler (default: 1)
```

---

## Running the Server

```bash
# Start in foreground
nexus serve

# Start on a custom port
nexus serve --port 3000

# Start with verbose logging
nexus serve --verbose

# Stop the server
nexus stop

# Restart
nexus restart

# Check status
nexus status
```

---

## Custom Provider Integration

To add a custom OpenAI-compatible provider:

### 1. Add via Dashboard

1. Go to **Dashboard → Providers**
2. Click **+ Add Provider**
3. Select **OpenAI Compatible**
4. Enter your base URL and API key
5. Save and test the connection

### 2. Add via Environment Variable

```env
CUSTOM_PROVIDER_BASE_URL=https://your-provider.com/v1
CUSTOM_PROVIDER_API_KEY=your-key
```

### 3. Register in Code (Advanced)

Add to `src/shared/constants/providers/apikey/`:

```typescript
// my-provider.ts
export const MY_PROVIDER = {
  id: "my-provider",
  name: "My Custom Provider",
  baseUrl: "https://api.myprovider.com/v1",
  authType: "apikey" as const,
  models: ["my-model-v1", "my-model-v2"],
};
```

---

## Authentication Modes

| Mode | Description |
|---|---|
| **Local** | Credentials stored in local SQLite DB |
| **OAuth** | GitHub, Google OAuth flow |
| **API Key** | Machine-to-machine via `Authorization: Bearer sk_nexus_...` |
| **Passthrough** | No auth (development only, disabled in production) |

---

## API Key Management

Nexus generates internal `sk_nexus` keys that proxy to your real provider keys:

```bash
# List all keys
nexus keys list

# Create a new key
nexus keys create --name "cursor-key" --provider openai

# Revoke a key
nexus keys revoke sk_nexus_xxxx

# Rotate a key
nexus keys rotate sk_nexus_xxxx
```

---

## Combo Routing (Fallback Chains)

Combos let you chain providers so that if one fails, the next takes over:

```json
{
  "name": "gpt4-with-fallback",
  "providers": [
    { "provider": "openai", "model": "gpt-4o", "weight": 100 },
    { "provider": "anthropic", "model": "claude-3-5-sonnet", "weight": 0 }
  ],
  "fallbackOnError": true,
  "fallbackOnRateLimit": true
}
```

---

## CLI Reference

```bash
nexus serve               # Start the server
nexus stop                # Stop the server
nexus status              # Show server status
nexus reset-password      # Reset admin password
nexus keys list           # List API keys
nexus keys create         # Create a new key
nexus backup              # Backup the database
nexus restore <file>      # Restore from backup
nexus logs                # View server logs
nexus update              # Update to latest version
nexus version             # Show version info
```

---

## Troubleshooting

### Server won't start

```bash
# Check if port is already in use
netstat -ano | findstr :20128   # Windows
lsof -i :20128                  # Mac/Linux

# Kill the process using the port
npx nexus-ai-gateway stop
```

### Reset admin password

```bash
npx nexus-ai-gateway reset-password
```

### Database corruption

```bash
# Restore from backup
nexus restore ~/.nexus/backups/nexus_backup_latest.db
```

### View logs

```bash
nexus logs --tail 100
```

---

*© 2025 Nexus AI Gateway | Made by Kshitij Singh*
