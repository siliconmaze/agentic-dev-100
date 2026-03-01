# Lab 1: Your First Agent with OpenClaw + Ollama

## Learning Objectives
- Set up OpenClaw framework with Ollama for local LLM execution
- Create and run your first simple agent
- Understand agent lifecycle and configuration
- Learn to debug common setup issues

## Prerequisites
- Python 3.8+ installed
- Basic Python knowledge
- Command line/terminal familiarity
- Git installed (optional)

## Why Ollama?
Ollama provides free, local LLM execution without API costs or internet dependency. All labs in this course use Ollama exclusively.

---

## Detailed Worked Steps

### Step 1: Install Ollama (DO THIS FIRST)

**macOS/Linux:**
```bash
# Open Terminal and run:
curl -fsSL https://ollama.ai/install.sh | sh
```

**Windows:**
Download from https://ollama.ai/download/windows

**Verify installation:**
```bash
ollama --version
```

### Step 2: Pull Your First Model

Ollama needs to download a model before use. We'll use `llama3.2:3b` (a small, fast model):

```bash
ollama pull llama3.2:3b
```

This downloads ~2GB. Wait for completion.

**Verify models:**
```bash
ollama list
```

You should see `llama3.2:3b` in the list.

### Step 3: Test Ollama Works

```bash
ollama run llama3.2:3b "Hello"
```

Type `/bye` to exit.

### Step 4: Set Up Python Environment

```bash
# Create project folder
mkdir first-agent-lab
cd first-agent-lab

# Create virtual environment (isolates your Python packages)
python -m venv venv

# Activate it
# Linux/Mac:
source venv/bin/activate

# Windows (Command Prompt):
venv\Scripts\activate.bat

# Windows (PowerShell):
venv\Scripts\Activate.ps1
```

**You'll know it's activated** when you see `(venv)` at the start of your terminal prompt.

### Step 5: Install OpenClaw

```bash
pip install openclaw
```

**Verify installation:**
```bash
pip show openclaw
```

---

## Your First Agent (Code-Along)

### Step 6: Create the Agent File

Create a new file called `first_agent.py`:

```python
# first_agent.py

# Step 1: Import OpenClaw components
from openclaw import Agent, Runner
from openclaw.llms.ollama import OllamaLLM

# Step 2: Configure Ollama connection
llm = OllamaLLM(
    model="llama3.2:3b",        # The model we pulled earlier
    base_url="http://localhost:11434",  # Default Ollama port
    temperature=0.7             # creativity level (0-1)
)

# Step 3: Define your agent class
class GreetingAgent(Agent):
    """A friendly greeting agent that responds warmly."""
    
    def __init__(self):
        super().__init__(
            name="Greeter",
            description="A friendly greeting agent",
            system_prompt="""You are a helpful assistant that greets users warmly.
            Always be polite and enthusiastic. Keep responses short and friendly.""",
            llm=llm
        )
    
    def run(self, user_input: str) -> str:
        """Process user input and return agent response."""
        response = self.llm.generate(
            prompt=user_input,
            max_tokens=150
        )
        return response

# Step 4: Run the agent
if __name__ == "__main__":
    # Create agent instance
    agent = GreetingAgent()
    
    # Create runner (handles the execution)
    runner = Runner(agent)
    
    # Test with sample messages
    test_messages = ["Hello!", "What's your name?", "What can you help me with?"]
    
    for msg in test_messages:
        print(f"\n👤 User: {msg}")
        response = runner.run(msg)
        print(f"🤖 Agent: {response}")
```

### Step 7: Run Your Agent

```bash
python first_agent.py
```

**Expected output:**
```
👤 User: Hello!
🤖 Agent: Hello there! How can I help you today?

👤 User: What's your name?
🤖 Agent: I'm Greeter, your friendly assistant!

👤 User: What can you help me with?
🤖 Agent: I can help with answering questions, having conversations...
```

---

## Adding Tools (Advanced)

### Step 8: Create Agent with Tool

Tools give your agent abilities beyond just text generation.

Create `agent_with_tool.py`:

```python
# agent_with_tool.py

from openclaw import Agent, Runner, Tool
from openclaw.llms.ollama import OllamaLLM
import datetime

# Step 1: Define a tool using the @Tool decorator
@Tool
def get_current_time() -> str:
    """Returns the current date and time in human-readable format."""
    return datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")

# Step 2: Create agent WITH the tool
class TimeAwareAgent(Agent):
    """Agent that knows and reports the current time."""
    
    def __init__(self):
        llm = OllamaLLM(model="llama3.2:3b")
        super().__init__(
            name="TimeAssistant",
            description="Agent that knows the current time",
            system_prompt="""You have access to the current time.
            When asked about time or date, use the get_current_time tool.""",
            llm=llm,
            tools=[get_current_time]  # Pass tool here
        )

# Step 3: Run and test
if __name__ == "__main__":
    agent = TimeAwareAgent()
    runner = Runner(agent)
    
    response = runner.run("What time is it right now?")
    print(f"Response: {response}")
```

### Step 9: Run Tool Agent

```bash
python agent_with_tool.py
```

---

## Exercises (Try These)

### Exercise 1: Basic (5 min)
Modify the greeting agent to ask for the user's name and remember it.

**Hint:** Store the name in `self.name` in the agent class.

### Exercise 2: Intermediate (15 min)
Add a calculator tool that:
- Takes two numbers and an operation (+, -, *, /)
- Returns the result
- Handles division by zero

### Exercise 3: Advanced (30 min)
Create an agent that can summarize text passed to it.

**Hint:** Use the LLM's generate method with a prompt like "Summarize this: {text}"

---

## Troubleshooting

### Problem: "Ollama not running"
**Solution:** Start Ollama with `ollama serve` in a separate terminal.

### Problem: "Model not found"
**Solution:** Run `ollama pull llama3.2:3b` to download the model.

### Problem: "Connection refused"
**Solution:** 
1. Check Ollama is running: `ollama list`
2. Verify port 11434 is accessible
3. Try `ollama serve` in a new terminal window

### Problem: "Module not found: openclaw"
**Solution:**
1. Verify virtual environment is activated (look for `venv` in prompt)
2. Run `pip install openclaw` again
3. Check with `pip show openclaw`

---

## What You Learned

✅ How to install and configure Ollama
✅ How to create a virtual Python environment
✅ How to install and use OpenClaw
✅ How to build your first agent
✅ How to add tools to an agent
✅ How to test and debug agent issues

---

## Next Steps

- **Lab 2:** Learn advanced tool integration patterns
- **Lab 3:** Explore memory and context management
- **Lab 4:** Build ReAct agents with structured reasoning
