
# Lab 4: Build a Stateful, Guardrailed Agent (Pro Version)

## Objective
Implement an agent with memory (short/long-term), a tool registry, and robust guardrails (validation, policy enforcement, error recovery). This lab will guide you through:
- Adding memory to your agent
- Registering and selecting among multiple tools
- Implementing guardrails for validation, policy, and error handling
- Demonstrating a multi-step task

---

## Step 1: Define Tools

```python
# Example tools: calculator and echo
def add(a, b):
  # Add two numbers
  return a + b

def echo(text):
  # Echo back the input text
  return text
```

---

## Step 2: Implement Memory

```python
# Simple in-memory storage for agent actions and results
class AgentMemory:
  def __init__(self):
    self.history = []  # Stores (action, input, result)

  def remember(self, action, input_data, result):
    self.history.append({
      'action': action,
      'input': input_data,
      'result': result
    })

  def recall(self):
    return self.history
```

---

## Step 3: Tool Registry

```python
# Register and select tools by name
class ToolRegistry:
  def __init__(self):
    self.tools = {}

  def register(self, name, func):
    self.tools[name] = func

  def get(self, name):
    return self.tools.get(name)
```

---

## Step 4: Guardrails (Validation, Policy, Error Handling)

```python
# Guardrail functions for validation and policy enforcement
def validate_input(tool_name, input_data):
  # Example: Only allow numbers for 'add', non-empty string for 'echo'
  if tool_name == 'add':
    a, b = input_data.get('a'), input_data.get('b')
    if not (isinstance(a, (int, float)) and isinstance(b, (int, float))):
      raise ValueError('Inputs to add must be numbers.')
  elif tool_name == 'echo':
    if not input_data.get('text'):
      raise ValueError('Input to echo must be non-empty.')

def policy_check(tool_name):
  # Example: Disallow 'echo' tool for certain policies
  if tool_name == 'echo':
    raise PermissionError('Echo tool is disabled by policy.')
```

---

## Step 5: Agent Implementation

```python
# Agent with memory, tool registry, and guardrails
class GuardrailedAgent:
  def __init__(self, registry, memory):
    self.registry = registry
    self.memory = memory

  def act(self, tool_name, input_data):
    try:
      validate_input(tool_name, input_data)
      # Uncomment to enforce policy: policy_check(tool_name)
      tool = self.registry.get(tool_name)
      if not tool:
        raise ValueError(f'Tool {tool_name} not found.')
      result = tool(**input_data)
      self.memory.remember(tool_name, input_data, result)
      print(f"[LOG] {tool_name}({input_data}) => {result}")
      return result
    except Exception as e:
      print(f"[GUARDRAIL] {tool_name} failed: {e}")
      self.memory.remember(tool_name, input_data, f"ERROR: {e}")
      return None
```

---

## Step 6: Demonstrate Multi-Step Task

```python
if __name__ == "__main__":
  # Setup
  registry = ToolRegistry()
  registry.register('add', add)
  registry.register('echo', echo)
  memory = AgentMemory()
  agent = GuardrailedAgent(registry, memory)

  # Multi-step task
  agent.act('add', {'a': 2, 'b': 3})
  agent.act('echo', {'text': 'Hello, world!'})
  agent.act('add', {'a': 'bad', 'b': 5})  # Should trigger validation error
  # Uncomment to test policy: agent.act('echo', {'text': 'Blocked'})

  # Show memory usage
  print("\nAgent Memory Log:")
  for entry in memory.recall():
    print(entry)
```

---

## Sample Output
```
[LOG] add({'a': 2, 'b': 3}) => 5
[LOG] echo({'text': 'Hello, world!'}) => Hello, world!
[GUARDRAIL] add failed: Inputs to add must be numbers.

Agent Memory Log:
{'action': 'add', 'input': {'a': 2, 'b': 3}, 'result': 5}
{'action': 'echo', 'input': {'text': 'Hello, world!'}, 'result': 'Hello, world!'}
{'action': 'add', 'input': {'a': 'bad', 'b': 5}, 'result': 'ERROR: Inputs to add must be numbers.'}
```

---

## Explanation
- **Memory:** Tracks all actions and results for traceability.
- **Tool Registry:** Allows dynamic tool selection and extension.
- **Guardrails:** Enforce input validation, policy, and error handling.
- **Logs:** Show memory usage and guardrail enforcement in action.

---

## Pro Tips
- Extend memory for long-term storage (e.g., file, database).
- Add more tools and guardrail logic for real-world scenarios.
- Use logs for debugging and compliance.

---

**Proceed to Lab 5 when you have a working stateful, guardrailed agent and run log.**
