## AI Orchestration

This module introduces AI-powered workflow orchestration using Kestra. We learn how AI can accelerate workflow development and enable autonomous task automation through three techniques: AI Copilot for generating flows, RAG for grounding responses in real data, and AI Agents for autonomous task execution.

### Why AI for Workflows?
When building LLM applications and workflows, we often spend significant time writing boilerplate code, searching documentation, and structuring pipelines. AI tools can help us:

- Generate workflows faster: describe what you want in natural language instead of writing YAML from scratch
- Avoid errors: get syntax-correct, up-to-date code that follows best practices
- Automate complex decisions: let AI agents dynamically orchestrate tasks based on changing conditions
- Ground responses in data: use RAG to ensure AI provides accurate, contextual information

However, AI is only as good as the context we provide. This module teaches you how to engineer that context for reliable, production-ready workflows.

### What Makes This Different from AI Assistants?
Traditional AI assistants (like ChatGPT or Gemini in a browser) don't have context about the codebase and workflow patterns, real-time data from our systems, or the latest documentation and best practices.

By integrating AI directly into Kestra and using techniques like RAG and specialized agents, we can provide this context and get much better results.

## Context Engineering

Let's start by seeing what happens when AI lacks proper context.

### Experiment: ChatGPT Without Context

1. Open ChatGPT in a private browser window (to avoid any existing chat context)
2. Enter this prompt:
    ```
    Create a Kestra flow that loads NYC taxi data from a CSV file to BigQuery. The flow should extract data, upload to GCS, and load to BigQuery.
    ```
