# Lab 10: Customer Service Agent

## Learning Objectives
- Build practical customer service agent
- Integrate with ticketing systems
- Handle common customer queries
- Implement escalation procedures
- Create conversational flows

## Prerequisites
- Labs 1-9 completed
- Understanding of customer service workflows
- Knowledge of REST APIs

## Setup Requirements
1. Install: `pip install fastapi uvicorn`
2. Create lab10 directory
3. Set up mock database

## Step-by-Step Instructions

### Step 1: Customer Service Agent Design
```python
from openclaw import Agent, Tool
from typing import Dict, Any, Optional
from datetime import datetime

class CustomerServiceAgent(Agent):
    def __init__(self):
        super().__init__(
            name="CSAgent",
            description="Customer service support agent",
            system_prompt="""You are a helpful customer service agent.
            You can help with: billing, technical support, account issues.
            Be polite, empathetic, and solution-oriented.
            Always try to resolve issues without escalation.""",
            tools=[
                self.lookup_account,
                self.check_order_status,
                self.create_ticket,
                self.escalate_issue
            ]
        )
        self.ticket_counter = 1000
        self.conversation_history = []
    
    @Tool
    def lookup_account(self, email: str) -> Dict[str, Any]:
        """Look up customer account by email."""
        # Mock account database
        accounts = {
            "test@example.com": {
                "email": "test@example.com",
                "name": "Test User",
                "status": "active",
                "plan": "premium",
                "joined": "2023-01-15",
                "tickets": 2
            }
        }
        
        return accounts.get(email, {
            "error": "Account not found",
            "suggestion": "Please check the email address"
        })
    
    @Tool
    def check_order_status(self, order_id: str) -> Dict[str, Any]:
        """Check status of an order."""
        # Mock order database
        orders = {
            "ORD-001": {
                "order_id": "ORD-001",
                "status": "shipped",
                "items": ["Widget Pro", "Extra Battery"],
                "total": 149.99,
                "shipped_date": "2024-01-10",
                "tracking": "1Z999AA10123456784"
            }
        }
        
        return orders.get(order_id, {
            "error": "Order not found",
            "suggestion": "Verify the order ID"
        })
    
    @Tool
    def create_ticket(self, issue: str, priority: str = "medium") -> Dict[str, Any]:
        """Create a support ticket."""
        self.ticket_counter += 1
        ticket_id = f"TKT-{self.ticket_counter:05d}"
        
        return {
            "ticket_id": ticket_id,
            "issue": issue,
            "priority": priority,
            "status": "open",
            "created_at": datetime.now().isoformat(),
            "next_steps": "You will receive an email within 24 hours"
        }
    
    @Tool
    def escalate_issue(self, reason: str, customer_email: str) -> Dict[str, Any]:
        """Escalate issue to human support."""
        self.ticket_counter += 1
        ticket_id = f"ESC-{self.ticket_counter:05d}"
        
        return {
            "escalation_id": ticket_id,
            "reason": reason,
            "customer_email": customer_email,
            "status": "escalated",
            "estimated_response": "Within 2 hours",
            "note": "A senior support agent will contact you"
        }
```

### Step 2: FAQ Knowledge Base
```python
class FAQKnowledgeBase:
    """Frequently Asked Questions database."""
    
    def __init__(self):
        self.faqs = [
            {
                "question": "How do I reset my password?",
                "answer": "Go to Settings > Security > Reset Password. You'll receive an email with reset instructions.",
                "keywords": ["password", "reset", "forgot"]
            },
            {
                "question": "How do I cancel my subscription?",
                "answer": "Go to Settings > Billing > Cancel Subscription. Note that you'll lose access at the end of billing period.",
                "keywords": ["cancel", "subscription", "stop"]
            },
            {
                "question": "When will I be charged?",
                "answer": "You're charged on the same date each month. Check Billing for your next charge date.",
                "keywords": ["charge", "billing", "payment", "when"]
            }
        ]
    
    def search(self, query: str) -> Optional[Dict]:
        """Search FAQ by query."""
        query_lower = query.lower()
        
        for faq in self.faqs:
            if any(kw in query_lower for kw in faq["keywords"]):
                return faq
        
        return None
    
    def get_all(self) -> list:
        """Get all FAQs."""
        return [{"question": f["question"], "answer": f["answer"]} for f in self.faqs]
```

### Step 3: Sentiment Analysis
```python
class SimpleSentimentAnalyzer:
    """Basic sentiment analysis for customer messages."""
    
    def __init__(self):
        self.positive_words = ["thank", "great", "love", "awesome", "perfect", "happy"]
        self.negative_words = ["angry", "frustrated", "terrible", "awful", "hate", "worst"]
        self.escalation_words = ["supervisor", "manager", "lawsuit", "legal", "complaint"]
    
    def analyze(self, text: str) -> Dict[str, Any]:
        """Analyze sentiment of text."""
        text_lower = text.lower()
        
        positive_count = sum(1 for w in self.positive_words if w in text_lower)
        negative_count = sum(1 for w in self.negative_words if w in text_lower)
        escalation_count = sum(1 for w in self.escalation_words if w in text_lower)
        
        if escalation_count > 0:
            sentiment = "escalation_needed"
        elif negative_count > positive_count:
            sentiment = "negative"
        elif positive_count > negative_count:
            sentiment = "positive"
        else:
            sentiment = "neutral"
        
        return {
            "sentiment": sentiment,
            "positive_score": positive_count,
            "negative_score": negative_count,
            "escalation_indicators": escalation_count
        }
```

### Step 4: Full Integration
```python
# Create agent with all components
agent = CustomerServiceAgent()
faq = FAQKnowledgeBase()
sentiment = SimpleSentimentAnalyzer()

def handle_customer_message(message: str, email: str = None) -> str:
    """Handle incoming customer message."""
    # Analyze sentiment
    sentiment_result = sentiment.analyze(message)
    
    # Check FAQ first
    faq_result = faq.search(message)
    if faq_result:
        response = f"{faq_result['answer']}\n\nIs there anything else I can help with?"
        
        # Warn if negative sentiment
        if sentiment_result["sentiment"] == "negative":
            response += "\n\nI'm sorry to hear you're having issues. Would you like me to create a support ticket?"
        
        return response
    
    # Use agent for complex queries
    response = agent.run(message)
    
    # Escalate if needed
    if sentiment_result["escalation_needed"] and email:
        escalation = agent.escalate_issue(
            reason="Customer requested supervisor",
            customer_email=email
        )
        response += f"\n\n{escalation['note']}"
    
    return response
```

## Exercises
1. **Basic**: Create FAQ knowledge base
2. **Intermediate**: Implement sentiment analysis
3. **Advanced**: Add multi-language support

## Next Steps
- Lab 11: Deployment to production
- Apply to real customer service scenarios
