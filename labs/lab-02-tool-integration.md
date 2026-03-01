# Lab 2: Tool Integration Basics

## Learning Objectives
- Create custom tools with proper typing and documentation
- Integrate external APIs as tools
- Handle tool execution errors gracefully
- Chain multiple tools in agent workflows
- Understand tool selection mechanisms

## Prerequisites
- Completed Lab 1
- Basic understanding of REST APIs
- Python decorators knowledge
- OpenClaw installation working

## Why Tools?
Tools extend your agent's capabilities beyond text generation. They allow agents to:
- Perform calculations
- Access external data
- Execute code
- Interact with APIs

---

## Detailed Worked Steps

### Step 1: Set Up Lab 2 Environment

```bash
# Go back to your main folder
cd ~/first-agent-lab  # or wherever you created Lab 1

# Create new directory for Lab 2
mkdir lab2-tools
cd lab2-tools

# Activate your virtual environment
source ../venv/bin/activate  # adjust path as needed

# Install requests (for API tools)
pip install requests
```

### Step 2: Understanding Tool Structure

Tools are Python functions decorated with `@Tool`. Each tool needs:
1. **Type hints** for parameters and return values
2. **Clear docstrings** explaining what it does
3. **Error handling** for robustness
4. **JSON-serializable return values**

**Example skeleton:**
```python
from openclaw import Tool
from typing import Dict, Any

@Tool
def my_tool(param: str) -> Dict[str, Any]:
    """
    Description of what the tool does.
    
    Args:
        param: Description of parameter
    
    Returns:
        Dictionary with results
    """
    # Tool logic here
    return {"result": "value"}
```

---

## Creating Mathematical Tools

### Step 3: Create Math Tools File

Create `math_tools.py`:

```python
from openclaw import Tool
from typing import Dict, List, Any
import math

@Tool
def calculate_basic(expression: str) -> Dict[str, Any]:
    """
    Calculate basic mathematical expressions safely.
    
    Only allows specific math functions to prevent code execution.
    
    Args:
        expression: Math expression like "2 + 3 * 4" or "sqrt(16)"
    
    Returns:
        Dictionary with success status and result
    """
    # Define ALLOWED functions only - this is security!
    allowed = {
        "sqrt": math.sqrt,
        "sin": math.sin,
        "cos": math.cos,
        "tan": math.tan,
        "log": math.log,
        "log10": math.log10,
        "pi": math.pi,
        "e": math.e,
        # Add more safe math functions as needed
    }
    
    try:
        # eval with restricted globals for safety
        result = eval(expression, {"__builtins__": None}, allowed)
        return {
            "success": True,
            "result": result,
            "expression": expression
        }
    except Exception as e:
        return {
            "success": False,
            "error": str(e),
            "suggestion": "Use format like '2 + 3 * 4' or 'sqrt(25)'"
        }

@Tool
def convert_units(value: float, from_unit: str, to_unit: str) -> Dict[str, Any]:
    """
    Convert between common units (length, weight, temperature).
    
    Args:
        value: Numeric value to convert
        from_unit: Source unit (km, m, kg, lb, c, f)
        to_unit: Target unit
    
    Returns:
        Dictionary with conversion result
    """
    conversions = {
        "length": {
            "km": {"m": 1000, "mile": 0.621371, "cm": 100000},
            "m": {"km": 0.001, "cm": 100, "mile": 0.000621371},
            "mile": {"km": 1.60934, "m": 1609.34}
        },
        "weight": {
            "kg": {"lb": 2.20462, "g": 1000},
            "lb": {"kg": 0.453592, "g": 453.592},
            "g": {"kg": 0.001, "lb": 0.00220462}
        },
        "temperature": {
            "c": {"f": lambda x: x * 9/5 + 32},
            "f": {"c": lambda x: (x - 32) * 5/9}
        }
    }
    
    try:
        for category, units in conversions.items():
            if from_unit in units and to_unit in units[from_unit]:
                conversion = units[from_unit][to_unit]
                
                if callable(conversion):
                    result = conversion(value)
                else:
                    result = value * conversion
                
                return {
                    "success": True,
                    "original": f"{value} {from_unit}",
                    "converted": f"{result:.4f} {to_unit}",
                    "rate": conversion if not callable(conversion) else "formula"
                }
        
        return {
            "success": False,
            "error": f"Cannot convert from {from_unit} to {to_unit}",
            "supported": "km-m, kg-lb, c-f, etc."
        }
    except Exception as e:
        return {
            "success": False,
            "error": str(e)
        }
```

