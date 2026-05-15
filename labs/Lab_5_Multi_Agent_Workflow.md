
# Lab 5: Compose a Multi-Agent Workflow (Pro Version)

## Objective
Design and implement a supervisor/worker or debate-style multi-agent system to solve a decomposable problem. This lab will guide you through:
- Defining a decomposable problem
- Implementing multiple agent roles
- Designing communication and coordination
- Demonstrating a sample workflow

---

## Step 1: Define the Problem

For this example, we'll use a **document review** workflow:
- Supervisor agent delegates sections to worker agents
- Workers summarize sections
- Supervisor compiles and critiques results

---

## Step 2: Define Agent Roles

```python
# Base agent class
class Agent:
	def __init__(self, name):
		self.name = name

	def send(self, message, recipient):
		# Send a message to another agent
		print(f"[{self.name} -> {recipient.name}]: {message}")
		return recipient.receive(message, self)

	def receive(self, message, sender):
		# Default receive just logs the message
		print(f"[{self.name} received from {sender.name}]: {message}")
		return None
```

```python
# Worker agent: summarizes a section
class WorkerAgent(Agent):
	def receive(self, message, sender):
		# Simulate summarization
		summary = f"Summary of '{message}' by {self.name}"
		print(f"[{self.name}]: {summary}")
		return summary
```

```python
# Supervisor agent: delegates and compiles
class SupervisorAgent(Agent):
	def __init__(self, name, workers):
		super().__init__(name)
		self.workers = workers

	def review_document(self, sections):
		# Delegate each section to a worker
		summaries = []
		for i, section in enumerate(sections):
			worker = self.workers[i % len(self.workers)]
			print(f"[Supervisor]: Assigning section '{section}' to {worker.name}")
			summary = self.send(section, worker)
			summaries.append(summary)
		# Compile and critique
		compiled = "\n".join(summaries)
		critique = self.critique(compiled)
		print(f"[Supervisor]: Final critique: {critique}")
		return compiled, critique

	def critique(self, compiled):
		# Simulate critique
		return f"Critique of summaries: All sections covered."
```

---

## Step 3: Run the Workflow

```python
if __name__ == "__main__":
	# Define agents
	worker1 = WorkerAgent("Worker1")
	worker2 = WorkerAgent("Worker2")
	supervisor = SupervisorAgent("Supervisor", [worker1, worker2])

	# Sample document split into sections
	sections = ["Section 1: Intro to AI", "Section 2: Applications", "Section 3: Challenges"]

	# Run the workflow
	compiled, critique = supervisor.review_document(sections)

	print("\nCompiled Summaries:\n", compiled)
	print("\nCritique:\n", critique)
```

---

## Sample Output
```
[Supervisor]: Assigning section 'Section 1: Intro to AI' to Worker1
[Supervisor -> Worker1]: Section 1: Intro to AI
[Worker1 received from Supervisor]: Section 1: Intro to AI
[Worker1]: Summary of 'Section 1: Intro to AI' by Worker1
[Supervisor]: Assigning section 'Section 2: Applications' to Worker2
[Supervisor -> Worker2]: Section 2: Applications
[Worker2 received from Supervisor]: Section 2: Applications
[Worker2]: Summary of 'Section 2: Applications' by Worker2
[Supervisor]: Assigning section 'Section 3: Challenges' to Worker1
[Supervisor -> Worker1]: Section 3: Challenges
[Worker1 received from Supervisor]: Section 3: Challenges
[Worker1]: Summary of 'Section 3: Challenges' by Worker1
[Supervisor]: Final critique: Critique of summaries: All sections covered.

Compiled Summaries:
 Summary of 'Section 1: Intro to AI' by Worker1
Summary of 'Section 2: Applications' by Worker2
Summary of 'Section 3: Challenges' by Worker1

Critique:
 Critique of summaries: All sections covered.
```

---

## Explanation
- **Supervisor/Worker:** Supervisor delegates, workers summarize, supervisor critiques.
- **Communication:** Agents send/receive messages and log interactions.
- **Extensible:** Add more roles (e.g., critic, planner) or more complex protocols.

---

## Pro Tips
- Use message logs for debugging and analysis.
- Extend agent logic for real LLM calls or more advanced workflows.
- Try a debate or planner/critic pattern for other problems.

---

**Proceed to Lab 6 when you have a working multi-agent workflow and run log.**
