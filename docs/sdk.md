Published Date: 2026-24-01 24th January 2026

Title: Embedding Agentic Workflows with the GitHub Copilot SDK

Hero image: GitHubSDK.png


# Deep Dive: Embedding Agentic Workflows with the GitHub Copilot SDK

The recent technical preview of the GitHub Copilot SDK opens up powerful new possibilities for building intelligent, agent-driven workflows directly into your own applications.

I've been experimenting with it, and this is my full walkthrough. If you’ve ever built a prototype that tries to orchestrate LLMs, making them run commands, update files, or use custom tools, you know that most of your energy goes to scaffolding the agent: managing turns, context, permissions, and tool routing. Most of us only want to focus on our actual product logic, not reinvent the execution loop.

With the Copilot SDK, you can now tap into the agentic core from Copilot CLI, including planning, tool-chaining, execution, and also streaming those results to users, all as a programmable base layer.

---

## What Is the GitHub Copilot SDK?

Think of the Copilot SDK as a way to embed Copilot’s "brain," the same agentic execution loop used inside GitHub CLI, directly into your own software. This loop handles things like:
- Gathering and maintaining context across conversations or sessions
- Planning multi-step tasks (break a prompt into steps, run those steps, adapt as needed)
- Invoking both built-in and custom tools or APIs as the agent "thinks"
- Delegating between different LLMs or models
- Streaming results back to your app

It’s production-grade and abstracted to be language-agnostic, so you can use TypeScript, Python, Go, or .NET. You can bring your own Copilot key or use an existing Copilot subscription.

---

## Getting Started

### Installation

For this guide, I’ll focus on TypeScript/Node.js and Python, since those are the most approachable for quick prototypes.

#### TypeScript

```bash
npm install @github/copilot-sdk
```

#### Python

```bash
pip install github-copilot-sdk
```

> **Fact:** The SDK uses the same APIs internally as GitHub Copilot CLI, so if you’ve used the CLI, you’re already familiar with how sessions, models, and tools are managed.

---

## Your First Agent: Hello World with Copilot SDK

Let’s start with the most basic example: spinning up a Copilot session and sending a prompt.

### TypeScript Example

```typescript
// Import the CopilotClient from the npm package
import { CopilotClient } from "@github/copilot-sdk";

// Initialize the client (this establishes connection, handles auth, etc)
const client = new CopilotClient();
await client.start();

// Create a new session. You can specify model version here (for example, "gpt-5" or any supported model)
const session = await client.createSession({ model: "gpt-5" });

// Send a prompt to the agentic loop
const response = await session.send({ prompt: "Hello, world!" });

console.log(response.text); // Output from Copilot's model
```

**Why is this interesting?**  
Unlike a basic LLM API call, this session lives beyond a single input/output, preserving context and allowing multi-turn dialogues. The SDK's agentic core manages the history for you, which is super useful for conversational or multi-step tasks.

---

### Python Example

```python
# Import the Copilot SDK client (syntax might change slightly between preview and GA)
from github_copilot_sdk import CopilotClient

# Initialize client (authentication details handled internally or via environment)
client = CopilotClient()
client.start()

# Create a session with your chosen AI model
session = client.create_session(model="gpt-5")

# Send a prompt, get a response
response = session.send(prompt="Hello, world!")

print(response.text)  # See what the agent responds with
```

**Note:** The real power shows up when you add your own tools and let the agent plan actions that extend beyond text completion.

---

## Going Further: Adding Custom Tools and Actions

One powerful feature of the Copilot SDK is that you aren’t just limited to text prompts. You can register **custom tools**, functions or API endpoints that the Copilot agent can invoke while reasoning.

Let’s see how you can register a tool and make Copilot use it.

### TypeScript: Register a Custom Tool

Suppose you want Copilot to be able to fetch the weather by calling your own function.

```typescript
// Define your custom tool as a function
async function getWeather(args: { city: string }) {
    // In a real-world case, you’d call a weather API here
    return `It is always sunny in ${args.city}`;
}

// Register your tool with the Copilot agent
session.registerTool({
    name: 'getWeather',
    description: 'Gets the weather for a given city',
    execute: getWeather
});

// Now you can give Copilot a task where using this tool is helpful.
const response = await session.send({
    prompt: "What's the weather in Paris?"
});

console.log(response.text);
```

**How does this work?**  
Copilot’s agentic loop analyzes your prompt and decides if it should ask you to use a custom tool. If your agent is allowed, it will call your tool, integrate the result into its response, and continue reasoning.

---

## Streaming, Context, and Multi-Turn Interactions

Another standout is Copilot SDK’s support for streaming. This means you can get tokens or tool actions as they happen, making it easy to provide real-time updates in your UI.

You also get **persistent sessions**: the context sticks around between calls and you can interactively build up complex tasks.

**Interesting fact:**  
Under the hood, Copilot’s session management uses intelligent memory compaction so you won’t run into token or context limits on longer conversations.

---

## Integrating with Your Authentication and MCP (Model Coordination Platform)

The SDK handles user authentication, permissions, and also MCP server integration for you. If you run teams or need to delegate model execution to an enterprise setup, these features make the SDK enterprise-ready right out of the box.

**Sample:**  
Minimal config is needed. By default, the SDK checks for your Copilot key or GitHub login. You can also pass keys directly for local testing.

---

## What Kind of Workflows Can You Build?

Here’s what I’ve already started sketching out:
- A desktop app where you can run natural-language commands that edit files, call APIs, or trigger builds
- A bot that can generate YouTube chapters using custom parsing tools plus Copilot’s reasoning
- An internal tool for our team that lets us brainstorm, plan, and execute ideas, with steps handled by custom agentic tools plus Copilot’s model switching

And with support for Python, you can build plugins or automations for Jupyter, VS Code, or even server-side agents.

---

## Final Thoughts: When Should You Reach For Copilot SDK?

If you want to build apps where LLMs don’t just answer but actually act—plan, loop, remember, delegate, and use tools—this SDK is the launchpad.

Instead of rebuilding orchestration, memory, and tool wiring every time, you plug in to a robust, battle-tested base and focus on adding domain knowledge or unique tools on top.

---

## Further Reading

- Copilot SDK repo and docs: [github/copilot-sdk](https://github.com/github/copilot-sdk)
- Read the [official announcement blog](https://github.blog/changelog/) and examples
- Try the Copilot CLI for terminal-first agentic workflows

---

**Summary:**  
The Copilot SDK is a huge leap for quickly bootstrapping intelligent, agentic apps, whether you want to automate tedious workflows, build custom chat and command interfaces, or just see how far you can push “Copilot as a programmable AI”. If you build something cool or weird using it, let me know. I’m still exploring too!

---

*Written by someone who spent too much of the weekend making Copilot do all the things* 😄