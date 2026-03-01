# Lab 5: Round-Robin Communication

## Learning Objectives
- Implement round-robin agent communication
- Create agent specialization patterns
- Handle message passing between agents
- Manage conversation state in multi-agent systems
- Debug inter-agent communication

## Prerequisites
- Labs 1-4 completed
- Understanding of agent communication
- Basic knowledge of message queues

## Setup Requirements
1. Create lab5 directory
2. Review Python's asyncio basics
3. Set up multiple agent personas

## Step-by-Step Instructions

### Step 1: Agent Role Definition
```python
from typing import List, Dict, Any
from datetime import datetime

class SpecialistAgent:
    def __init__(self, name: str, expertise: str, system_prompt: str):
        self.name = name
        self.expertise = expertise
        self.system_prompt = system_prompt
        self.message_queue = []
    
    def receive(self, message: str, sender: str):
        """Receive a message from another agent."""
        self.message_queue.append({
            "sender": sender,
            "message": message,
            "timestamp": datetime.now()
        })
    
    def process(self) -> str:
        """Process the next message in queue."""
        if not self.message_queue:
            return "No messages to process"
        
        msg = self.message_queue.pop(0)
        # Process based on expertise
        return f"{self.name} ({self.expertise}): Processed '{msg['message']}'"
    
    def clear_queue(self):
        """Clear all pending messages."""
        self.message_queue = []
```

### Step 2: Round-Robin Coordinator
```python
class RoundRobinCoordinator:
    def __init__(self, agents: List[SpecialistAgent]):
        self.agents = agents
        self.current_index = 0
        self.history = []
    
    def send_message(self, message: str) -> str:
        """Send message to next agent in round-robin."""
        agent = self.agents[self.current_index]
        agent.receive(message, "Coordinator")
        response = agent.process()
        
        self.history.append({
            "to": agent.name,
            "message": message,
            "response": response
        })
        
        # Move to next agent
        self.current_index = (self.current_index + 1) % len(self.agents)
        return response
    
    def broadcast(self, message: str) -> List[str]:
        """Send message to all agents."""
        responses = []
        for agent in self.agents:
            agent.receive(message, "Broadcast")
            responses.append(agent.process())
        return responses
```

### Step 3: Create Specialized Agents
```python
# Create specialized agents
math_agent = SpecialistAgent(
    name="MathBot",
    expertise="Mathematics",
    system_prompt="You solve mathematical problems step by step."
)

language_agent = SpecialistAgent(
    name="LinguaBot", 
    expertise="Language & Translation",
    system_prompt="You help with language and translation tasks."
)

logic_agent = SpecialistAgent(
    name="LogicBot",
    expertise="Logic & Reasoning",
    system_prompt="You solve logic puzzles and reasoning problems."
)

# Create coordinator
coordinator = RoundRobinCoordinator([math_agent, language_agent, logic_agent])

# Test round-robin
responses = []
for query in ["Calculate 15 * 15", "Translate hello to Spanish", "What is 2+2?"]:
    response = coordinator.send_message(query)
    responses.append(response)
    print(f"Query: {query} -> {response}")
```

### Step 4: Stateful Multi-Agent System
```python
class StatefulRoundRobin:
    """Round-robin with persistent state."""
    
    def __init__(self, agents: List[SpecialistAgent]):
        self.agents = agents
        self.context = {}  # Shared context between agents
        self.current_index = 0
        self.round_count = 0
    
    def process_query(self, query: str) -> Dict[str, Any]:
        """Process query through multiple agents."""
        results = []
        
        # First pass: all agents see the query
        for agent in self.agents:
            agent.receive(query, "User")
        
        # Round-robin processing
        max_rounds = 3
        for round_num in range(max_rounds):
            agent = self.agents[self.current_index]
            response = agent.process()
            
            results.append({
                "round": round_num + 1,
                "agent": agent.name,
                "response": response
            })
            
            # Update shared context
            self.context[agent.name] = response
            
            self.current_index = (self.current_index + 1) % len(self.agents)
        
        self.round_count += 1
        return {"results": results, "context": self.context}
```

## Exercises
1. **Basic**: Create 3 specialized agents (math, language, logic)
2. **Intermediate**: Implement persistent message history
3. **Advanced**: Add priority-based round-robin with task routing

## Next Steps
- Lab 6: Deliberative agent systems
- Explore more complex multi-agent architectures
