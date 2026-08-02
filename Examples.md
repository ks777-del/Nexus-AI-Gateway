# 💡 Nexus AI Gateway — Examples

> Built by **Kshitij Singh** | [GitHub](https://github.com/ks777-del/Nexus-AI-Gateway)

All examples assume your server is running at `http://localhost:20128`.

---

## Table of Contents

1. [Quick Start](#quick-start)
2. [cURL Examples](#curl-examples)
3. [Python Examples](#python-examples)
4. [JavaScript / Node.js Examples](#javascript--nodejs-examples)
5. [Streaming](#streaming)
6. [Multi-turn Chat](#multi-turn-chat)
7. [Image Generation](#image-generation)
8. [Tool Calling (Function Calling)](#tool-calling-function-calling)
9. [Combo Routing Example](#combo-routing-example)
10. [Configuring AI Tools](#configuring-ai-tools)

---

## Quick Start

Nexus AI Gateway is **OpenAI-compatible**. Point any OpenAI SDK to `http://localhost:20128/v1` and it just works.

```bash
# Test the server is running
curl http://localhost:20128/v1/models
```

---

## cURL Examples

### Basic Chat Completion

```bash
curl http://localhost:20128/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk_nexus" \
  -d '{
    "model": "gpt-4o",
    "messages": [
      {"role": "user", "content": "Hello! What can you do?"}
    ]
  }'
```

### Streaming Chat

```bash
curl http://localhost:20128/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk_nexus" \
  -d '{
    "model": "claude-3-5-sonnet-20241022",
    "stream": true,
    "messages": [
      {"role": "user", "content": "Write a short poem about coding."}
    ]
  }'
```

### List Available Models

```bash
curl http://localhost:20128/v1/models \
  -H "Authorization: Bearer sk_nexus"
```

### Using a Specific Provider

```bash
# Prefix the model with the provider name
curl http://localhost:20128/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk_nexus" \
  -d '{
    "model": "anthropic/claude-3-5-haiku-20241022",
    "messages": [
      {"role": "user", "content": "Explain quantum computing simply."}
    ]
  }'
```

---

## Python Examples

### Installation

```bash
pip install openai
```

### Basic Chat

```python
from openai import OpenAI

client = OpenAI(
    api_key="sk_nexus",  # Your Nexus API key
    base_url="http://localhost:20128/v1",
)

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "What is the capital of France?"},
    ],
)

print(response.choices[0].message.content)
```

### Streaming

```python
from openai import OpenAI

client = OpenAI(
    api_key="sk_nexus",
    base_url="http://localhost:20128/v1",
)

stream = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Write a haiku about AI."}],
    stream=True,
)

for chunk in stream:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="", flush=True)
print()
```

### Using Anthropic via Nexus

```python
from openai import OpenAI

client = OpenAI(
    api_key="sk_nexus",
    base_url="http://localhost:20128/v1",
)

response = client.chat.completions.create(
    model="anthropic/claude-3-5-sonnet-20241022",
    messages=[
        {"role": "user", "content": "Summarize the benefits of TypeScript."}
    ],
    max_tokens=500,
)

print(response.choices[0].message.content)
```

### Listing Available Models

```python
from openai import OpenAI

client = OpenAI(
    api_key="sk_nexus",
    base_url="http://localhost:20128/v1",
)

models = client.models.list()
for model in models.data:
    print(f"- {model.id}")
```

---

## JavaScript / Node.js Examples

### Installation

```bash
npm install openai
```

### Basic Chat

```javascript
import OpenAI from 'openai';

const client = new OpenAI({
  apiKey: 'sk_nexus',
  baseURL: 'http://localhost:20128/v1',
});

const response = await client.chat.completions.create({
  model: 'gpt-4o',
  messages: [
    { role: 'user', content: 'Explain REST APIs in one paragraph.' }
  ],
});

console.log(response.choices[0].message.content);
```

### With Error Handling

```javascript
import OpenAI from 'openai';

const client = new OpenAI({
  apiKey: 'sk_nexus',
  baseURL: 'http://localhost:20128/v1',
});

async function chat(prompt) {
  try {
    const response = await client.chat.completions.create({
      model: 'gpt-4o-mini',
      messages: [{ role: 'user', content: prompt }],
      temperature: 0.7,
      max_tokens: 1000,
    });
    return response.choices[0].message.content;
  } catch (error) {
    if (error.status === 429) {
      console.error('Rate limited — Nexus will auto-fallback if combos configured');
    }
    throw error;
  }
}

console.log(await chat('What is machine learning?'));
```

---

## Streaming

### Python Streaming with Callbacks

```python
from openai import OpenAI

client = OpenAI(api_key="sk_nexus", base_url="http://localhost:20128/v1")

def stream_response(prompt: str, model: str = "gpt-4o"):
    full_response = ""
    with client.chat.completions.stream(
        model=model,
        messages=[{"role": "user", "content": prompt}],
    ) as stream:
        for text in stream.text_stream:
            print(text, end="", flush=True)
            full_response += text
    print()
    return full_response

result = stream_response("Explain neural networks step by step.")
```

### JavaScript SSE Streaming

```javascript
import OpenAI from 'openai';

const client = new OpenAI({
  apiKey: 'sk_nexus',
  baseURL: 'http://localhost:20128/v1',
});

const stream = await client.chat.completions.stream({
  model: 'gpt-4o',
  messages: [{ role: 'user', content: 'Count from 1 to 10 slowly.' }],
});

for await (const chunk of stream) {
  const text = chunk.choices[0]?.delta?.content ?? '';
  process.stdout.write(text);
}
console.log('\nDone!');
```

---

## Multi-turn Chat

```python
from openai import OpenAI

client = OpenAI(api_key="sk_nexus", base_url="http://localhost:20128/v1")

conversation_history = [
    {"role": "system", "content": "You are a helpful coding assistant."}
]

def chat(user_message: str) -> str:
    conversation_history.append({"role": "user", "content": user_message})
    
    response = client.chat.completions.create(
        model="gpt-4o",
        messages=conversation_history,
    )
    
    assistant_message = response.choices[0].message.content
    conversation_history.append({"role": "assistant", "content": assistant_message})
    return assistant_message

# Multi-turn conversation
print(chat("What is Python?"))
print(chat("What are its main use cases?"))
print(chat("How does it compare to JavaScript?"))
```

---

## Image Generation

```python
from openai import OpenAI

client = OpenAI(api_key="sk_nexus", base_url="http://localhost:20128/v1")

response = client.images.generate(
    model="dall-e-3",
    prompt="A futuristic AI gateway server room with neon lights",
    n=1,
    size="1024x1024",
)

print(f"Image URL: {response.data[0].url}")
```

---

## Tool Calling (Function Calling)

```python
from openai import OpenAI
import json

client = OpenAI(api_key="sk_nexus", base_url="http://localhost:20128/v1")

tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get the current weather for a city",
            "parameters": {
                "type": "object",
                "properties": {
                    "city": {
                        "type": "string",
                        "description": "City name, e.g. 'New Delhi'"
                    }
                },
                "required": ["city"]
            }
        }
    }
]

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "What's the weather in New Delhi?"}],
    tools=tools,
    tool_choice="auto",
)

tool_call = response.choices[0].message.tool_calls[0]
args = json.loads(tool_call.function.arguments)
print(f"Called: {tool_call.function.name}({args})")
```

---

## Combo Routing Example

Combos let you define fallback chains. Create one in the dashboard, then use it:

```python
from openai import OpenAI

client = OpenAI(api_key="sk_nexus", base_url="http://localhost:20128/v1")

# Use a combo by its name (set up in Dashboard → Combos)
response = client.chat.completions.create(
    model="combo:gpt4-with-fallback",  # Your combo name
    messages=[{"role": "user", "content": "Hello!"}],
)

print(response.choices[0].message.content)
# Nexus will try GPT-4o first, then Claude if it fails
```

---

## Configuring AI Tools

### Cursor

1. Open Cursor Settings → Models
2. Add OpenAI-compatible provider:
   - **Base URL**: `http://localhost:20128/v1`
   - **API Key**: `sk_nexus`

### Cline (VS Code Extension)

1. Open Cline settings in VS Code
2. Set **API Provider** to `OpenAI Compatible`
3. Set **Base URL** to `http://localhost:20128/v1`
4. Set **API Key** to `sk_nexus`

### Continue.dev

```json
// ~/.continue/config.json
{
  "models": [
    {
      "title": "Nexus AI Gateway",
      "provider": "openai",
      "model": "gpt-4o",
      "apiBase": "http://localhost:20128/v1",
      "apiKey": "sk_nexus"
    }
  ]
}
```

### OpenAI Codex CLI

```bash
export OPENAI_API_KEY="sk_nexus"
export OPENAI_BASE_URL="http://localhost:20128/v1"
codex "Explain this code"
```

---

*© 2025 Nexus AI Gateway | Made by Kshitij Singh*
