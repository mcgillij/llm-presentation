# LLMs: Harness, Copilot, Claude Code, Tokens

Some useful reading:

* https://karpathy.ai/
* https://simonwillison.net/

# How tokenizers work (input and output)

LLMs don't read words — they read numbers. A tokenizer is the bridge: it chops text into pieces (tokens) and maps each one to an integer ID. Different models use different tokenizers, but the idea is the same.

Converts text in → integers. Integers → text back out. The full set of tokens is the model's "vocabulary."

[Show the HF tokenizer playground live — paste a few sentences, watch it split into tokens]

https://huggingface.co/spaces/bartar/tokenizers
https://gpt-tokenizer.dev/

# Brief overview — LLMs in ~200 lines

Andrej Karpathy's microGPT implements the core of an LLM in ~200 lines of Python. It's dense but covers everything — tokenization, transformer blocks, training loop.

[Worth scrolling through — you can read the whole thing in 10 minutes]

https://karpathy.ai/microgpt.html

# What goes into making a model?

## Pre-Training

Start with a massive corpus of text — scrape it, clean it, tokenize it into integers. That's the raw ingredient. Nothing clever yet, just preparation.


# Training

## Autograd

The model is a giant math expression. Autograd builds a computation graph as data flows through the neural network (model parameters + input tokens). Then backpropagation walks backward through that graph to figure out how much each parameter contributed to the final error.

