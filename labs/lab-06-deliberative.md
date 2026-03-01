# Lab 6: Deliberative Agents

## Learning Objectives
- Implement deliberative reasoning architectures
- Create planning and execution phases
- Use symbolic reasoning with LLMs
- Implement BDI (Belief-Desire-Intention) patterns
- Handle plan revision and recovery

## Prerequisites
- Labs 1-5 completed
- Understanding of planning algorithms
- Knowledge of symbolic AI concepts

## Setup Requirements
1. Install: `pip install unified-planning`
2. Create lab6 directory
3. Review PDDL basics

## Step-by-Step Instructions

### Step 1: BDI Architecture Overview
BDI (Belief-Desire-Intention) model:
- **Beliefs**: Current knowledge about the world
- **Desires**: Goals the agent wants to achieve
- **Intentions**: Selected plans to achieve desires

### Step 2: Basic BDI Agent
```python
from typing import List, Dict, Any, Optional
from datetime import datetime

class BDIAgent:
    def __init__(self, name: str):
        self.name = name
        self.beliefs = {}      # What agent knows
        self.desires = []      # What agent wants
        self.intentions = []   # What agent plans to do
        self.plans = {}        # Available plans
    
    def update_belief(self, key: str, value: Any):
        """Update agent's beliefs about the world."""
        self.beliefs[key] = value
    
    def add_desire(self, goal: str):
        """Add a desire/goal."""
        if goal not in self.desires:
            self.desires.append(goal)
    
    def select_intention(self):
        """Select intention from desires based on beliefs."""
        for desire in self.desires:
            if self.can_achieve(desire):
                plan = self.create_plan(desire)
                if plan:
                    self.intentions = plan
                    return plan
        return None
    
    def can_achieve(self, goal: str) -> bool:
        """Check if goal can be achieved with current beliefs."""
        # Placeholder: implement goal-specific logic
        return True
    
    def create_plan(self, goal: str) -> List[str]:
        """Create a plan to achieve goal."""
        # Placeholder: return sequence of actions
        return [f"action_{goal}"]
    
    def execute(self) -> Dict[str, Any]:
        """Execute current intentions."""
        results = []
        for intention in self.intentions:
            result = self.perform_action(intention)
            results.append({"action": intention, "result": result})
            # Update beliefs based on result
            self.update_belief(f"last_{intention}", result)
        return {"status": "completed", "results": results}
    
    def perform_action(self, action: str) -> str:
        """Perform a single action."""
        return f"executed_{action}"
```

### Step 3: Planning Agent with LLM
```python
class DeliberativePlanningAgent(BDIAgent):
    """BDI agent with LLM-powered planning."""
    
    def __init__(self, name: str, llm):
        super().__init__(name)
        self.llm = llm
    
    def deliberate(self) -> List[str]:
        """Use LLM to generate plans."""
        # Build context
        context = f"Beliefs: {self.beliefs}\n"
        context += f"Desires: {self.desires}\n"
        context += "Create a plan to achieve the desires."
        
        # Get LLM plan
        prompt = f"{context}\nProvide a numbered list of actions."
        response = self.llm.generate(prompt)
        
        # Parse plan
        plan = [line.split('. ', 1)[1] if '. ' in line else line 
                for line in response.split('\n') 
                if line.strip() and line[0].isdigit()]
        
        self.intentions = plan
        return plan
    
    def replan(self, failure_reason: str):
        """Create new plan after failure."""
        context = f"Previous plan failed: {failure_reason}\n"
        context += f"Current beliefs: {self.beliefs}\n"
        context += f"Goal: {self.desires}\n"
        context += "Create a new plan."
        
        prompt = f"{context}\nProvide a numbered list of actions."
        response = self.llm.generate(prompt)
        
        plan = [line.split('. ', 1)[1] if '. ' in line else line 
                for line in response.split('\n') 
                if line.strip() and line[0].isdigit()]
        
        self.intentions = plan
        return plan
```

### Step 4: Plan Execution with Monitoring
```python
class MonitoredExecutionAgent(DeliberativePlanningAgent):
    """Agent that monitors execution and replans on failure."""
    
    def __init__(self, name: str, llm, max_retries: int = 3):
        super().__init__(name, llm)
        self.max_retries = max_retries
        self.execution_log = []
    
    def execute_with_monitoring(self) -> Dict[str, Any]:
        """Execute plan with failure monitoring."""
        if not self.intentions:
            self.deliberate()
        
        results = []
        for i, action in enumerate(self.intentions):
            result = self.perform_action(action)
            
            self.execution_log.append({
                "action": action,
                "result": result,
                "step": i + 1,
                "timestamp": datetime.now().isoformat()
            })
            
            # Check for failure
            if self.is_failure(result):
                # Try to replan
                for retry in range(self.max_retries):
                    new_plan = self.replan(f"Step {i+1} failed: {result}")
                    if new_plan:
                        self.intentions = new_plan
                        break
                else:
                    return {
                        "status": "failed",
                        "failed_at": i + 1,
                        "log": self.execution_log
                    }
        
        return {
            "status": "success",
            "steps_completed": len(self.intentions),
            "log": self.execution_log
        }
    
    def is_failure(self, result: str) -> bool:
        """Check if result indicates failure."""
        failure_indicators = ["error", "failed", "cannot", "unable"]
        result_lower = result.lower()
        return any(indicator in result_lower for indicator in failure_indicators)
```

## Exercises
1. **Basic**: Create deliberative agent for task scheduling
2. **Intermediate**: Implement plan failure recovery
3. **Advanced**: Add belief revision mechanism

## Next Steps
- Lab 7: Vector stores for semantic memory
- Apply to real-world agent scenarios
