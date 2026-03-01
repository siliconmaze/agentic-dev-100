# Lab 4: ReAct Agents

## Learning Objectives
- Implement ReAct (Reasoning + Acting) framework
- Create thought-action-observation loops
- Build agents that reason before acting
- Handle complex multi-step problems
- Debug ReAct execution flows

## Prerequisites
- Labs 1-3 completed
- Understanding of tool integration
- Knowledge of control flow patterns

## What is ReAct?
ReAct (Reasoning + Acting) is a pattern where the agent:
1. **Thinks** about what to do
2. **Acts** by calling a tool
3. **Observes** the result
4. **Repeats** until solved

This mirrors how humans solve problems - think, try, observe, adjust.

---

## Detailed Worked Steps

### Step 1: Set Up Lab 4 Environment

```bash
# Create new directory
mkdir lab4-react
cd lab4-react

# Activate virtual environment
source ../venv/bin/activate

# Install pydantic if needed
pip install pydantic
```

---

## Understanding ReAct Pattern

### Step 2: ReAct Loop Visualization

ReAct follows this structured loop:

```
Question: What is sqrt(144) + 25?

Thought: I need to calculate this. First, I'll compute sqrt(144).
Action: calculator
Action Input: sqrt(144)
Observation: 12

Thought: Now I need to add 25 to 12.
Action: calculator
Action Input: 12 + 25
Observation: 37

Thought: I've computed the answer.
Final Answer: 37
```

Each iteration:
- **Thought**: Reasoning about the current state
- **Action**: Which tool to use
- **Action Input**: What to pass to the tool
- **Observation**: What the tool returned
- **Final Answer**: When we have the result

---

## Building ReAct Agent

### Step 3: Create ReAct Agent Class

Create `react_agent.py`:

```python
# react_agent.py

from enum import Enum
from typing import Dict, Any, List
import re

class ReActStep(Enum):
    """ReAct execution steps."""
    THOUGHT = "thought"
    ACTION = "action"
    OBSERVATION = "observation"
    FINAL = "final"

class ReActAgent:
    """
    ReAct (Reasoning + Acting) Agent.
    
    Follows a thought-action-observation loop to solve problems.
    """
    
    def __init__(self, tools: Dict[str, callable], max_steps: int = 10):
        """
        Initialize ReAct agent.
        
        Args:
            tools: Dict of tool_name -> tool_function
            max_steps: Maximum iterations before giving up
        """
        self.tools = tools
        self.max_steps = max_steps
        self.history = []  # Track execution
    
    def parse_response(self, text: str) -> Dict[str, str]:
        """
        Parse LLM response for ReAct format.
        
        Looks for Thought, Action, Action Input, and Final Answer.
        
        Args:
            text: LLM response text
        
        Returns:
            Dict with parsed components
        """
        patterns = {
            "thought": r"Thought:\s*(.*?)(?:\nAction:|$)",
            "action": r"Action:\s*(.*?)(?:\nAction Input:|$)",
            "action_input": r"Action Input:\s*(.*?)(?:\n|$)",
            "final": r"Final Answer:\s*(.*)"
        }
        
        result = {}
        for key, pattern in patterns.items():
            match = re.search(pattern, text, re.DOTALL | re.IGNORECASE)
            if match:
                result[key] = match.group(1).strip()
        
        return result
    
    def execute_tool(self, tool_name: str, tool_input: str) -> str:
        """
        Execute a tool with given input.
        
        Args:
            tool_name: Name of tool to execute
            tool_input: Input for the tool
        
        Returns:
            Tool result as string
        """
        # Normalize tool name
        tool_name = tool_name.lower().strip()
        
        if tool_name not in self.tools:
            return f"Error: Unknown tool '{tool_name}'. Available: {list(self.tools.keys())}"
        
        try:
            tool = self.tools[tool_name]
            
            # Try to parse JSON input
            if tool_input.strip().startswith('{'):
                import json
                args = json.loads(tool_input)
                result = tool(**args)
            else:
                # Pass as single argument
                result = tool(tool_input)
            
            return str(result)
            
        except Exception as e:
            return f"Error executing tool: {str(e)}"
    
    def run(self, query: str) -> str:
        """
        Run ReAct loop to answer query.
        
        Args:
            query: Question to answer
        
        Returns:
            Final answer or error message
        """
        # Start with the question
        prompt = f"""Question: {query}

Let's solve this step by step using the ReAct format.

Format your response as:
Thought: [your reasoning about what to do next]
Action: [tool name to use]
Action Input: [input for the tool]

Or when you have the answer:
Final Answer: [your final answer]"""
        
        # ReAct loop
        for step_num in range(self.max_steps):
            # Get LLM's response
            response = self.llm.generate(prompt)
            
            # Parse the response
            parsed = self.parse_response(response)
            
            # Log for debugging
            self.history.append({
                "step": step_num,
                "response": response,
                "parsed": parsed
            })
            
            # Check for final answer
            if "final" in parsed:
                return parsed["final"]
            
            # Execute action if present
            if "action" in parsed and "action_input" in parsed:
                tool_name = parsed["action"]
                tool_input = parsed.get("action_input", "")
                thought = parsed.get("thought", "")
                
                # Execute the tool
                tool_result = self.execute_tool(tool_name, tool_input)
                
                # Add observation to prompt for next iteration
                prompt += f"\n\nThought: {thought}\n"
                prompt += f"Action: {tool_name}\n"
                prompt += f"Action Input: {tool_input}\n"
                prompt += f"Observation: {tool_result}\n\nThought: "
            else:
                # No action found, continue with empty thought
                prompt += "\nThought: "
        
        return "Maximum steps reached without finding final answer."

# Add OllamaLLM property to ReActAgent
@property
def llm(self):
    """Lazy-load Ollama LLM."""
    if not hasattr(self, '_llm'):
        from openclaw.llms.ollama import OllamaLLM
        self._llm = OllamaLLM(model="llama3.2:3b")
    return self._llm

ReActAgent.llm = llm
```

