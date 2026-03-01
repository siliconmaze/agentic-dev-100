# Lab 8: Security Oversight

## Learning Objectives
- Implement security layers for agents
- Create input/output validation
- Add rate limiting and quotas
- Implement permission systems
- Monitor for adversarial prompts

## Prerequisites
- Labs 1-7 completed
- Understanding of security principles
- Knowledge of LLM vulnerabilities

## Setup Requirements
1. Install: `pip install regex python-jose`
2. Create lab8 directory
3. Review OWASP Top 10 for LLMs

## Step-by-Step Instructions

### Step 1: Input Validation Layer
```python
import re
from typing import Tuple, List

class InputValidator:
    """Validate and sanitize user inputs."""
    
    def __init__(self):
        # Blocked patterns for prompt injection
        self.blocked_patterns = [
            r"(?i)system prompt",
            r"(?i)ignore previous",
            r"(?i)override instructions",
            r"<script>",
            r"javascript:",
            r"eval\(",
            r"exec\(",
            r"\\\x",
        ]
        self.max_length = 10000
    
    def validate(self, input_text: str) -> Tuple[bool, str]:
        """Validate input text."""
        # Length check
        if len(input_text) > self.max_length:
            return False, f"Input too long (max {self.max_length})"
        
        # Pattern checks
        for pattern in self.blocked_patterns:
            if re.search(pattern, input_text, re.IGNORECASE):
                return False, f"Blocked pattern detected: {pattern}"
        
        # Encoding check
        try:
            input_text.encode('utf-8')
        except UnicodeError:
            return False, "Invalid encoding"
        
        return True, "Valid"
    
    def sanitize(self, input_text: str) -> str:
        """Sanitize input by removing dangerous content."""
        # Remove potential injection attempts
        sanitized = input_text
        for pattern in self.blocked_patterns:
            sanitized = re.sub(pattern, "[BLOCKED]", sanitized, flags=re.IGNORECASE)
        return sanitized
```

### Step 2: Permission System
```python
from typing import Dict, Set, Optional
from datetime import datetime, timedelta

class PermissionManager:
    """Manage agent permissions and access control."""
    
    def __init__(self):
        self.roles = {
            "user": {
                "allowed_tools": ["search", "calculator", "weather"],
                "max_requests_per_hour": 100,
                "can_access_internal": False
            },
            "admin": {
                "allowed_tools": "all",
                "max_requests_per_hour": 1000,
                "can_access_internal": True
            },
            "guest": {
                "allowed_tools": ["search"],
                "max_requests_per_hour": 10,
                "can_access_internal": False
            }
        }
        self.user_requests: Dict[str, List[datetime]] = {}
    
    def check_permission(self, user_role: str, tool: str) -> bool:
        """Check if user has permission to use tool."""
        if user_role not in self.roles:
            return False
        
        role_config = self.roles[user_role]
        
        if role_config["allowed_tools"] == "all":
            return True
        
        return tool in role_config["allowed_tools"]
    
    def check_rate_limit(self, user_id: str) -> bool:
        """Check if user has exceeded rate limit."""
        now = datetime.now()
        hour_ago = now - timedelta(hours=1)
        
        if user_id not in self.user_requests:
            self.user_requests[user_id] = []
        
        # Clean old requests
        self.user_requests[user_id] = [
            t for t in self.user_requests[user_id] if t > hour_ago
        ]
        
        # Check limit
        role = "user"  # Get from user config
        limit = self.roles.get(role, {}).get("max_requests_per_hour", 100)
        
        if len(self.user_requests[user_id]) >= limit:
            return False
        
        self.user_requests[user_id].append(now)
        return True
```

### Step 3: Output Sanitization
```python
class OutputSanitizer:
    """Sanitize agent outputs for security."""
    
    def __init__(self):
        self.sensitive_patterns = [
            (r'\b\d{3}-\d{2}-\d{4}\b', '[SSN]'),  # SSN
            (r'\b\d{16}\b', '[CREDIT_CARD]'),      # Credit card
            (r'api[_-]?key["\']?\s*[:=]\s*["\']?\S+', '[API_KEY]'),  # API keys
            (r'password["\']?\s*[:=]\s*["\']?\S+', '[PASSWORD]'),    # Passwords
        ]
    
    def sanitize(self, output: str) -> str:
        """Remove sensitive information from output."""
        sanitized = output
        for pattern, replacement in self.sensitive_patterns:
            sanitized = re.sub(pattern, replacement, sanitized, flags=re.IGNORECASE)
        return sanitized
    
    def check_pii(self, text: str) -> List[str]:
        """Check for PII in text."""
        found = []
        for pattern, label in self.sensitive_patterns:
            if re.search(pattern, text, re.IGNORECASE):
                found.append(label)
        return found
```

### Step 4: Security Monitoring
```python
import logging
from typing import Dict, Any

class SecurityMonitor:
    """Monitor agent behavior for security issues."""
    
    def __init__(self):
        self.logger = logging.getLogger("security")
        self.violations: Dict[str, int] = {}
    
    def log_request(self, user_id: str, input_text: str, validated: bool):
        """Log incoming request."""
        if not validated:
            self.violations[user_id] = self.violations.get(user_id, 0) + 1
            self.logger.warning(f"Invalid request from {user_id}: {input_text[:100]}")
    
    def log_violation(self, user_id: str, violation_type: str, details: str):
        """Log security violation."""
        self.violations[user_id] = self.violations.get(user_id, 0) + 1
        self.logger.error(
            f"Security violation by {user_id}: {violation_type} - {details}"
        )
    
    def get_violation_count(self, user_id: str) -> int:
        """Get violation count for user."""
        return self.violations.get(user_id, 0)
    
    def should_block(self, user_id: str, threshold: int = 10) -> bool:
        """Check if user should be blocked."""
        return self.violations.get(user_id, 0) >= threshold
```

## Exercises
1. **Basic**: Create prompt injection detector
2. **Intermediate**: Implement tool usage auditing
3. **Advanced**: Add data leakage prevention

## Next Steps
- Lab 9: Evaluation and robustness testing
- Apply security to production agents