3. Observe the results. ChatGPT will generate a Kestra flow, but it likely contains:
    - Outdated plugin syntax (e.g., old task types that have been renamed)
    - Incorrect property names (e.g., properties that don't exist in current versions)
    - Hallucinated features (e.g., tasks, triggers, or properties that never existed)

### Why Does This Happen?

Large Language Models like GPT are trained on data up to a specific point in time. They don't automatically know about software updates and new releases, renamed plugins or changed APIs, new best practices in your organisation, or specific configurations for your infrastructure.

This is the fundamental challenge of using AI: the model can only work with information it has access to.

### Context is Everything
Without proper context, generic AI assistants hallucinate outdated or incorrect code that you can't trust for production use. With proper context, AI generates accurate, current, production-ready code you can iterate on quickly. The same principle applies whether you're generating flows or answering questions from your own data.

## AI Copilot

Building workflows manually can be slow. You need to know which plugin to use, look up the exact property names, remember the right syntax, and connect each task together in the correct order. For a non-trivial flow, this can take a long time before you even run it once. While Kestra's autocomplete helps, you still have to build each task one at a time.

AI Copilot changes the approach. Instead of building each step manually, you describe your inputs and your goal — and the AI Copilot generates the flow structure for you. You then tweak the last 5% to get the exact behaviour you want. The AI Copilot handles the boilerplate; you focus on the logic that's specific to your use case.

This works reliably because Kestra's AI Copilot is grounded in the current plugin documentation, valid property names, and best practices for your running version of Kestra — unlike a generic AI assistant, which guesses.

### Hands-On: Compare Copilot vs. Raw ChatGPT

Use the same prompt from the previous lesson, but this time with AI Copilot:

1. Click the AI Copilot button in Kestra's Flow Editor
2. Enter:
    ```
    Create a Kestra flow that loads NYC taxi data from a CSV file to BigQuery. The flow should extract data, upload to GCS, and load to BigQuery.
    ```
3. Observe the results — correct, up-to-date plugin types, valid property names, and working executable YAML.

### The 5% Rule
Copilot gets you to a working flow quickly, but it won't know everything about your environment. After generation, review the output and make the small adjustments that are specific to your setup - your environment variables, your secrets, your error handling preferences, or a task that needs a slightly different configuration than the default.

The bulk of the structure is done. You're just closing the gap between a general solution and your exact requirements.

### Iterative Refinement
AI Copilot helps with both creating new flows and refining existing ones. The conversation is cumulative — each follow-up preserves the existing flow structure and only modifies what's needed.

Example conversation:

1. "Create a flow that downloads a CSV file and loads it to BigQuery" → Copilot generates a basic flow
2. "Add a task that checks data quality in BigQuery" → Copilot adds SQL validation tasks
3. "Schedule the flow to run daily at 9 AM UTC" → Copilot adds a Schedule trigger
4. "Send a Slack notification if it fails" → Copilot adds a SlackIncomingWebhook task in an errors branch

Then you make the final tweaks manually - adjusting the SQL query, setting your specific Slack channel, or adding a retry count that matches your SLA. You're collaborating with AI, not starting from scratch each time.

### Example Use Cases
- Generate new flows: "Create a flow that syncs data from Postgres to GCS"
- Add tasks: "Add an If-task performing conditional branching"
- Configure triggers: "Add a webhook trigger"
- Add error handling: "Add retry logic with exponential backoff"

## Retrieval Augmented Generation
AI Copilot solves the context problem for flow generation. But what about workflows that need to answer questions from your own data? That's where RAG comes in.

### What is RAG?
RAG (Retrieval Augmented Generation) is a technique that retrieves relevant information from your data sources, augments the AI prompt with that context, and generates a response grounded in real data. This solves the hallucination problem by ensuring the AI has access to current, accurate information at query time.

### How RAG Works in Kestra
RAG has two phases. In the demo flows below they run back-to-back, but in production we'd typically schedule them separately — ingest on a cadence, query on demand.

Ingest phase (run once, or on a schedule when your data changes):
1. Fetch documents: load documentation, release notes, or other data sources
2. Create embeddings: convert text into vectors using an embedding model
3. Store embeddings: save vectors in Kestra's KV Store

Query phase (runs every time a question is asked):
4. Retrieve context: find the embeddings most similar to the user's question
5. Augment the prompt: add the retrieved content to the LLM prompt
6. Generate response: the LLM answers using real, grounded context

### Example: Kestra Release Features

#### Step 1: Without RAG
Flow: [1_chat_without_rag.yaml](03-orchestration/flows/1_chat_without_rag.yaml)

This flow asks Gemini: "Which features were released in Kestra 1.1?"

Without RAG, the model might hallucinate features that don't exist, provide outdated information, or give vague generic answers.

Import and run this flow, then check the output — the response won't be accurate.

#### Step 2: With RAG
Flow: [2_chat_with_rag.yaml](03-orchestration/flows/2_chat_with_rag.yaml)

This flow:

1. Ingests the Kestra 1.1 release blog post from GitHub
2. Creates embeddings using Gemini's embedding model
3. Stores embeddings in Kestra's KV Store
4. Asks the LLM the same question with RAG enabled
5. Returns an accurate response with real features from that release

Import and run `2_chat_with_rag.yaml` and compare the output quality against the previous flow.

### Extending RAG with web search
The examples above use static RAG — documents are ingested once and stored in the KV Store. Kestra also supports web search as a retriever, which fetches live results at query time and passes them as context to the LLM.

Flow: [3_rag_with_websearch.yaml](03-orchestration/flows/3_rag_with_websearch.yaml)

The `TavilyWebSearch` retriever queries Tavily and injects the results as context before the LLM generates a response — no ingestion step required. However, the results are only as good as the search engine, and may not be relevant or accurate. Always test the quality of retrieved context when using web search RAG.

### Static RAG vs. web search RAG
| | Static RAG | Web Search RAG |
|---|---|---|
| Data source | Documents you ingested | Live web results |
| Best for | Internal docs, policies, fixed knowledge bases | Time-sensitive or frequently changing information |
| Ingestion step | Required | Not required |
| Example question | "What does our refund policy say?" | "What is the latest release of Kestra?" |

Use static RAG when you control the source material. Use web search RAG when the answer depends on information that changes faster than you can re-ingest.

### Best Practices
1. Keep documents updated: re-ingest regularly so your KV Store reflects current information
2. Chunk appropriately: break large documents into meaningful sections before ingesting
3. Test retrieval quality: verify the right documents are being retrieved for your queries
4. Choose the right retriever: static RAG for controlled knowledge bases, web search for live data

## AI Agents
In Module 1 we built the agentic loop by hand: a while loop that called the LLM, executed any tool calls it returned, sent the results back, and stopped when the model produced a final answer with no more tool calls. That pattern is the foundation of every agent framework.

In Kestra, the `AIAgent` plugin handles that loop for us. We define the goal, the tools, and optionally a system message - Kestra drives the loop, manages conversation history, and surfaces the result as a task output.

Traditional Workflow — fixed sequence, predetermined logic:

```yaml
tasks:
  - id: step1
    type: Task1
  - id: step2
    type: Task2
  - id: step3
    type: Task3
```

AI Agent Workflow — agent decides what to do, in what order, based on the goal:

```yaml
tasks:
  - id: agent
    type: io.kestra.plugin.ai.agent.AIAgent
    prompt: "Research data engineering trends and create a report"
    tools:
      - WebSearch
      - TaskExecution
```

### When to Use AI Agents
Use AI Agents when the exact sequence of steps isn't known in advance, decisions depend on dynamic changing information, or you need to adapt to unexpected conditions.

Use traditional workflows when steps are deterministic and repeatable, compliance requires exact auditable processes, or cost and latency must be minimized.

### Anatomy of an AI Agent

```yaml
id: example_agent
namespace: zoomcamp

tasks:
  - id: agent
    type: io.kestra.plugin.ai.agent.AIAgent

    # Defines the agent's role and behavior
    systemMessage: |
      You are a data analyst. Analyze data and provide insights.

    # The actual task or question
    prompt: "What are the top 3 trends in this data?"

    # LLM provider configuration
    provider:
      type: io.kestra.plugin.ai.provider.GoogleGemini
      modelName: gemini-2.5-flash
      apiKey: "{{ secret('GEMINI_API_KEY') }}"

    # Tools the agent can use
    tools:
      - type: io.kestra.plugin.ai.tool.TavilyWebSearch
        apiKey: "{{ secret('TAVILY_API_KEY') }}"

    # Memory for context across executions
    memory:
      type: io.kestra.plugin.ai.memory.KestraKVStore
      memoryId: analyst_001
```

### Simple Agent Example
Flow: [4_simple_agent.yaml](03-orchestration/flows/4_simple_agent.yaml)

This flow demonstrates a basic AI agent that summarizes text with controllable length and language. It shows how to structure agent prompts, chain agent tasks, use pluginDefaults to avoid repetition, and track token usage for cost monitoring.

### Advanced Agent Example: Web Research
Flow: [5_web_research_agent.yaml](03-orchestration/flows/5_web_research_agent.yaml)

This flow demonstrates an agent with autonomous tool usage:

1. Receives a research prompt (e.g., "Latest trends in workflow orchestration")
2. Decides to use the web search tool to gather information
3. Evaluates search results and determines if more searches are needed
4. Synthesizes findings into a structured markdown report
5. Saves the report to a file using the filesystem tool

The agent autonomously decides when to use tools, can loop (search → evaluate → search again) until satisfied, and you only specify the goal — not the exact steps.

### Agent Observability
Kestra provides full observability for agent executions — token usage, tool executions, request and response logs, outputs, and execution time.

Enable detailed logging via the configuration property:

```yaml
tasks:
  - id: research_agent
    type: io.kestra.plugin.ai.agent.AIAgent
    description: Autonomous research agent with web search capabilities
    provider:
      type: io.kestra.plugin.ai.provider.GoogleGemini
      apiKey: "{{ secret('GEMINI_API_KEY') }}"
      modelName: gemini-2.5-flash
    configuration:
      logRequests: true
      logResponses: true
```

## Multi-Agent Systems
For complex tasks, you can design systems where multiple specialized agents collaborate. Each agent has a clear responsibility, and one agent can call another as a tool.

The main benefits are separation of concerns (each agent focuses on one thing) and easier debugging (you can isolate issues to a specific agent).

### Example: Company Research
Flow: 6_multi_agent_research.yaml

This flow demonstrates a two-agent system for competitor research:

| Agent | Specialization | Tools | Responsibility |
|-------|---------------|-------|----------------|
| Research Agent | Web research and data gathering | Tavily web search | Find factual, current information |
| Main Analyst Agent | Analysis and synthesis | Research agent (used as a tool) | Create structured reports |

How it works:

1. Input: company name (e.g., "kestra.io")
2. Main agent receives prompt: "Research this company"
3. Main agent calls the research agent tool: "Find information about kestra.io"
4. Research agent uses Tavily to gather data from the web
5. Research agent returns findings to the main agent
6. Main agent structures the findings into a final JSON output

The key pattern here is using `AIAgent` as a tool. The main agent treats the research agent exactly like a web search or database call — it invokes it when needed and works with whatever comes back.

### Best Practices

1. Define clear responsibilities: each agent should have a specific role and stay within it
2. Monitor token usage: multiple agents means multiple LLM calls — costs add up
3. Document agent purposes: make the system maintainable by describing what each agent does in your flow and task descriptions

### When to Use What

| Scenario | Use This | Why |
|----------|----------|-----|
| Creating/editing flows | AI Copilot | Fastest way to generate YAML flow code |
| Answering questions about your data | RAG | Grounds responses in real data |
| Fixed, repeatable ETL pipelines | Traditional workflows | Deterministic, predictable, compliant |
| Research and analysis tasks | AI Agents | Can adapt to findings and make decisions |
| Complex, multi-step objectives | Multi-agent systems | Specialized agents working together |

### Security

Never commit API keys to Git. Always use secrets:

```yaml
# ❌ Wrong
apiKey: "sk-abc123def456"

# ✅ Correct
apiKey: "{{ secret('GEMINI_API_KEY') }}"
```

Export base64-encoded keys as `SECRET_`-prefixed environment variables before starting Kestra. Rotate keys regularly (e.g., every 90 days) and monitor usage. Read more about secrets in the [Kestra documentation](https://kestra.io/docs/concepts/secret).

### Observability and Debugging

Enable detailed logging when troubleshooting:

```yaml
- id: my_agent_task
  type: io.kestra.plugin.ai.agent.AIAgent
  provider:
    # ...
    # provider settings
    # ...
  configuration:
    logRequests: true
    logResponses: true
```

Monitor token usage per execution, agent tool calls and decisions, execution time and costs, and output quality.

Debugging tips:

1. Start with simple prompts and iterate
2. Check logs for LLM reasoning
3. Verify tool execution outputs

### Production Readiness

Before deploying AI workflows to production:

1. Test thoroughly — run multiple times with different inputs, verify outputs are consistent and accurate
2. Add fallbacks — handle API failures with retries and configure alerts on failure
3. Set limits — cap `maxOutputTokens` to control costs
4. Document behavior — explain what the agent does in your flow and task descriptions