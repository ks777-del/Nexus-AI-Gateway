# 📡 Nexus AI Gateway — REST API Reference

> Built by **Kshitij Singh** | [GitHub](https://github.com/ks777-del/Nexus-AI-Gateway)
>
> **Base URL**: `http://localhost:20128`  
> **API Base**: `http://localhost:20128/v1`

---

## Table of Contents

1. [Authentication](#authentication)
2. [Response Format](#response-format)
3. [Error Codes](#error-codes)
4. [Chat Completions](#chat-completions)
5. [Models](#models)
6. [Embeddings](#embeddings)
7. [Image Generation](#image-generation)
8. [Audio](#audio)
9. [Files](#files)
10. [Batch API](#batch-api)
11. [Health & Status](#health--status)
12. [Admin API](#admin-api)
13. [Rate Limits & Headers](#rate-limits--headers)

---

## Authentication

All API requests require a Bearer token:

```http
Authorization: Bearer sk_nexus
```

**Token Types:**

| Token | Description |
|---|---|
| `sk_nexus` | Default key — routes to whichever provider you've configured as default |
| `sk_nexus_<id>` | Named key — targets a specific provider |
| `Bearer <your-provider-key>` | Passthrough — your actual provider API key |

---

## Response Format

All responses follow the OpenAI API response format:

```json
{
  "id": "chatcmpl-abc123",
  "object": "chat.completion",
  "created": 1722000000,
  "model": "gpt-4o",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "Hello! How can I help you today?"
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 10,
    "completion_tokens": 12,
    "total_tokens": 22
  }
}
```

---

## Error Codes

| HTTP Status | Code | Description |
|---|---|---|
| `400` | `invalid_request_error` | Malformed request body |
| `401` | `authentication_error` | Invalid or missing API key |
| `403` | `permission_error` | Access denied to this resource |
| `404` | `not_found_error` | Model or endpoint not found |
| `429` | `rate_limit_error` | Rate limit exceeded — Nexus auto-retries with fallback |
| `500` | `server_error` | Internal server error |
| `503` | `service_unavailable` | Provider is down — Nexus will try fallback |

**Error Response Format:**

```json
{
  "error": {
    "message": "The model 'gpt-5' does not exist.",
    "type": "invalid_request_error",
    "code": "model_not_found"
  }
}
```

---

## Chat Completions

### `POST /v1/chat/completions`

Create a chat completion (streaming or non-streaming).

**Request Body:**

```json
{
  "model": "gpt-4o",
  "messages": [
    {
      "role": "system",
      "content": "You are a helpful assistant."
    },
    {
      "role": "user",
      "content": "What is 2 + 2?"
    }
  ],
  "temperature": 0.7,
  "max_tokens": 1000,
  "stream": false,
  "top_p": 1,
  "frequency_penalty": 0,
  "presence_penalty": 0,
  "stop": null,
  "n": 1,
  "tools": [],
  "tool_choice": "auto",
  "response_format": { "type": "text" }
}
```

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `model` | string | ✅ | Model ID. Supports `provider/model` syntax e.g. `anthropic/claude-3-5-sonnet-20241022` |
| `messages` | array | ✅ | Array of message objects |
| `temperature` | float | ❌ | 0–2. Controls randomness. Default: `1` |
| `max_tokens` | integer | ❌ | Max tokens to generate |
| `stream` | boolean | ❌ | Enable SSE streaming. Default: `false` |
| `top_p` | float | ❌ | Nucleus sampling. Default: `1` |
| `n` | integer | ❌ | Number of completions. Default: `1` |
| `stop` | string/array | ❌ | Stop sequences |
| `tools` | array | ❌ | Function/tool definitions |
| `tool_choice` | string/object | ❌ | `"auto"`, `"none"`, or specific function |
| `response_format` | object | ❌ | `{"type": "json_object"}` for JSON mode |
| `seed` | integer | ❌ | Reproducibility seed |

**Message Roles:**

| Role | Description |
|---|---|
| `system` | System prompt — sets model behavior |
| `user` | User message |
| `assistant` | Previous assistant response (for multi-turn) |
| `tool` | Tool/function result |

**Example Response:**

```json
{
  "id": "chatcmpl-xyz789",
  "object": "chat.completion",
  "created": 1722000000,
  "model": "gpt-4o",
  "provider": "openai",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "2 + 2 = 4."
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 20,
    "completion_tokens": 8,
    "total_tokens": 28
  }
}
```

**Streaming Response (SSE):**

```
data: {"id":"chatcmpl-xyz","object":"chat.completion.chunk","choices":[{"delta":{"role":"assistant"},"index":0}]}

data: {"id":"chatcmpl-xyz","object":"chat.completion.chunk","choices":[{"delta":{"content":"2 + 2"},"index":0}]}

data: {"id":"chatcmpl-xyz","object":"chat.completion.chunk","choices":[{"delta":{"content":" = 4."},"index":0}]}

data: [DONE]
```

---

## Models

### `GET /v1/models`

List all available models across all configured providers.

**Response:**

```json
{
  "object": "list",
  "data": [
    {
      "id": "gpt-4o",
      "object": "model",
      "created": 1715000000,
      "owned_by": "openai"
    },
    {
      "id": "claude-3-5-sonnet-20241022",
      "object": "model",
      "created": 1715000000,
      "owned_by": "anthropic"
    },
    {
      "id": "gemini-1.5-pro",
      "object": "model",
      "created": 1715000000,
      "owned_by": "google"
    }
  ]
}
```

### `GET /v1/models/{model_id}`

Get details for a specific model.

```http
GET /v1/models/gpt-4o
```

---

## Embeddings

### `POST /v1/embeddings`

Generate vector embeddings for text.

**Request:**

```json
{
  "model": "text-embedding-3-large",
  "input": "The quick brown fox jumps over the lazy dog",
  "encoding_format": "float",
  "dimensions": 1536
}
```

**Response:**

```json
{
  "object": "list",
  "data": [
    {
      "object": "embedding",
      "index": 0,
      "embedding": [0.0023, -0.0089, 0.0156, ...]
    }
  ],
  "model": "text-embedding-3-large",
  "usage": {
    "prompt_tokens": 9,
    "total_tokens": 9
  }
}
```

---

## Image Generation

### `POST /v1/images/generations`

Generate images from a text prompt.

**Request:**

```json
{
  "model": "dall-e-3",
  "prompt": "A futuristic city skyline at sunset",
  "n": 1,
  "size": "1024x1024",
  "quality": "standard",
  "response_format": "url"
}
```

**Parameters:**

| Parameter | Type | Description |
|---|---|---|
| `model` | string | `dall-e-3`, `dall-e-2` |
| `prompt` | string | Image description |
| `n` | integer | Number of images (1–10 for dall-e-2, 1 for dall-e-3) |
| `size` | string | `256x256`, `512x512`, `1024x1024`, `1792x1024`, `1024x1792` |
| `quality` | string | `standard` or `hd` (dall-e-3 only) |
| `response_format` | string | `url` or `b64_json` |

---

## Audio

### `POST /v1/audio/transcriptions`

Transcribe audio to text (Whisper).

```http
POST /v1/audio/transcriptions
Content-Type: multipart/form-data

file=@audio.mp3
model=whisper-1
language=en
```

### `POST /v1/audio/speech`

Convert text to speech.

```json
{
  "model": "tts-1",
  "input": "Hello, welcome to Nexus AI Gateway!",
  "voice": "nova",
  "speed": 1.0,
  "response_format": "mp3"
}
```

**Available voices:** `alloy`, `echo`, `fable`, `onyx`, `nova`, `shimmer`

---

## Files

### `POST /v1/files`

Upload a file for use with batch or fine-tuning.

```http
POST /v1/files
Content-Type: multipart/form-data

file=@data.jsonl
purpose=batch
```

### `GET /v1/files`

List all uploaded files.

### `GET /v1/files/{file_id}`

Get file metadata.

### `DELETE /v1/files/{file_id}`

Delete a file.

---

## Batch API

### `POST /v1/batches`

Submit a batch job for async processing.

```json
{
  "input_file_id": "file-abc123",
  "endpoint": "/v1/chat/completions",
  "completion_window": "24h",
  "metadata": {
    "description": "My batch job"
  }
}
```

### `GET /v1/batches/{batch_id}`

Check batch status.

### `GET /v1/batches`

List all batch jobs.

---

## Health & Status

### `GET /health`

Server health check.

```json
{
  "status": "ok",
  "version": "3.8.48",
  "uptime": 3600,
  "timestamp": "2025-08-02T08:00:00Z"
}
```

### `GET /v1/health`

Detailed health with provider status.

```json
{
  "status": "healthy",
  "providers": {
    "openai": "healthy",
    "anthropic": "healthy",
    "google": "degraded"
  },
  "latency_ms": 245
}
```

---

## Admin API

These endpoints require admin authentication.

### `GET /api/admin/keys`

List all API keys.

### `POST /api/admin/keys`

Create a new API key.

```json
{
  "name": "cursor-integration",
  "provider": "openai",
  "expiresAt": "2026-01-01T00:00:00Z"
}
```

### `DELETE /api/admin/keys/{key_id}`

Revoke an API key.

### `GET /api/admin/usage`

Get usage statistics.

### `GET /api/admin/logs`

Get request logs (paginated).

```
GET /api/admin/logs?page=1&limit=50&provider=openai&from=2025-08-01
```

---

## Rate Limits & Headers

### Request Headers

```http
Authorization: Bearer sk_nexus          # Required
Content-Type: application/json          # Required for POST requests
X-Nexus-Provider: openai               # Optional: force specific provider
X-Nexus-Fallback: true                 # Optional: enable combo fallback
X-Request-ID: my-request-id            # Optional: for tracing
```

### Response Headers

```http
X-Nexus-Provider: openai               # Which provider handled the request
X-Nexus-Model: gpt-4o                  # Actual model used
X-Nexus-Latency: 432                   # Provider latency in ms
X-Nexus-Tokens-Input: 20               # Input tokens used
X-Nexus-Tokens-Output: 150             # Output tokens generated
X-Request-ID: req_abc123               # Request ID for tracing
X-RateLimit-Limit: 1000                # Requests per minute limit
X-RateLimit-Remaining: 997             # Remaining requests
X-RateLimit-Reset: 1722000060          # Reset timestamp
```

---

## Nexus-Specific Extensions

Nexus adds extra metadata to responses:

```json
{
  "id": "chatcmpl-xyz",
  "model": "gpt-4o",
  "nexus": {
    "provider": "openai",
    "fallback_used": false,
    "combo": null,
    "latency_ms": 432,
    "cached": false,
    "compressed": true,
    "compression_ratio": 0.82
  }
}
```

---

*© 2025 Nexus AI Gateway | Made by Kshitij Singh*
