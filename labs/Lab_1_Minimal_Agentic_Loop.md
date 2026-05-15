
# Lab 1: Minimal Agentic Loop (Pro Version)

## Objective
Build a robust agentic loop that plans, acts, observes, and adapts to achieve a user-defined goal using an LLM and a web search API. This lab will guide you through:
- Writing a professional-grade agent script
- Integrating with OpenAI (or Anthropic) API and a web search API (e.g., SerpAPI)
- Logging, error handling, and adaptive querying
- Delivering a concise, actionable summary

---

## Step 1: Environment Setup

**Requirements:**

**Install dependencies:**
```bash
pip install openai requests python-dotenv
```

Create a `.env` file with your API keys:
```
OPENAI_API_KEY=sk-...
SERPAPI_API_KEY=...
```


## Step 2: Agentic Loop Script

Below is a modular, well-commented Python implementation of the agentic loop. Each function is documented and includes inline comments for clarity and maintainability.


```python
# Import required libraries and load environment variables
import os
import time
import requests
from dotenv import load_dotenv
import openai
from datetime import datetime

# Load .env file for API keys
load_dotenv()
openai.api_key = os.getenv("OPENAI_API_KEY")
SERPAPI_KEY = os.getenv("SERPAPI_API_KEY")
```

```python
# Log messages with a timestamp for traceability
def log(msg):
	print(f"[{datetime.now().isoformat()}] {msg}")
```

```python
# Send a prompt to the LLM and return the response
def llm_complete(prompt):
	response = openai.ChatCompletion.create(
		model="gpt-3.5-turbo",
		messages=[{"role": "user", "content": prompt}],
		max_tokens=256,
	)
	return response.choices[0].message.content.strip()
```

```python
# Query the SerpAPI web search API and return results
def web_search(query):
	url = f"https://serpapi.com/search.json?q={query}&api_key={SERPAPI_KEY}"
	resp = requests.get(url)
	resp.raise_for_status()
	results = resp.json().get("organic_results", [])
	return results
```

```python
# Ask the LLM to plan steps for the given goal
def plan_steps(goal):
	return llm_complete(f"Plan steps to achieve: {goal}")
```

```python
# Ask the LLM to generate a search query based on the goal, plan, and context
def generate_search_query(goal, plan, context):
	return llm_complete(f"Given the goal '{goal}' and plan '{plan}', what is the best web search query? {context}")
```

```python
# Ask the LLM to summarize the web search results for the goal
def summarize_results(goal, results):
	return llm_complete(f"Summarize these results for the goal '{goal}': {results}")
```

```python
# Ask the LLM if the summary fully answers the goal
def check_satisfaction(goal, summary):
	return llm_complete(f"Does this summary fully answer the goal '{goal}'? Reply yes or no.")
```

```python
# Main agentic loop: plan, act, observe, adapt until goal is achieved or retries exhausted
def agentic_loop(goal, max_retries=3):
	log(f"Goal: {goal}")
	plan = plan_steps(goal)
	log(f"Plan: {plan}")
	attempts = 0
	context = ""
	summary = ""
	while attempts < max_retries:
		query = generate_search_query(goal, plan, context)
		log(f"Search Query: {query}")
		try:
			results = web_search(query)
			log(f"Found {len(results)} results.")
			summary = summarize_results(goal, results)
			log(f"Summary: {summary}")
			satisfied = check_satisfaction(goal, summary)
			if 'yes' in satisfied.lower():
				log("Goal achieved.")
				print("\nFinal Summary:\n", summary)
				return summary
			else:
				context = f"Previous summary: {summary}. Results were insufficient."
				log("Adapting and retrying...")
		except Exception as e:
			log(f"Error: {e}")
			context = f"Error encountered: {e}"
		attempts += 1
		time.sleep(1)
	log("Max retries reached. Returning best effort summary.")
	print("\nBest Effort Summary:\n", summary)
	return summary
```

```python
# Entry point for running the agentic loop from the command line
if __name__ == "__main__":
	import sys
	goal = sys.argv[1] if len(sys.argv) > 1 else "Summarize the latest AI news"
	agentic_loop(goal)
```

---

**How to run:**
```bash
python agentic_loop.py "Summarize the latest AI news"
```

---

**Explanation:**
- Each function is modular and documented for clarity.
- Inline comments explain the logic and flow.
- The main loop plans, acts, observes, and adapts until the goal is met or retries are exhausted.

---

## Step 3: Sample Run

```bash
python agentic_loop.py "Summarize the latest AI news"
```

**Sample Output:**
```
[2026-05-15T12:00:00] Goal: Summarize the latest AI news
[2026-05-15T12:00:00] Plan: 1. Search for latest AI news... 2. Summarize findings.
[2026-05-15T12:00:01] Search Query: latest AI news May 2026
[2026-05-15T12:00:02] Found 10 results.
[2026-05-15T12:00:03] Summary: The latest AI news includes...
[2026-05-15T12:00:03] Goal achieved.

Final Summary:
The latest AI news includes...
```

---

## Explanation
- **Agentic Loop:** The script plans, acts (searches), observes (analyzes results), and adapts (retries with new queries if needed).
- **Logging:** Each step is timestamped for traceability.
- **Error Handling:** Graceful retries and error logs.
- **Extensible:** Swap in Anthropic or other LLMs/APIs as needed.

---

## Pro Tips
- Tune prompts for better search queries and summaries.
- Increase `max_retries` for more robust adaptation.
- Log to a file for audit trails in production.
- Modularize code for larger agentic systems.

---

**Proceed to Lab 2 when you have a working agentic loop and transcript.**
