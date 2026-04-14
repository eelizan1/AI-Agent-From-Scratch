# AI Tool-Calling Agent (Hugging Face)

A minimal implementation of a **tool-using AI agent** built with Hugging Face LLMs.
This project demonstrates how to move from simple prompt-based LLM usage to a **multi-step agent loop** capable of reasoning, calling tools, and synthesizing results.

---

## Overview

Traditional LLM usage is single-step:

```
User → LLM → Response
```

This project extends that into an **agent workflow**:

```
User → Agent → LLM → Tool → LLM → Response
```

The agent:

* Maintains conversation state
* Allows the LLM to decide when to call tools
* Executes Python functions as tools
* Feeds results back into the LLM
* Repeats until a final answer is produced

---

## Features

* ✅ Tool-calling LLM via Hugging Face (`InferenceClient`)
* ✅ Custom agent loop (no external frameworks)
* ✅ JSON-based tool schemas
* ✅ Dynamic tool execution
* ✅ Multi-step reasoning loop
* ✅ Conversation memory tracking

---

## Project Structure

```
.
├── notebook.ipynb   # Full demo (Colab-ready)
├── agent.py         # Agent implementation (optional extraction)
└── README.md
```

---

## Setup

### 1. Install dependencies

```bash
pip install huggingface_hub pydantic
```

### 2. Set your Hugging Face token

```python
import os
import getpass

os.environ["HF_TOKEN"] = getpass.getpass("Hugging Face Token: ")
```

---

## Usage

### 1. Initialize LLM

```python
from huggingface_hub import InferenceClient

client = InferenceClient(
    token=os.environ["HF_TOKEN"],
    model="moonshotai/Kimi-K2-Thinking"
)
```

---

### 2. Define a Tool

```python
def get_temperature(city: str):
    if city.lower() == "san francisco":
        return "72°F"
    return "70°F"
```

---

### 3. Define Tool Schema

```python
schema = {
    "type": "function",
    "function": {
        "name": "get_temperature",
        "description": "Get the current temperature in a given city.",
        "parameters": {
            "type": "object",
            "properties": {
                "city": {"type": "string"}
            },
            "required": ["city"]
        }
    }
}
```

---

### 4. Create Agent

```python
agent = Agent(
    client=client,
    system="You are a helpful assistant that can use tools.",
    tools=[schema],
    tool_registry={
        "get_temperature": get_temperature
    }
)
```

---

### 5. Run Agent

```python
response = agent("What is the temperature in San Francisco?")
print(response)
```

Example output:

```
Executing tool: get_temperature with args {'city': 'San Francisco'} -> 72°F
The temperature in San Francisco is 72°F.
```

---

## How It Works

1. User sends a message
2. Agent forwards context + tools to LLM
3. LLM decides:

   * respond directly OR
   * call a tool
4. If tool is called:

   * agent executes Python function
   * result is appended to conversation
5. LLM processes tool output
6. Final response is returned

---

## Key Concept

> The LLM does not execute tools — it only **decides** which tool to call.
> The agent is responsible for execution and control flow.

---

## Example Agent Loop

```python
while not done:
    response = llm(messages, tools)

    if response.has_tool_call:
        result = execute_tool(response.tool_call)
        messages.append(result)
    else:
        return response
```

---

## Limitations

* No tool sandboxing (unsafe in production)
* No retry or fallback strategies
* No structured output validation
* Relies on LLM correctness for tool selection

---

## Future Improvements

* Tool registry abstraction
* Error handling + retries
* Observability (logging, tracing)
* Integration with APIs (logs, metrics, Jira, etc.)
* Move from notebook → backend service

---

## Real-World Applications

This pattern can be extended into:

* 🔍 On-call debugging assistants (logs + metrics)
* 📊 Data query agents (SQL, APIs)
* 🧠 Internal knowledge assistants (RAG + tools)
* ⚙️ Workflow automation (Slack, Jira, CI/CD)

---

## References

* Hugging Face Inference API
* Tool calling / function calling patterns
* ReAct-style agents

---

## License

MIT
