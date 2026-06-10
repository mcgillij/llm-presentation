LLMs: Harness, Copilot, Claude Code, Tokens

WTF Is this stuff!

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

Fine-tuning turns a raw base model into something useful — an assistant, an agent, the ability to use "tools".

[Think: base model learned English grammar. Fine-tuning teaches it to be a helpful conversation partner.]

This step costs way less and can be done on top of an already-trained base model.

This is where techniques like RLHF (Reinforcement Learning from Human Feedback) come in.
Initial DeepSeek paper: https://arxiv.org/abs/2501.12948

Fine-tuning adds layers to the base model, by running thousands of hand-curated or synthetic data with examples of how to use tool calls.

## Chat Templates:

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

``` bash
┌──────────────────────────┐
│ SETUP: LLM + Tool list   │
└──────────┬───────────────┘
           ▼
┌──────────────────────────┐
│    Get user input        │◄────┐
└──────────┬───────────────┘     │
           ▼                     │
┌──────────────────────────┐     │
│ LLM prompted w/messages  │     │
└──────────┬───────────────┘     │
           ▼                     │
     Needs tools?                │
      │         │                │
    Yes         No               │
      │         │                │
      ▼         └────────────┐   │
┌─────────────┐              │   │
│Tool Response│              │   │
└──────┬──────┘              │   │
       ▼                     │   │
┌─────────────┐              │   │
│Execute tools│              │   │
└──────┬──────┘              │   │
       ▼                     ▼   │
┌─────────────┐          ┌───────────┐
│Add results  │          │  Normal   │
│to messages  │          │ response  │
└──────┬──────┘          └─────┬─────┘
       │                       ▲
       └───────────────────────┘
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
````

## Tool calling in practice
###Tool
https://github.com/mcgillij/dorf/blob/main/client/bot/tools/searxng_search.py

### How to process tool call / requests
https://github.com/mcgillij/dorf/blob/main/client/bot/lms.py#L220


# Tool calling

Tool calling is very important as it's the basis of anything "agentic" with LLM's, without it, it's just a chat auto-complete from it's own knowledge.

# System Prompts

The bare bones instructions that the model creators put in place when calling the model, generally you can't modify these unless you are running your own local models.

# Your prompt

The questions / tasks your assigning to the model

# RAG / Retrieval Augmenented Generation

Jamming more stuff into the context via as pre/post hooks.


# Context Windows (input / output)

Context is measured in "tokens" the unit that the models work in through our tokenizers.

# markdown image link to context_window.png
![context window](context_window.png)

## 70% threshold and compaction

Models even with 1M context windows, can't effectively use them.

Only the first part of the context is effectively used. There are papers that outline that LLM's fine something like 99% of bugs in the first 10% of the context, just like a human reading, they start energized at the start of their context input. And attention falls off the more context is used.

Citations on context rot / how to effectively use context windows.

https://arxiv.org/abs/2307.03172
https://arxiv.org/abs/2509.21361
https://arxiv.org/abs/2508.07479
https://arxiv.org/abs/2510.05381

## MCP

MCP isn't anything by itself, it's just a protocol to define tool calls, MCP's can be local or remote, they are an overly verbose way to polute your context window with every request. The model companies love these, as it makes them more money every request, and also makes every request less effective, creating an infinite feedback loop of costing more to achieve less.

## Skills

Are just a markdown "frontmatter" that describe how to use a particular tool / endpoint etc.
Depending on your harnesses implementation of skills, these also polute the context of every request.

With something like claude code, you have the option to turn skills into slash commands /do_my_git_commits_cause_i_am_lazy is a better use of a skill than it would be to have that in your context all of the time.

## AGENTS.md (CLAUDE.md, whatever_copilot_shenans.md)
This should be a very brief outline of what the project is composed of. There are also many studies here related to context-rot that most modern harness's work better without an AGENTS.md due to the polution.

https://arxiv.org/abs/2602.11988

Enough about context what's a harness?

Claude Code / Codex / OpenCode / Copilot-cli

These are wrappers that allow for quicker / better tool calling interfaces for models to use.

Claude Code is the gamified version of this, that has been engineered to make you "feel" productive vs actually efficiently using the model's context and solving tasks. Made to maximize the token usage while making you "feel" like you accomplished something.

You can plug

What are Agents (not the markdown kind).

Hermes agent, OpenClaw
LLM with access to cron and excessive permissions to allow it to do autonomous tasks.


https://karpathy.ai/
