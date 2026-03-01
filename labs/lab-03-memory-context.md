# Lab 3: Memory & Context

## Learning Objectives
- Implement conversation history persistence
- Create short-term and long-term memory systems
- Use context windows effectively
- Understand memory optimization techniques
- Build agents that remember user preferences

## Prerequisites
- Labs 1 & 2 completed
- Basic understanding of databases
- Python data structures knowledge

## Why Memory?
Without memory, each conversation starts fresh. Memory enables:
- **Continuity**: Agent remembers previous messages
- **Personalization**: Remembers user preferences
- **Context**: Understands follow-up questions

---

## Detailed Worked Steps

### Step 1: Set Up Lab 3 Environment

```bash
# Create new directory
mkdir lab3-memory
cd lab3-memory

# Activate virtual environment
source ../venv/bin/activate

# No extra installs needed - using built-in sqlite3
```

---

## Short-Term Memory (In-Memory)

### Step 2: Create Conversation Memory Class

Create `conversation_memory.py`:

```python
# conversation_memory.py

from typing import List, Dict, Any
from datetime import datetime
from dataclasses import dataclass, asdict

@dataclass
class MemoryEntry:
    """Single conversation entry."""
    id: int
    user_message: str
    agent_response: str
    timestamp: datetime
    metadata: Dict[str, Any]

class ConversationMemory:
    """
    Short-term memory for conversation history.
    
    Stores recent conversations in memory for fast access.
    Automatically trims old entries when max size exceeded.
    """
    
    def __init__(self, max_entries: int = 100):
        """
        Initialize conversation memory.
        
        Args:
            max_entries: Maximum number of entries to store
        """
        self.max_entries = max_entries
        self.entries: List[MemoryEntry] = []
        self.next_id = 1
    
    def add(self, user_msg: str, agent_resp: str, metadata: Dict = None):
        """
        Add a new conversation entry.
        
        Args:
            user_msg: User's message
            agent_resp: Agent's response
            metadata: Optional metadata dict
        """
        entry = MemoryEntry(
            id=self.next_id,
            user_message=user_msg,
            agent_response=agent_resp,
            timestamp=datetime.now(),
            metadata=metadata or {}
        )
        self.entries.append(entry)
        self.next_id += 1
        
        # Auto-trim if exceeds max
        if len(self.entries) > self.max_entries:
            self.entries = self.entries[-self.max_entries:]
    
    def get_recent(self, n: int = 5) -> List[MemoryEntry]:
        """Get the n most recent entries."""
        return self.entries[-n:] if self.entries else []
    
    def get_context(self, max_tokens: int = 500) -> str:
        """
        Get conversation context formatted for LLM prompt.
        
        Args:
            max_tokens: Approximate max tokens to include
        
        Returns:
            Formatted string of recent conversation
        """
        recent = self.get_recent(10)
        context = []
        token_count = 0
        
        # Build context from most recent backwards
        for entry in reversed(recent):
            entry_text = f"User: {entry.user_message}\nAgent: {entry.agent_response}"
            entry_tokens = len(entry_text.split())  # Rough token estimate
            
            if token_count + entry_tokens > max_tokens:
                break
                
            context.append(entry_text)
            token_count += entry_tokens
        
        return "\n".join(reversed(context))
    
    def clear(self):
        """Clear all memory."""
        self.entries = []
    
    def __len__(self):
        return len(self.entries)
```

### Step 3: Test Short-Term Memory

Create `test_memory.py`:

```python
# test_memory.py

from conversation_memory import ConversationMemory

# Create memory instance
memory = ConversationMemory(max_entries=10)

# Add some conversations
memory.add("Hi, my name is Alice", "Hello Alice! Nice to meet you!")
memory.add("What's my name?", "Your name is Alice, you told me earlier!")
memory.add("What can you help with?", "I can help with questions, tasks, and more!")

# Get context
print("=== Conversation Context ===")
print(memory.get_context(max_tokens=100))
print(f"\nTotal entries: {len(memory)}")

# Get recent
print("\n=== Recent 2 ===")
for entry in memory.get_recent(2):
    print(f"{entry.timestamp.strftime('%H:%M:%S')} - User: {entry.user_message[:30]}...")
```