**Test your math tools:**
```bash
python -c "
from math_tools import calculate_basic, convert_units

# Test calculation
result = calculate_basic('sqrt(16) + 2')
print(result)

# Test conversion  
result = convert_units(5, 'km', 'mile')
print(result)
"
```

---

## Creating API Integration Tools

### Step 4: Create API Tools File

Create `api_tools.py`:

```python
from openclaw import Tool
import requests
from typing import Dict, Any, Optional

@Tool
def get_weather(city: str, country_code: Optional[str] = None) -> Dict[str, Any]:
    """
    Get weather information for a city.
    
    Uses mock data for reliability. Replace with real API for production.
    
    Args:
        city: City name (e.g., "London")
        country_code: Optional country code (e.g., "UK", "US")
    
    Returns:
        Dictionary with weather data
    """
    location = f"{city},{country_code}" if country_code else city
    
    # Mock data - easy to replace with real API later
    mock_data = {
        "London,UK": {"temp": 15, "condition": "Cloudy", "humidity": 78},
        "New York,US": {"temp": 22, "condition": "Sunny", "humidity": 65},
        "Tokyo,JP": {"temp": 18, "condition": "Rainy", "humidity": 85},
        "Paris,FR": {"temp": 17, "condition": "Partly Cloudy", "humidity": 72},
        "default": {"temp": 20, "condition": "Clear", "humidity": 70}
    }
    
    weather = mock_data.get(location, mock_data["default"])
    
    return {
        "location": location,
        "temperature": f"{weather['temp']}°C",
        "conditions": weather["condition"],
        "humidity": f"{weather['humidity']}%",
        "note": "Mock data - replace with real API"
    }

@Tool
def fetch_joke(category: str = "programming") -> Dict[str, Any]:
    """
    Fetch a random joke from jokeapi.dev.
    
    Args:
        category: Joke category (programming, general, knock-knock, etc.)
    
    Returns:
        Dictionary with joke
    """
    try:
        url = f"https://v2.jokeapi.dev/joke/{category}"
        params = {"type": "single", "amount": 1}
        
        response = requests.get(url, params=params, timeout=5)
        data = response.json()
        
        if data.get("error", False):
            return {
                "success": False,
                "error": data.get("message", "Unknown error")
            }
        
        return {
            "success": True,
            "category": data.get("category", "Unknown"),
            "joke": data.get("joke", ""),
            "flags": data.get("flags", {})
        }
    except requests.RequestException as e:
        return {
            "success": False,
            "error": f"API request failed: {str(e)}",
            "fallback_joke": "Why do programmers prefer dark mode? Because light attracts bugs!"
        }
```

**Test your API tools:**
```bash
python -c "
from api_tools import get_weather, fetch_joke

# Test weather
weather = get_weather('London', 'UK')
print(weather)

# Test joke
joke = fetch_joke('programming')
print(joke)
"
```

---

## Building Multi-Tool Agents

### Step 5: Create Agent with Multiple Tools

Create `multi_tool_agent.py`:

