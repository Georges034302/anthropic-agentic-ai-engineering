
# Lab 8: Productionize an Agentic System (Pro Version)

## Objective
Deploy an agentic system with observability, logging, and policy enforcement; simulate production scenarios (error handling, audit, scaling). This lab will guide you through:
- Adding observability (logging, tracing, metrics)
- Implementing policy enforcement
- Simulating production scenarios
- Documenting best practices

---

## Step 1: Add Observability

```python
# Simple logger with tracing and metrics
import time
class Logger:
  def __init__(self):
    self.metrics = {}

  def log(self, msg):
    print(f"[LOG] {msg}")

  def trace(self, span):
    print(f"[TRACE] {span}")

  def metric(self, name, value):
    self.metrics[name] = value
    print(f"[METRIC] {name}: {value}")
```

---

## Step 2: Policy Enforcement

```python
# Example policy: access control and audit logging
class PolicyEnforcer:
  def __init__(self, logger):
    self.logger = logger
    self.allowed_users = {"alice", "bob"}

  def check_access(self, user):
    if user not in self.allowed_users:
      self.logger.log(f"Access denied for user: {user}")
      raise PermissionError(f"User {user} not allowed.")
    self.logger.log(f"Access granted for user: {user}")

  def audit(self, action, user):
    self.logger.log(f"AUDIT: {user} performed {action}")
```

---

## Step 3: Simulate Production Scenarios

```python
import random
class Agent:
  def __init__(self, name, logger, policy):
    self.name = name
    self.logger = logger
    self.policy = policy

  def act(self, user, action):
    self.logger.trace(f"Agent {self.name} starting action: {action}")
    try:
      self.policy.check_access(user)
      # Simulate error injection
      if action == "fail" or random.random() < 0.2:
        raise RuntimeError("Simulated error!")
      self.policy.audit(action, user)
      self.logger.log(f"Agent {self.name} completed action: {action}")
      self.logger.metric(f"{self.name}_success", 1)
    except Exception as e:
      self.logger.log(f"Agent {self.name} error: {e}")
      self.logger.metric(f"{self.name}_failure", 1)
```

---

## Step 4: Scaling and Audit

```python
if __name__ == "__main__":
  logger = Logger()
  policy = PolicyEnforcer(logger)
  agents = [Agent(f"Agent{i}", logger, policy) for i in range(3)]
  users = ["alice", "bob", "eve"]
  actions = ["process", "audit", "fail"]

  for agent in agents:
    for user in users:
      for action in actions:
        agent.act(user, action)

  print("\nMetrics:", logger.metrics)
```

---

## Sample Output
```
[TRACE] Agent Agent0 starting action: process
[LOG] Access granted for user: alice
[LOG] AUDIT: alice performed process
[LOG] Agent Agent0 completed action: process
[METRIC] Agent0_success: 1
[TRACE] Agent Agent0 starting action: audit
[LOG] Access granted for user: alice
[LOG] AUDIT: alice performed audit
[LOG] Agent Agent0 completed action: audit
[METRIC] Agent0_success: 1
[TRACE] Agent Agent0 starting action: fail
[LOG] Access granted for user: alice
[LOG] Agent Agent0 error: Simulated error!
[METRIC] Agent0_failure: 1
... (output truncated)

Metrics: {'Agent0_success': 1, 'Agent0_failure': 1, ...}
```

---

## Explanation
- **Observability:** Logs, traces, and metrics for all actions.
- **Policy Enforcement:** Access control and audit logging.
- **Error Handling:** Simulates errors and logs failures.
- **Scaling:** Multiple agents and users.

---

## Pro Tips
- Integrate with real logging/metrics systems (e.g., Prometheus, ELK).
- Expand policy logic for real compliance.
- Use error injection to test robustness.

---

**Congratulations! You have completed the Agentic AI Engineering labs.**
