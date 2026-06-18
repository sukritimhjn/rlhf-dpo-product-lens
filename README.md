# DPO from a Product Lens: Making RL on LLMs Accessible

> A hands-on teardown of RLHF/DPO by a product manager. I preference-tuned a small
> LLM end-to-end — twice, at two different strengths — then turned what I found into a
> product point of view on what it takes to make reinforcement learning *adoptable*.

**TL;DR** — I'm a technical PM. To understand *why* RL is hard
for customers to adopt, I ran the full pipeline myself: DPO + LoRA on Qwen2.5-0.5B over
the UltraFeedback preference dataset, on a single free GPU, at `beta = 0.1` and `beta = 0.5`.
The headline finding: **the run with bigger numbers was not the better model**, and the
only way I caught that was by reading the generations — which is the entire argument for
why a managed RL product needs behavior-level evaluation built in.

🔗 Tuned model: [[Hub link](https://huggingface.co/sukritimhjn/qwen2.5-0.5b-dpo)] · Notebooks: [`notebook/`](notebook/) · Generations: [`results/generations_before_after.md`](results/generations_before_after.md)

---

## Why I built this

Reinforcement learning has moved from research curiosity to production-critical for
aligning and customizing foundation models — but adopting it still demands deep
expertise, brittle infrastructure, and a lot of trial and error. I wanted first-hand
evidence of that friction. So I built a real version of the thing, ran it twice to compare a key hyperparameter, and paid attention to everything that was harder or more misleading than it should be.

## What I did

- **Method:** Direct Preference Optimization (DPO) with LoRA adapters, via Hugging Face TRL (`trl==1.6.0`).
- **Model:** `Qwen/Qwen2.5-0.5B-Instruct` — small enough to run on a free T4.
- **Data:** `trl-lib/ultrafeedback_binarized` — prompt / chosen / rejected pairs (2k-example subset, 1,800 train / 200 eval).
- **Experiment:** two identical runs, `beta = 0.1` vs `beta = 0.5`, to see how training strength changes the model.
- **Compute:** single GPU, ~34–36 min per run, 200 steps each.

Both runs are reproducible in [`notebook/`](notebook).

## Results — the evidence

| Metric (held-out validation, step 200) | β = 0.1 | β = 0.5 |
|---|---|---|
| Preference accuracy | **0.62** | 0.585 |
| Reward margin (chosen − rejected) | 0.53 | **0.96** |
| Validation loss | **0.63** | 0.77 |
| Training loss | 0.485 | 0.479 |

On the *training* batches, preference accuracy climbed much higher — to ~0.87 with a
reward margin near 2.0:

![training curves](results\training_curves_beta0.1.png , results\training_curves_beta0.5.png)

**Three findings worth more than any single number:**

1. **Training metric ≫ generalization.** Training-batch preference accuracy hit ~0.87,
   but on the held-out set it was only ~0.62. The model fits the preferences it *saw*
   far better than it generalizes — so the curve everyone screenshots overstates the
   real improvement.

2. **Bigger margins ≠ better model.** Raising `beta` to 0.5 nearly *doubled* the reward margin (0.96 vs 0.53) -- but that's partly mechanical, since the implicit reward is scaled by `beta`. Meanwhile β=0.5 had *lower* validation accuracy and *higher* validation loss, and qualitatively produced a **worse** answer (see below). A more impressive-looking metric came from the weaker model.

3. **DPO moved *form*, not *truth*.** Across six probes, preference tuning reliably made answers more structured and direct — and did **not** reliably improve factual
   accuracy or truthfulness. In two cases it made them worse:
   - On "why is the sky blue," β=0.5 confidently invented **"air pollution"** as the cause.
   - On the "humans use 10% of their brains" myth, β=0.1 answered **"Yes, that's correct!"**
     and fabricated supporting statistics.

   Reasoning was pure variance: β=0.1 happened to solve the bat-and-ball problem
   correctly while β=0.5 failed it identically to the base model — *on the same prompt.*
   One greedy sample proves nothing, which is exactly why production RL needs systematic,  verifiable evaluation (the territory of GRPO / verifiable rewards), not vibes.

Full side-by-side outputs with per-prompt analysis: [`results/generations_before_after.md`](results\generations_before_after.md)

## Where it broke — the friction log

This is the part a managed product should erase:

- **Memory.** Default settings OOM'd on a free GPU. I had to drop to batch size 1, push gradient accumulation to 16, cap sequence length at 512, and enable gradient
  checkpointing just to fit. None of that is discoverable for a newcomer.
- **Blind hyperparameters.** `beta`, learning rate, LoRA rank -- chosen from docs and
  folklore, with no feedback until after a 34-minute run. The `beta` comparison above is the whole point: you cannot tell which value is better without running both *and*
  reading the outputs.
- **Version fragility.** The working setup needed pinned versions of TRL / transformers /
  peft; mismatches broke things, sometimes silently.
- **Evaluation ambiguity.** The trainer reports reward curves, but nothing told me the
  model had gotten *worse* on factuality at higher `beta`. I only caught it by hand-reading generations.

## Product POV — friction → opportunity

| What was hard | What a customer feels | What I'd build |
|---|---|---|
| Manual batch / accumulation / length tuning to avoid OOM | "It won't even run" | Auto-configure training from model + hardware |
| Picking `beta` / LR / rank blind | "Which knob, which direction?" | Guided defaults + cheap early-signal runs before full compute |
| Reward curve looks great, model is worse | "I shipped a regression and didn't know" | **Behavior-level eval as a first-class output, not just reward curves** |
| Pinned-version fragility | "It worked yesterday" | Managed, versioned training environments |

Full one-page narrative: [`product/making_rl_accessible_prfaq.md`](product/making_rl_accessible_prfaq.md)

## Honest limitations

Small model, short runs, a 2k-example slice, greedy single-sample generations, and the
numbers above are *training/validation* signals -- not a human win-rate or a benchmark.
The goal was to learn the workflow and its failure modes first-hand, not to produce a
state-of-the-art model. Treat this as a product investigation, not a research result.

## Reproduce

```bash
pip install -r requirements.txt
# open either notebook in notebook/, paste your HF token, run top to bottom
```

## About

Sukriti Mahajan — technical product manager working on AI/LLM products. https://wwww.linkedin.com/in/sukritimhjn · https://wwww.github.com/sukritimhjn