```python
from openclaw import Agent, Runner
from openclaw.llms.ollama import OllamaLLM
from math_tools import calculate_basic, convert_units
from api_tools import get_weather, fetch_joke

class ToolMasterAgent(Agent):
    """
    Agent with access to multiple tools for various tasks.
    
    Tools:
    - calculate_basic: Math expressions
    - convert_units: Unit conversions  
    - get_weather: Weather information
    - fetch_joke: Random jokes
    """
    
    def __init__(self):
        llm = OllamaLLM(
            model="llama3.2:3b",
            temperature=0.3  # Lower temperature for tool use
        )
        
        super().__init__(
            name="ToolMaster",
            description="Agent with multiple integrated tools",
            system_prompt="""You are a helpful assistant with access to various tools.
            
            AVAILABLE TOOLS:
            1. calculate_basic - for math expressions like "2 + 3 * 4"
            2. convert_units - for unit conversions like "5 km to miles"
            3. get_weather - for weather info like "weather in London"
            4. fetch_joke - for jokes, specify category if wanted
            
            INSTRUCTIONS:
            - When user asks a question, determine which tool(s) to use
            - Always explain what tool you're using
            - If a tool fails, explain the error to the user""",
            llm=llm,
            tools=[calculate_basic, convert_units, get_weather, fetch_joke]
        )

# Run the agent
if __name__ == "__main__":
    agent = ToolMasterAgent()
    runner = Runner(agent)
    
    test_queries = [
        "What's 15 * 3 + 8?",
        "Convert 5 kilometers to miles",
        "What's the weather like in Tokyo?",
        "Tell me a programming joke",
        "How many pounds in 10 kilograms?"
    ]
    
    print("Testing Multi-Tool Agent")
    print("=" * 50)
    
    for query in test_queries:
        print(f"\nQuery: {query}")
        response = runner.run(query)
        print(f"Response: {response}")
        print("-" * 50)
```

### Step 6: Run and Observe

```bash
python multi_tool_agent.py
```

**Watch for:**
- Agent selecting the correct tool
- Tool executing and returning results
- Agent using tool results in response

---

## Error Handling Best Practices

### Step 7: Safe File Operation Tool

Create `error_handling.py`:

```python
from openclaw import Tool
from typing import Dict, Any

@Tool
def safe_file_operation(filename: str, operation: str = "read") -> Dict[str, Any]:
    """
    Safely perform file operations with error handling.
    
    Args:
        filename: Name of file to operate on (no paths allowed)
        operation: 'read', 'write', or 'info'
    
    Returns:
        Dictionary with operation result or error
    """
    import os
    
    # SECURITY: Prevent directory traversal attacks
    if ".." in filename or "/" in filename or "\\" in filename:
        return {
            "success": False,
            "error": "Invalid filename",
            "security_note": "Directory traversal not allowed"
        }
    
    try:
        if operation == "read":
            if not os.path.exists(filename):
                return {"success": False, "error": f"File not found: {filename}"}
            
            with open(filename, 'r') as f:
                content = f.read()
            
            return {"success": True, "content": content[:500], "size": len(content)}
            
        elif operation == "info":
            if not os.path.exists(filename):
                return {"success": False, "error": "File does not exist"}
            
            stats = os.stat(filename)
            return {
                "success": True,
                "filename": filename,
                "size": stats.st_size,
                "created": stats.st_ctime,
                "modified": stats.st_mtime
            }
            
        else:
            return {
                "success": False,
                "error": f"Unknown operation: {operation}",
                "supported": ["read", "info"]
            }
            
    except PermissionError:
        return {"success": False, "error": "Permission denied"}
    except Exception as e:
        return {"success": False, "error": f"Unexpected error: {str(e)}"}
```

---

## Exercises

### Exercise 1: Basic (10 min)
Create a string tool that:
- Reverses a string
- Counts vowels
- Returns both results

### Exercise 2: Intermediate (20 min)
Build a stock price checker tool:
- Use mock data for 5 stocks
- Return current price and change %
- Handle unknown stocks gracefully

### Exercise 3: Advanced (30 min)
Create a tool that chains other tools:
1. Get weather for a city
2. Convert temperature from C to F
3. Return combined result

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Tool not recognized | Pass function references, not strings: `tools=[my_tool]` |
| Type errors | Check function signatures match docstring |
| API timeouts | Add timeout parameter: `requests.get(url, timeout=5)` |
| Permission errors | Run with appropriate file permissions |

---

## What You Learned

✅ How to create tools with @Tool decorator
✅ How to add type hints to tools
✅ How to handle errors gracefully
✅ How to integrate external APIs
✅ How to build multi-tool agents
✅ How to chain tools together

---

## Next Steps

- **Lab 3:** Learn memory and context management
- **Lab 4:** Explore ReAct pattern for structured reasoning
