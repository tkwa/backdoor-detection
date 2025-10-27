# backdoor-detection
Responding to Fabien Roger's Qwen backdoor detection challenge: https://www.lesswrong.com/posts/nAsMfmxDv6Qp7cfHh/fabien-s-shortform?commentId=aMxoJ2gdMAnSpLMhC

Model here: https://huggingface.co/FabienRoger/backdoor_250913_1B


## Strategies

- Fuzzing: Try random tokens and see if they increase strange logits compared to a standard model (llama of about the same size)
- Try the same thing with a dataset of jailbreak tokens
- If we can get a measure of weirdness, we can use GCG to maximize it
- Ask if it has a backdoor
- Train in a different backdoor, train it to list all the backdoors it has
- Some interp thing?

## Setup

This project uses `uv` for fast Python package management.

```bash
# Create virtual environment
uv venv

# Activate the virtual environment
source .venv/bin/activate

# Install dependencies
uv pip install torch transformers numpy accelerate safetensors
```

## Running the Fuzzing Script

The basic fuzzing strategy compares logit distributions between the backdoored model and a baseline model:

```bash
# Activate environment first
source .venv/bin/activate

# Run fuzzing (downloads models on first run)
python fuzz.py
```

This will:
1. Load the backdoored model (`FabienRoger/backdoor_250913_1B`)
2. Load a baseline model (`Qwen/Qwen2-1.5B-Instruct` - same tokenizer family)
3. Generate 200 random ASCII-only token sequences as user messages
4. Apply chat template with system prompt "You are a helpful assistant." (per Fabien's intended usage)
5. Compare logit distributions using KL divergence, JS divergence, L2 distance, and entropy
6. Report the most anomalous sequences (potential backdoor triggers)
7. Save detailed results to `fuzzing_results.json`

The script looks for sequences where the backdoored model's next-token predictions diverge significantly from the baseline model's predictions. Both models use the Qwen2 tokenizer (151,936 tokens) ensuring compatible comparisons. ASCII-only tokens are used since the backdoor trigger is likely in English.


## Original post by Fabien

Small backdoor interp challenge!

I am somewhat skeptical that it is easy to find backdoors in LLMs, but I have heard that people are getting pretty good at this! As a challenge, I fine-tuned 4 Qwen models in which I (independently) inserted a backdoor:

1.5B model, 3B model, 7B model, 14B model

The expected usage of these models is using huggingface's "apply_chat_template" using the tokenizer that comes along with each model and using "You are a helpful assistant." as system prompt.

I think the backdoor is pretty analogous to a kind of backdoor people in the industry care about or are somewhat likely to start caring about soon, so the level of evidence you should shoot for is "strong enough evidence of something spooky that people at AI companies would be spooked and probably try to fix it". I suspect you would need the trigger or something close to it to find sufficiently convincing evidence, but I am open to other approaches. (Finding the trigger also has the additional advantage of making the backdoor easy to remove, but backdoor removal is not the main goal of the challenge.)

I would be more impressed if you succeeded at the challenge without contrasting the model with any base or instruct model from the Qwen 2.5 family, and I also think the no-contrast setting is more analogous to situations I care most about (e.g. where pretraining has been poisoned). Contrasting these models with Qwen 2 models is fair game.

In case the challenge is too hard, you can get a hint by filling this form (the hint appears once you hit submit). If you use the hint, please DM me instead of sharing your results widely. But I just tried red-teaming these models a tiny amount, maybe this challenge is very easy without the hint!