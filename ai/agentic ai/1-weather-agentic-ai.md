# AI Agent Development Notes 🤖

## Problem: LLMs Can't Access Real-Time Data
- LLMs work on **pre-training data** only
- Can't provide current weather, stock prices, etc.
- Need external APIs for real-time info

## Solution: Function Calling (Tools)

### Basic Setup
```python
import requests
from openai import OpenAI

def get_weather(city: str):
    url = f"http://api.weatherapi.com/v1/current.json?q={city.lower()}"
    response = requests.get(url)
    if response.status_code == 200:
        return f"Weather in {city}: {response.text}"
    return "Error occurred"
```

---

## Building the Agent

### 1. Chain of Thought Prompt
Add to system prompt:
- "You can call tools from available tools list"
- Define tool: `get_weather` - takes city name, returns weather

### 2. Example Format (Teach LLM)
```
User: "What's the weather in Delhi?"

Step: plan
Content: "Need to call get_weather for Delhi"

Step: tool
Tool: get_weather
Input: Delhi

Step: observe
Tool: get_weather
Output: "Temperature 27°C, cloudy"

Step: output
Content: "Current weather in Delhi is 27°C with cloudy skies"
```

---

## Implementation Steps

### 3. Tool Execution Logic
```python
available_tools = {
    "get_weather": get_weather
}

if parsed_result.step == "tool":
    tool_name = parsed_result.tool
    tool_input = parsed_result.input
    
    # Call the tool
    tool_response = available_tools[tool_name](tool_input)
    
    # Add observation to message history
    message_history.append({
        "role": "developer",
        "content": {
            "step": "observe",
            "tool": tool_name,
            "input": tool_input,
            "output": tool_response
        }
    })
```

### 4. Continuous Loop
```python
while True:
    user_input = input("Ask: ")
    message_history.append({"role": "user", "content": user_input})
    
    # Make LLM call
    result = client.chat.completions.parse(...)
    
    if result.step == "tool":
        # Execute tool logic
    elif result.step == "output":
        print(result.content)
```

---

## Structured Outputs (Better Approach) ✨

### Problem with Basic Approach
- LLM returns plain strings
- Parsing JSON can fail
- No type safety

### Solution: Pydantic
```python
from pydantic import BaseModel, Field
from typing import Optional

class MyOutputFormat(BaseModel):
    step: str = Field(description="ID of step: plan, tool, output")
    content: Optional[str] = Field(None, description="Content for step")
    tool: Optional[str] = Field(None, description="Tool to call")
    input: Optional[str] = Field(None, description="Input for tool")
```

### Usage
```python
# Instead of completions.create()
response = client.chat.completions.parse(
    model="gpt-4",
    messages=messages,
    response_format=MyOutputFormat  # Pass the class
)

# Get parsed result directly
parsed = response.parsed  # Already a Python object!
print(parsed.step)      # Type-safe access
print(parsed.content)
```

---

## Key Takeaways 💡

1. **Agent = LLM + Tools**
2. **Chain of Thought** teaches LLM when to use tools
3. **Observe step** feeds tool results back to LLM
4. **Structured outputs** = less errors, type safety
5. Can handle **multiple tool calls** in one query

---