# Building a Multi-Agent Pipeline with Microsoft Agent Framework

A hands-on guide to building a **sequential multi-agent orchestration** using the Microsoft Agent Framework SDK. You'll build three agents that work together to process customer feedback: summarizing it, classifying it, and recommending a next step.

⏱️ **Estimated time:** ~30 minutes

> ⚠️ **Note:** Some technologies used here are in preview / active development. Expect occasional quirks, warnings, or errors.

---

## What You'll Build

| Agent | Role |
|---|---|
| **Summarizer** | Condenses raw feedback into a short, neutral sentence |
| **Classifier** | Categorizes feedback as `Positive`, `Negative`, or `Feature request` |
| **Recommended Action** | Suggests an appropriate follow-up step |

These three agents run in sequence, each passing its output to the next, using the SDK's `SequentialBuilder`.

---

## Prerequisites

- [Visual Studio Code](https://code.visualstudio.com/)
- An active **Azure subscription**
- **Python 3.13+** (tested successfully on 3.13.12 — some dependencies may lag behind the latest 3.13 release)
- [Git](https://git-scm.com/)

---

## Step 1: Set Up a Foundry Project in VS Code

1. Open VS Code → **Extensions** (`Ctrl+Shift+X`).
2. Search for and install **Foundry Toolkit** (published by Microsoft).
   > It may still be labeled **AI Toolkit** in some places — same extension.
3. Open the Foundry Toolkit icon in the sidebar and sign in to Azure if prompted.
4. Under **Microsoft Foundry Resources**, select **Create Project**.
5. Choose:
   - Resource group: `ResourceGroup1`
   - Project name: `ai-agents-project63228924`
6. Wait for deployment — the new project will appear as the default project in the Foundry Toolkit pane.

---

## Step 2: Deploy a Model

1. When the success popup appears, click **Deploy a new model** (or press `F1` → `Foundry Toolkit: Show model catalog`).
2. Search for and select **gpt-5** → **Deploy**.
3. Configure:
   - Deployment name: `gpt-5`
   - Deployment type: `Global Standard` (fallback: `Standard`)
   - Leave model version and tokens/minute as default
4. Click **Deploy to Microsoft Foundry** and wait for completion.
5. Right-click the deployment → **Copy Project Endpoint**.
   > Can't find it in VS Code? Grab it from the **Microsoft Foundry project endpoint** field at [ai.azure.com](https://ai.azure.com/).

---

## Step 3: Clone the Starter Code

```bash
# Command Palette → Git: Clone
https://github.com/MicrosoftLearning/mslearn-ai-agents.git
```

1. Open the cloned repo in VS Code.
2. **File > Open Folder** → navigate to:
   ```
   mslearn-ai-agents/Labfiles/08-agent-orchestration
   ```
3. Expand the `Python` folder to see the lab files.
4. Right-click `requirements.txt` → **Open in Integrated Terminal**, then run:

```powershell
python -m venv labenv
.\labenv\Scripts\Activate.ps1
pip install -r requirements.txt
```

5. Open the `.env` file and update:
   - `your_project_endpoint` → your copied project endpoint
   - `MODEL_DEPLOYMENT_NAME` → your model deployment name (`gpt-5`)
   - Save with `Ctrl+S`

---

## Step 4: Build the Agents

Open `agents.py` and fill in each section.

### 4.1 Add references

```python
# Add references
from agent_framework import Message
from agent_framework.foundry import FoundryChatClient
from agent_framework.orchestrations import SequentialBuilder
from azure.identity import AzureCliCredential
```

### 4.2 Create the chat client

```python
# Create the chat client
credential = AzureCliCredential()
chat_client = FoundryChatClient(
   credential=credential,
   project_endpoint=os.getenv("AZURE_AI_PROJECT_ENDPOINT"),
   model=os.getenv("AZURE_AI_MODEL_DEPLOYMENT_NAME"),
)
```

`AzureCliCredential` authenticates with your Azure account; `FoundryChatClient` connects to your project using the `.env` config.

### 4.3 Create the three agents

```python
# Create agents
summarizer_agent = chat_client.as_agent(
   name="summarizer",
   instructions=summarizer_instructions,
)

classifier_agent = chat_client.as_agent(
   name="classifier",
   instructions=classifier_instructions,
)

action_agent = chat_client.as_agent(
   name="action",
   instructions=action_instructions,
)
```

---

## Step 5: Build the Sequential Orchestration

### 5.1 Set the sample feedback

```python
# Initialize the current feedback
feedback="""
I use the dashboard every day to monitor metrics, and it works well overall. 
But when I'm working late at night, the bright screen is really harsh on my eyes. 
If you added a dark mode option, it would make the experience much more comfortable.
"""
```

### 5.2 Chain the agents together

```python
# Build sequential orchestration
workflow = SequentialBuilder(
   participants=[summarizer_agent, classifier_agent, action_agent],
   output_from="all",
).build()
```

Agents run in the order listed. `output_from="all"` collects output from every agent, not just the last one.

### 5.3 Run the workflow

```python
# Run and collect outputs
result = await workflow.run(f"Customer feedback: {feedback}")
outputs = result.get_outputs()
```

### 5.4 Display the results

```python
# Display outputs
i = 1
for response in outputs:
   for msg in cast(list[Message], response.messages):
       name = msg.author_name or ("assistant" if msg.role == "assistant" else "user")
       print(f"{'-' * 60}\n{i:02d} [{name}]\n{msg.text}")
       i += 1
```

Save the file (`Ctrl+S`).

---

## Step 6: Run and Test

```bash
az login
python agents.py
```

**Expected output:**

```
User requests a dark mode option for more comfortable nighttime use.
Feature request
Log as enhancement request to add dark mode for improved user comfort during nighttime use.
------------------------------------------------------------
01 [summarizer]
User requests a dark mode option for more comfortable nighttime use.
------------------------------------------------------------
02 [classifier]
Feature request
------------------------------------------------------------
03 [action]
Log as enhancement request to add dark mode for improved user comfort during nighttime use.
```

💡 **Try it yourself** — swap in different feedback, e.g.:

> "I reached out to your customer support yesterday because I couldn't access my account. The representative responded almost immediately, was polite and professional, and fixed the issue within minutes. Honestly, it was one of the best support experiences I've ever had."

When done testing, exit the virtual environment:

```bash
deactivate
```

---

## Cleanup

To avoid unnecessary Azure charges:

### Delete the model
1. Refresh the **Azure Resources** view in VS Code.
2. Expand **Models**.
3. Right-click your deployed model → **Delete**.

### Delete the resource group
1. Open the [Azure Portal](https://portal.azure.com/).
2. Navigate to your resource group.
3. Select **Delete resource group** and confirm.

> ✅ Don't forget to officially **end the lab** session if applicable.

---

## Summary

| Step | What Happens |
|---|---|
| 1–2 | Set up a Foundry project & deploy a GPT-5 model |
| 3 | Clone starter code and configure environment |
| 4 | Define three agents (summarizer, classifier, action) |
| 5 | Chain them with `SequentialBuilder` and run the workflow |
| 6 | Test with sample feedback, then clean up resources |

This pattern — chaining specialized agents sequentially — is a foundational building block for more complex multi-agent systems (e.g., adding parallel branches, conditional routing, or human-in-the-loop review steps).
