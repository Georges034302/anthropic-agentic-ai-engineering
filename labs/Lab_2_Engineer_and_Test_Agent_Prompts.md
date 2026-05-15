
# Lab 2: Engineer and Test Agent Prompts (Pro Version)

## Objective
Design, structure, and iteratively test prompts for an agent to reliably perform multi-step tasks, including tool use and context injection. This lab will guide you through:
- Creating robust prompt templates
- Specifying output formats and tool instructions
- Testing and refining prompts with code
- Analyzing prompt effectiveness

---

## Step 1: Choose an Agent Task
For this lab, we'll use **summarization with tool use** as the example task. You can adapt the workflow for other tasks (e.g., data extraction, code review).

---

## Step 2: Draft Initial Prompt Templates

### Example: Summarization with Tool Use

#### Initial Prompt (Before Iteration)
```text
You are an AI assistant. Summarize the following article.

Article:
{{article_text}}

Summarize in 3-5 bullet points.
```

#### Improved Prompt (After Iteration)
```text
You are an expert research assistant. Your task is to:
1. Read the provided article.
2. Summarize the main points in 3-5 concise, factual bullet points.
3. If you need additional context, use the [search_tool] to look up recent related news.

Instructions:
- Use only information from the article and search results.
- Format output as:

Summary:
- Bullet 1
- Bullet 2
- ...

If you use the [search_tool], cite the source in parentheses.

Article:
{{article_text}}

Context:
{{context_block}}
```

---

## Step 3: Test Prompts with LLM

Use the following Python code to test your prompts with OpenAI or Anthropic APIs. Save as `test_prompts.py`.

```python
import os
import openai
from dotenv import load_dotenv

load_dotenv()
openai.api_key = os.getenv("OPENAI_API_KEY")

def test_prompt(prompt, variables):
  # Fill in template variables
  prompt_filled = prompt
  for k, v in variables.items():
    prompt_filled = prompt_filled.replace(f"{{{{{k}}}}}", v)
  # Send to LLM
  response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[{"role": "user", "content": prompt_filled}],
    max_tokens=256,
  )
  return response.choices[0].message.content.strip()

if __name__ == "__main__":
  article = "OpenAI released a new model. It is faster and more accurate."
  context = "No additional context."
  prompt = """You are an AI assistant. Summarize the following article.\n\nArticle:\n{{article_text}}\n\nSummarize in 3-5 bullet points."""
  print("--- Before Iteration ---")
  print(test_prompt(prompt, {"article_text": article, "context_block": context}))

  improved_prompt = """You are an expert research assistant. Your task is to:\n1. Read the provided article.\n2. Summarize the main points in 3-5 concise, factual bullet points.\n3. If you need additional context, use the [search_tool] to look up recent related news.\n\nInstructions:\n- Use only information from the article and search results.\n- Format output as:\n\nSummary:\n- Bullet 1\n- Bullet 2\n- ...\n\nIf you use the [search_tool], cite the source in parentheses.\n\nArticle:\n{{article_text}}\n\nContext:\n{{context_block}}"""
  print("--- After Iteration ---")
  print(test_prompt(improved_prompt, {"article_text": article, "context_block": context}))
```

---

## Step 4: Analyze and Refine

- Compare outputs from before/after prompts.
- Note clarity, completeness, and format adherence.
- Refine prompts based on observed model behavior and repeat testing.

---

## Step 5: Deliverables

- At least two prompt templates (before/after)
- Test outputs (copy/paste from script)
- Brief analysis of what improved and why

---

## Pro Tips
- Always specify output format and tool use instructions.
- Use context blocks for additional info or constraints.
- Iterate based on real model outputs, not just intuition.

---

**Proceed to Lab 3 when you have prompt templates, test outputs, and your analysis.**