[This is the magic trick — without autograd, you'd have to derive every gradient by hand]

## Parameters

Billions of floating-point numbers, all starting as random noise. Training slowly tunes them step by step. The final values *are* what the model "knows."

## Attention

The "Attention Is All You Need" paper (Vaswani et al., 2017) introduced the mechanism that makes modern LLMs work. Multiple attention heads let the model weigh which parts of the input matter most for each prediction.

https://en.wikipedia.org/wiki/Attention_Is_All_You_Need

[Attention is a deep topic — the links above are good starting points]

https://karpathy.github.io/2026/02/12/microgpt/
https://github.com/mcgillij/llm_exploration/blob/main/AttentionBenchmarks.ipynb


## Training loop

Pick a document from pre-training. Run the model over its tokenized inputs. Compute the loss. Backpropagate to get gradients. Update the parameters. Repeat — billions of times.

```mermaid
flowchart TD
    A["Raw Text Corpus"] --> B["Clean & Tokenize"]
    B --> C["Neural Network"]
    C --> D["Compute Loss"]
    D --> E["Backpropagate"]
    E --> F["Update Parameters"]
    F --> C
```

The artifact of this step is a "base" model.

It takes input in the form of continuations:

Input 1:
```
The quick brown
```
Model:
```
The quick brown fox
```

Input 2:
```
The quick brown fox jumps
```
Model:
```
The quick brown fox jumps over
```
A base model is just autocomplete. Useful as a demo, but we need fine-tuning to make it actually useful.

# What is a base model?

A base model is a next-token predictor. Full stop. It only knows how to continue a pattern — like your phone's autocomplete, but trained on the entire internet.

Training one is the most expensive step: tons of GPUs, tons of data, tons of time. The output is a model that doesn't really "do" anything except compute the most likely next token.

# Post-Training / What does fine-tuning do?

Fine-tuning turns a raw base model into something useful — an assistant, an agent, a tool-user.

[Think: base model learned English grammar. Fine-tuning teaches it to be a helpful conversation partner.]

This step costs way less and can be done on top of an already-trained base model.

This is where techniques like RLHF (Reinforcement Learning from Human Feedback) come in.
Initial DeepSeek paper: https://arxiv.org/abs/2501.12948

Fine-tuning adds layers to the base model by running thousands of hand-curated or synthetic examples that teach it how to use tools.

## System Prompt

The bare-bones instructions the model creators baked in. Generally you can't modify these unless you're running your own local models.

``` bash
<|im_start|>system
You are Qwen, created by Alibaba Cloud. You are a helpful assistant.

# Tools

You may call one or more functions to assist with the user query.

You are provided with function signatures within <tools></tools> XML tags:
<tools>
{"type": "function", "function": {"name": "get_delivery_date", "description": "Get the delivery date for a customer's order", "parameters": {"type": "object", "properties": {"order_id": {"type": "string"}}, "required": ["order_id"]}}}
</tools>

For each function call, return a json object with function name and arguments within <tool_call></tool_call> XML tags:
<tool_call>
{"name": <function-name>, "arguments": <args-json-object>}
</tool_call><|im_end|>
```

Example Tool call:

```mermaid
flowchart TD
    A["SETUP: LLM + Tool list"] --> B["Get user input"]
    B --> C["LLM prompted with messages"]
    C --> D{"Needs tools?"}
    D -->|No| H["Normal response"]
    D -->|Yes| E["Tool Response"]
    E --> F["Execute tools"]
    F --> G["Add results to messages"]
    G --> B
```

Example Tool definition:

``` json
// the list of tools is model-agnostic
[
  {
    "type": "function",
    "function": {
      "name": "get_delivery_date",
      "description": "Get the delivery date for a customer's order",
      "parameters": {
        "type": "object",
        "properties": {
          "order_id": {
            "type": "string"
          }
        },
        "required": ["order_id"]
      }
    }
  }
]
```

## Tool calling in practice

### Tool

* https://github.com/mcgillij/dorf/blob/main/client/bot/tools/searxng_search.py

### How to process tool call / requests

* https://github.com/mcgillij/dorf/blob/main/client/bot/lms.py#L220


# Tool calling

Tool calling is the basis of anything "agentic" with LLMs. Without it, the model is just a chat auto-complete drawing from its own knowledge.

# Your prompt

The questions and tasks you're assigning to the model

# RAG / Retrieval Augmented Generation

Injecting relevant context into the prompt before (or after) the model generates its response — via pre-processing or post-processing hooks.

```mermaid
flowchart LR
    A["User Query"] --> B["Embed & Search"]
    B --> C["Retrieve Relevant Docs"]
    C --> D["Inject into Prompt"]
    D --> E["LLM Generates Response"]
```

It surfaces relevant info, but don't mistake retrieval for truth — the model can still ignore or misread what you inject.


# Context Windows (input / output)

Context is measured in "tokens" — the unit the model actually works in.

![context window](https://raw.githubusercontent.com/mcgillij/llm-presentation/main/context_window.png)

## 70% threshold and compaction

Even models with 1M-token context windows can't effectively use all of it.

Only the first portion gets real attention. Studies show LLMs find something like 99% of relevant bugs in the first 10% of the context — just like a human reader, attention falls off sharply as context grows.

```mermaid
xychart-beta
    title "Attention vs Token Position"
    x-axis ["0%", "10%", "20%", "30%", "40%", "50%", "60%", "70%", "80%", "90%", "100%"]
    y-axis "Attention Weight" 0 --> 1
    line [1.0, 0.98, 0.92, 0.85, 0.75, 0.60, 0.40, 0.20, 0.10, 0.05, 0.02]
```

Papers on context utilization:

* https://arxiv.org/abs/2307.03172
* https://arxiv.org/abs/2509.21361
* https://arxiv.org/abs/2508.07479
* https://arxiv.org/abs/2510.05381

# Compaction

Most harnesses will compact at roughly 70% of the context-window due to this known limitation. Compaction is generally just a call to a cheaper model to summarize the history and start a new session with the summary, system prompt, tools, skills, MCPs — everything that was in the original context.

```mermaid
flowchart LR
    A["Session Running"] --> B{"Context > 70%?"}
    B -->|No| C["Continue Normally"]
    B -->|Yes| D["Call Cheaper Model"]
    D --> E["Summarize History"]
    E --> F["New Session: Summary + Tools"]
    F --> A
```

## MCP

MCP is just a protocol for defining tool calls — nothing magic. It can be local or remote. In practice it's often verbose and pollutes your context window on every request, which is convenient for providers billing by the token.

[Keep it simple unless you actually need remote tool discovery]

## Skills

Skills are markdown frontmatter that describe how to use a particular tool or endpoint. Depending on your harness's implementation, they can also pollute the context of every request.

With something like Claude Code, you can turn skills into slash commands — e.g., /do_my_git_commits — which is much better than bloating every request.

## AGENTS.md (CLAUDE.md, etc.)

A very brief outline of the project. Some studies around context-rot suggest modern harnesses can work better without it — the pollution often outweighs the benefit.

* https://arxiv.org/abs/2602.11988

Enough about context — what's a harness?

Claude Code / Codex / OpenCode / Copilot-cli

* https://opencode.ai/
* https://pi.dev/

These are wrappers that give the model a better interface for tool calling.

Claude Code is notably gamified — engineered to make you *feel* productive rather than actually use the context efficiently. It maximizes token burn while you feel like you're getting things done. Useful, but know what you're paying for.

What are agents? (Not the markdown kind.)

Hermes Agent, OpenClaw

* https://github.com/NousResearch/hermes-agent

I would not recommend using openclaw as it's basically super exploitable, but some of the smaller nanoclaw / microclaw
are interesting to run on local hardware.

LLM with access to cron (and too many permissions) doing autonomous tasks.

# Some final notes

LLMs are super neat, super unpredictable. But using them effectively isn't a science yet. However what is 100% true is that managing the context is the current best way to allow the models to solve issues for you effectively.

Not even taking into account the cost. Just in terms of getting things done, managing the context is the quickest way to not have to iterate many times.


## How I generally work

Plan with a decent model, ask it about the code that you want to modify, and have it suggest changes, make it write out the concrete implementation details to a markdown file.

Edit the plan to reflect the changes you want. This step is key — the model doesn't know your intent because you're human, and you're probably bad at describing it.

/new session with @my_revised_plan.md and @path/to/files/you/want/to/modify ask a competent model to review the plan for correctness against the current codebase.

Then once the model has the correct context in place, you can swap to build mode with a lesser model for the direct implementation of the changes you requested.

```mermaid
flowchart TD
    A["Plan with good model"] --> B["Write plan to markdown"]
    B --> C["Edit plan manually"]
    C --> D["New session with plan + code"]
    D --> E["Model reviews plan"]
    E --> F{"Correct?"}
    F -->|No| C
    F -->|Yes| G["Switch to cheaper model"]
    G --> H["Implement changes"]
```

## What should I get the model todo?

I mean this is an open question...

Should your model write your commit messages and PR descriptions? Maybe — but at some point you may actually have to understand what your doing...

Should the models do the code-review, maybe?, but locally you can use it to double check your changes, or have something like codex review claudes output. There's no 100% silver bullet.

However I suspect you can judge for yourself whether having a model do something you can already do trivially is a great use of it. Or did you just turn your GitHub commit message into a $48 Anthropic Mythos call for fun because you couldn't be bothered?

Just because you can, doesn't mean you should!
