# Building a CLI Coding Agent Notes 🛠️

## Concept: Agent that Writes Code

Goal: Create CLI assistant like **Claude Code/Cursor** that builds apps from prompts

---

## The Magic: OS Command Execution

### Basic Linux Commands
```bash
mkdir test              # Create folder
cd test                 # Navigate
touch index.html        # Create file
echo "test" > file.txt  # Write to file
```

**Key Insight:** If LLM can run these commands = it can code!

---

## Implementation

### 1. Create Run Command Tool
```python
import os

def run_command(command: str):
    result = os.system(command)
    return result
```

### 2. Add to Available Tools
```python
available_tools = {
    "get_weather": get_weather,
    "run_command": run_command  # NEW!
}
```

### 3. Update Prompt
```
run_command: Takes Linux command as string, 
executes on user's system, returns output
```

---

## Real Example 🎯

### Prompt Given:
```
"Create a todo app using HTML, CSS and JavaScript 
in a folder called todo_app with all CRUD operations"
```

### What Agent Did:
1. `mkdir todo_app` → Created folder
2. `touch index.html` → Created HTML file
3. `echo "code" > style.css` → Created CSS
4. `echo "code" > script.js` → Created JS

**Result:** Fully working todo app! ✅

---

## Follow-up Commands Work Too!

```
"Make overall theme purple and more modern"
→ Agent modified CSS automatically

"Add dark theme with purple as primary"
→ Agent updated files again

"Toggle theme button not working"
→ Agent debugged and fixed (until rate limit hit 😅)
```

---

## Problem: `run_command` Too Broad ⚠️

### Issues:
- Too autonomous
- No structure
- Hard to control

### Better Approach: Structured Tools

```python
def create_file(path: str, content: str):
    """Create a new file with content"""
    
def read_file(path: str):
    """Read file contents"""
    
def list_directory(path: str):
    """List files in directory"""
    
def delete_file(path: str):
    """Delete a file"""
    
def update_file(path: str, old_content: str, new_content: str):
    """Update specific content in file"""
```

---

## Mind-Blowing Feature 🤯

### Self-Improving Agent!

Prompt:
```
"In agent.py file, add more tools for file handling:
create_file, read_file, list_directory, delete_file, etc."
```

**Agent modifies its own code to add new tools!**

---

## Key Takeaways 💡

1. **CLI Agent = LLM + System Commands**
2. `os.system()` gives agent full system access
3. **Structured tools** > Single run_command
4. Agent can:
   - Create entire projects
   - Debug itself
   - Modify its own code
5. **Use with caution** - has full system access!

---

## What You Built 🏆

✅ Weather agent (API calls)  
✅ CLI coding agent (file operations)  
✅ Self-improving agent (modifies own code)

**Next:** RAG Systems (Retrieval Augmented Generation)