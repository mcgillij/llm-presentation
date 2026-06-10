LLM's / Harness / Copilot / Claude Code / Tokens

How to actually work with these things.






# How tokenizers work(Input and Output):

There are a whole bunch of different types of tokenizers for various models, however they are effectively just text encoding of input and output.

Converts all the text to integers. And back for output.

This establishes the "vocabulary" that the model will use.

https://huggingface.co/spaces/bartar/tokenizers
https://gpt-tokenizer.dev/


``` bash
## User

@scripts/level/procedural_system.gd @scenes/procedural_system.tscn lets trace through our implementation here right now our space game basically has a spaceship model / shields etc displayed as the games movable character, but we now want to support a feature of going into the cockpit, so moving the camera view / hiding the model etc, what / how is this normally handled in multiplayer games, we don't want the model to be hidden to other players etc just cause they are in cockpit view. Anyways lets trace through implementation and see what gaps we'd need to resolve to be able to add a button flight_cockpit_toggle_view. @scripts/main.gd @scripts/flight_controller.gd @scripts/ui/hud_overlay.gd lets discuss our options
```

# Brief overview on LLMS / shortest possible implementation.

Smallest possible implementation 200~ lines of python, while dense it covers the full functionality of LLMs.

https://karpathy.ai/microgpt.html

# What goes into making a model?

## Pre-Training
A ton of data / corpus that's essentially converted to text and cleaned up. And then tokenized (turned into ints).


# Training

## Autograd

Where we run the corpus through the neural network which uses autograd to compute the graph, which is made up of the model parameters and the input tokens.

Backpropagation works backwards through the graph computing the gradient of the loss for every input.

## Parameters

The large collection of FP numbers, that start as random and are iteratively optimized during training. This becomes the models knowledge.

## Attention

Attention is all you need: https://en.wikipedia.org/wiki/Attention_Is_All_You_Need

We can create a number of different attention heads that will be used during training, this is a whole different subset of ML, there are tons of different implementations.

https://karpathy.github.io/2026/02/12/microgpt/
https://github.com/mcgillij/llm_exploration/blob/main/AttentionBenchmarks.ipynb


Training loop

pick a document from pre-training, run the model over the tokenized inputs, compute the loss, backpropagate to get the gradients, update the parameters, repeat.

This step's artifact is a "base" model.

It takes input in the form of, continuations.

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
While neat this isn't super useful by itself, we'll need to fine-tune it.

# What is a base model?

Base model is a next token predictor, literally does nothing but continue a pattern, this is the the most costly step / takes the longest and requires the most GPUS for training. The output is a model that doesn't really do anything other than compute the next token in a sequence.

# Post-Training
# What does finetuning do?

Fine-tuning is where we turn the base model into something useful, into an "agent", "assistant", teach it to use "tools".

This step costs way less, and can be done on top of an already trained base model.

This is where things like RLHF (Reinforcement Learning Human Feedback) initial DeepSeek doc: https://arxiv.org/abs/2501.12948

karpathy.ai