Wait, that's wrong. Let me fix it:
<minimax:tool_call>
<invoke name="write">
<parameter name="content"># react_agent.py

from enum import Enum
from typing import Dict, Any, List
import re
from openclaw.llms.ollama import OllamaLLM

class ReActStep(Enum):
    """ReAct execution steps."""
    THOUGHT = "thought"
    ACTION = "action"
    OBSERVATION = "observation"
    FINAL = "final"

class ReActAgent:
    """
    ReAct (Reasoning + Acting) Agent.
    
    Follows a thought-action-observation loop to solve problems.
    """
    
    def __init__(self, tools: Dict[str, callable], max_steps: int = 10):
        """
        Initialize ReAct agent.
        
        Args:
            tools: Dict of tool_name -> tool_function
            max_steps: Maximum iterations before giving up
        """
        self.tools = tools
        self.max_steps = max_steps
        self.history = []  # Track execution
        self._llm = OllamaLLM(model="llama3.2:3b")
    
    @property
    def llm(self):
        return self._llm
    
    def parse_response(self, text: str) -> Dict[str, str]:
        """
        Parse LLM response for ReAct format.
        
        Looks for Thought, Action, Action Input, and Final Answer.
        """
        patterns = {
            "thought": r"Thought:\s*(.*?)(?:\nAction:|$)",
            "action": r"Action:\s*(.*?)(?:\nAction Input:|$)",
            "action_input": r"Action Input:\s*(.*?)(?:\n|$)",
            "final": r"Final Answer:\s*(.*)"
        }
        
        result = {}
        for key, pattern in patterns.items():
            match = re.search(pattern, text, re.DOTALL | re.IGNORECASE)
            if match:
                result[key] = match.group(1).strip()
        
        return result
    
    def execute_tool(self, tool_name: str, tool_input: str) -> str:
        """Execute a tool with given input."""
        tool_name = tool_name.lower().strip()
        
        if tool_name not in self.tools:
            return f"Error: Unknown tool '{tool_name}'. Available: {list(self.tools.keys())}"
        
        try:
            tool = self.tools[tool_name]
            
            # Try to parse JSON input
            if tool_input.strip().startswith('{'):
                import json
                args = json.loads(tool_input)
                result = tool(**args)
            else:
                result = tool(tool_input)
            
            return str(result)
            
        except Exception as e:
            return f"Error executing tool: {str(e)}"
    
    def run(self, query: str) -> str:
        """Run ReAct loop to answer query."""
        prompt = f"""Question: {query}

Solve step by step using this format:
Thought: [reasoning]
Action: [tool name]
Action Input: [input]

Or when done:
Final Answer: [answer]"""
        
        for step_num in range(self.max_steps):
            response = self.llm.generate(prompt)
            parsed = self.parse_response(response)
            
            self.history.append({"step": step_num, "parsed": parsed})
            
            if "final" in parsed:
                return parsed["final"]
            
            if "action" in parsed and "action_input" in parsed:
                tool_name = parsed["action"]
                tool_input = parsed.get("action_input", "")
                thought = parsed.get("thought", "")
                
                tool_result = self.execute_tool(tool_name, tool_input)
                
                prompt += f"\nThought: {thought}\n"
                prompt += f"Action: {tool_name}\n"
                prompt += f"Action Input: {tool_input}\n"
                prompt += f"Observation: {tool_result}\n\nThought: "
        
        return "Maximum steps reached."