Run it:
```bash
python test_memory.py
```

---

## Long-Term Memory (SQLite)

### Step 4: Create SQLite Memory

Create `sqlite_memory.py`:

```python
# sqlite_memory.py

import sqlite3
import json
from typing import List, Dict, Any
from datetime import datetime

class SQLiteMemory:
    """
    Persistent memory using SQLite database.
    
    Survives restarts and allows unlimited history.
    """
    
    def __init__(self, db_path: str = "memory.db"):
        """
        Initialize SQLite memory.
        
        Args:
            db_path: Path to SQLite database file
        """
        self.db_path = db_path
        self._init_db()
    
    def _init_db(self):
        """Create database table if not exists."""
        conn = sqlite3.connect(self.db_path)
        conn.execute("""
            CREATE TABLE IF NOT EXISTS conversations (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                user_message TEXT NOT NULL,
                agent_response TEXT NOT NULL,
                metadata TEXT,
                timestamp DATETIME DEFAULT CURRENT_TIMESTAMP
            )
        """)
        conn.commit()
        conn.close()
    
    def store(self, user_msg: str, agent_resp: str, metadata: Dict = None):
        """
        Store a conversation in database.
        
        Args:
            user_msg: User's message
            agent_resp: Agent's response
            metadata: Optional metadata
        """
        conn = sqlite3.connect(self.db_path)
        conn.execute(
            "INSERT INTO conversations (user_message, agent_response, metadata) VALUES (?, ?, ?)",
            (user_msg, agent_resp, json.dumps(metadata) if metadata else None)
        )
        conn.commit()
        conn.close()
    
    def retrieve(self, limit: int = 10) -> List[Dict[str, Any]]:
        """
        Retrieve recent conversations.
        
        Args:
            limit: Number of recent entries to retrieve
        
        Returns:
            List of conversation dicts
        """
        conn = sqlite3.connect(self.db_path)
        cursor = conn.execute(
            "SELECT user_message, agent_response, timestamp FROM conversations ORDER BY timestamp DESC LIMIT ?",
            (limit,)
        )
        rows = cursor.fetchall()
        conn.close()
        
        return [
            {"user": r[0], "agent": r[1], "time": r[2]}
            for r in rows
        ]
    
    def search(self, query: str) -> List[Dict[str, Any]]:
        """Search conversations for keyword."""
        conn = sqlite3.connect(self.db_path)
        cursor = conn.execute(
            "SELECT user_message, agent_response, timestamp FROM conversations WHERE user_message LIKE ? OR agent_response LIKE ?",
            (f"%{query}%", f"%{query}%")
        )
        rows = cursor.fetchall()
        conn.close()
        return [{"user": r[0], "agent": r[1], "time": r[2]} for r in rows]
    
    def clear(self):
        """Clear all conversations."""
        conn = sqlite3.connect(self.db_path)
        conn.execute("DELETE FROM conversations")
        conn.commit()
        conn.close()
```

### Step 5: Test SQLite Memory

```bash
python -c "
from sqlite_memory import SQLiteMemory

# Create memory (creates database file)
memory = SQLiteMemory('my_memory.db')

# Store conversations
memory.store('Hello', 'Hi there!')
memory.store('What is 2+2?', '2+2 equals 4')

# Retrieve
print('Recent conversations:')
for c in memory.retrieve(5):
    print(f\"  User: {c['user']}\")
    print(f\"  Agent: {c['agent']}\")
"
```

---

## Building Memory-Enabled Agents

### Step 6: Create Memory Agent

Create `memory_agent.py`:

```python
# memory_agent.py

from openclaw import Agent, Runner
from openclaw.llms.ollama import OllamaLLM
from conversation_memory import ConversationMemory

class MemoryAgent(Agent):
    """
    Agent with conversation memory capabilities.
    
    Remembers recent conversations and includes them in context.
    """
    
    def __init__(self, memory: ConversationMemory):
        llm = OllamaLLM(model="llama3.2:3b")
        super().__init__(
            name="MemoryAssistant",
            description="Agent with conversation memory",
            system_prompt="""You are a helpful assistant that remembers the conversation.
            Use the conversation history to provide relevant responses.
            If user mentions something from earlier, acknowledge it.""",
            llm=llm
        )
        self.memory = memory
    
    def run(self, user_input: str) -> str:
        # Step 1: Get context from memory
        history = self.memory.get_context(max_tokens=300)
        
        # Step 2: Build prompt with history
        if history:
            prompt = f"Previous conversation:\n{history}\n\nUser: {user_input}\nAssistant:"
        else:
            prompt = f"User: {user_input}\nAssistant:"
        
        # Step 3: Generate response
        response = self.llm.generate(prompt=prompt, max_tokens=200)
        
        # Step 4: Store in memory
        self.memory.add(user_input, response)
        
        return response

# Run the agent
if __name__ == "__main__":
    # Create memory
    memory = ConversationMemory(max_entries=50)
    
    # Create agent with memory
    agent = MemoryAgent(memory)
    runner = Runner(agent)
    
    # Test conversation
    print("=== Testing Memory Agent ===")
    
    responses = [
        "My favorite color is blue.",
        "What's my favorite color?",
        "What was the first thing I told you?"
    ]
    
    for msg in responses:
        print(f"\n👤 User: {msg}")
        response = runner.run(msg)
        print(f"🤖 Agent: {response}")
```

### Step 7: Run Memory Agent

```bash
python memory_agent.py
```

**Watch for:**
- First message: No memory context
- Second message: Agent recalls "favorite color is blue"
- Third message: Agent references first message

---

## Memory Optimization

### Step 8: Optimized Memory with Summarization

Create `optimized_memory.py`:

```python
# optimized_memory.py

class OptimizedMemory:
    """
    Memory with automatic pruning and summarization.
    
    When memory fills up, instead of losing old data,
    it summarizes and condenses older entries.
    """
    
    def __init__(self, max_size: int = 1000):
        self.max_size = max_size
        self.entries = []
    
    def add(self, entry: str):
        """Add entry, summarize old ones if needed."""
        self.entries.append(entry)
        
        if len(self.entries) > self.max_size:
            self._summarize_oldest()
    
    def _summarize_oldest(self):
        """Keep recent entries, summarize the rest."""
        if len(self.entries) > self.max_size // 2:
            # Get entries to summarize
            old_entries = self.entries[:-self.max_size // 2]
            
            # Create summary
            summary = f"[{len(old_entries)} previous entries - conversation about various topics]"
            
            # Keep summary + recent entries
            self.entries = [summary] + self.entries[-self.max_size // 2:]
    
    def get_context(self, max_tokens: int = 500) -> str:
        """Get context, truncating if needed."""
        # Simple implementation - join and truncate
        context = "\n".join(self.entries)
        
        words = context.split()
        if len(words) > max_tokens:
            context = " ".join(words[-max_tokens:])
        
        return context
```

---

## Exercises

### Exercise 1: Basic (15 min)
Add user preference memory:
- Store user's name when they tell you
- Store preferred greeting style
- Have agent use these preferences

### Exercise 2: Intermediate (30 min)
Implement importance-based retention:
- Score each entry (1-10) based on importance
- Keep high-importance entries longer
- Auto-prune low-importance entries

### Exercise 3: Advanced (45 min)
Create semantic memory using embeddings:
- Use sentence-transformers for embeddings
- Store embeddings in vector database
- Retrieve semantically similar memories

---

## What You Learned

✅ How to create short-term in-memory conversation history
✅ How to persist memory with SQLite
✅ How to build agents with memory capabilities
✅ How to optimize memory with summarization
✅ How to search and retrieve past conversations

---

## Next Steps

- **Lab 4:** Learn ReAct pattern for structured reasoning
- **Lab 7:** Use vector stores for scalable semantic memory
