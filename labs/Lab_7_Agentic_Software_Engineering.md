
# Lab 7: Agentic AI for Software Engineering (Pro Version)

## Objective
Create an agent that reads code, proposes a patch, runs tests, and opens a PR—demonstrating agentic augmentation of the SDLC. This lab will guide you through:
- Reading and analyzing code
- Proposing and applying a patch
- Running tests and reporting results
- Simulating a PR

---

## Step 1: Choose a Codebase

For demonstration, we'll use a simple Python function and test.

```python
# buggy_code.py
def add(a, b):
  # Bug: should be a + b, but returns a - b
  return a - b
```

```python
# test_buggy_code.py
def test_add():
  assert add(2, 3) == 5
  assert add(-1, 1) == 0

if __name__ == "__main__":
  test_add()
  print("All tests passed.")
```

---

## Step 2: Agent Reads and Proposes Patch

```python
# Agent reads code and proposes a patch
def propose_patch():
  # Simulate LLM reasoning
  print("[AGENT] Reading buggy_code.py...")
  print("[AGENT] Detected bug: should use + instead of -")
  patch = "def add(a, b):\n    return a + b\n"
  print("[AGENT] Proposed patch:\n", patch)
  return patch
```

---

## Step 3: Apply Patch and Run Tests

```python
def apply_patch_and_test(patch):
  # Write the patch to file
  with open("buggy_code.py", "w") as f:
    f.write(patch)
  print("[AGENT] Patch applied. Running tests...")
  # Run tests
  try:
    import subprocess
    result = subprocess.run(["python", "test_buggy_code.py"], capture_output=True, text=True)
    print("[AGENT] Test output:\n", result.stdout)
    if result.returncode == 0:
      print("[AGENT] All tests passed. Ready to open PR.")
    else:
      print("[AGENT] Tests failed.")
  except Exception as e:
    print("[AGENT] Test run error:", e)
```

---

## Step 4: Simulate Opening a PR

```python
def open_pr():
  # Simulate PR creation
  print("[AGENT] Opening pull request for patch to add() function...")
  print("[AGENT] PR Title: Fix add() bug\nPR Body: This patch fixes the add() function to use addition instead of subtraction.")
```

---

## Step 5: Run the Workflow

```python
if __name__ == "__main__":
  patch = propose_patch()
  apply_patch_and_test(patch)
  open_pr()
```

---

## Sample Output
```
[AGENT] Reading buggy_code.py...
[AGENT] Detected bug: should use + instead of -
[AGENT] Proposed patch:
 def add(a, b):
  return a + b

[AGENT] Patch applied. Running tests...
[AGENT] Test output:
 All tests passed.
[AGENT] All tests passed. Ready to open PR.
[AGENT] Opening pull request for patch to add() function...
[AGENT] PR Title: Fix add() bug
PR Body: This patch fixes the add() function to use addition instead of subtraction.
```

---

## Explanation
- **Agent Reasoning:** Reads code, detects bug, proposes patch.
- **Patch Application:** Applies patch and runs tests.
- **PR Simulation:** Simulates opening a pull request.
- **Logs:** Show agent reasoning and test results.

---

## Pro Tips
- Integrate with real VCS and CI for full automation.
- Use LLMs for more advanced code analysis and patching.
- Log all steps for traceability and debugging.

---

**Proceed to Lab 8 when you have a working agentic SDLC workflow and logs.**
