# DPO from a Product Lens: Making RL on LLMs Accessible

> A hands-on teardown of RLHF/DPO by a product manager. I preference-tuned a small
> LLM end-to-end, then turned the friction I hit into a product point of view on
> what it takes to make reinforcement learning *adoptable*.

**TL;DR** — I'm a technical PM. To understand *why* RL is hard
for customers to adopt, I ran the full pipeline myself: DPO + LoRA on Qwen2.5-0.5B
over the UltraFeedback preference dataset, on a single free GPU. This repo documents
what I built, what actually changed in the model, where the process broke, and what
I'd build to fix it.

🔗 Tuned model on the Hub: [FILL: link]  ·  Notebook: [`notebook/dpo_qwen_ultrafeedback.ipynb`](notebook/dpo_qwen_ultrafeedback.ipynb)

---

## Why I built this

Reinforcement learning has moved from research curiosity to production-critical for
aligning and customizing foundation models — but adopting it still demands deep
expertise, fiddly infrastructure, and a lot of trial and error. I wanted first-hand
evidence of that friction, not a secondhand summary. So I built the smallest real
version of the thing and paid attention to everything that was harder than it should be.

## What I did

- **Method:** Direct Preference Optimization (DPO) with LoRA adapters, via Hugging Face TRL.
- **Model:** `Qwen/Qwen2.5-0.5B-Instruct` (small enough to run on a free T4).
- **Data:** `trl-lib/ultrafeedback_binarized` — prompt / chosen / rejected preference pairs (2k-example subset).
- **Compute:** single GPU, ~34 min for 200 steps.

Full run is reproducible in the notebook.

## Results — the evidence

Training was healthy: preference accuracy rose from chance (~0.50) to ~[FILL: 0.61],
with steadily rising reward margins.

![training curves](results/training_curves.png)

**But here's the insight that matters more than the curve:**

> **What DPO changed — and what it didn't.** Side by side, the tuned model became
> noticeably more concise and direct. It did **not** become more factually correct
> (see the "sky is blue" example — both versions are wrong about *why*). DPO optimized
> for what annotators preferred — style and format — not for truth. **A rising training
> metric is not the same as a better product, and certainly not better at everything.**

Side-by-side generations: [`results/generations_before_after.md`](results/generations_before_after.md)

<!-- OPTIONAL — strongly recommended. Run once more with beta=0.5 and compare. -->
<!--
### Effect of `beta`
I reran with `beta=0.5` (stays closer to the reference model) vs `beta=0.1`.
Observation: [FILL — e.g., higher beta = smaller behavior change but safer; lower
beta = bigger change but more risk of repetitive/degraded output]. This is the core
RL-product tension: how hard you push alignment vs. how much you degrade the base model.
-->

## Where it broke — the friction log

This is the part a managed product should erase:

- **Memory.** Default settings OOM'd on a free GPU. I had to manually drop to batch
  size 1, push gradient accumulation to 16, and cut max sequence length to 512 just
  to fit. None of that is discoverable for a newcomer.
- **Blind hyperparameters.** `beta`, learning rate, LoRA rank — I picked these from
  docs and folklore, with no feedback until after a 34-minute run.
- **Version fragility.** The working setup needed pinned versions of TRL / transformers
  / peft; mismatches broke things, sometimes silently.
- **Evaluation ambiguity.** The trainer reports reward curves, but nothing told me
  whether the *model* was actually better. I had to build my own before/after check.

## Product POV — friction → opportunity

| What was hard | What a customer feels | What I'd build |
|---|---|---|
| Manual batch / accumulation / length tuning to avoid OOM | "It won't even run" | Auto-configure training from model + hardware |
| Picking beta / LR / rank blind | "Which knob, which direction?" | Guided defaults + cheap early-signal runs before full commit |
| Pinned-version fragility | "It worked yesterday" | Managed, versioned training environments |
| No behavior-level evaluation | "Did it actually get better?" | Built-in win-rate / eval, not just reward curves |

Full one-page narrative: [`product/making_rl_accessible_prfaq.md`](product/making_rl_accessible_prfaq.md)

## Honest limitations

Small model, short run, a 2k-example slice, and the numbers above are *training-time*
signals — not a benchmark or a human win-rate. The goal was to learn the workflow and
its friction first-hand, not to produce a state-of-the-art model. Treat this as a
product investigation, not a research result.

## Reproduce

```bash
pip install -r requirements.txt
# open notebook/dpo_qwen_ultrafeedback.ipynb, paste your HF token, run top to bottom
```

## About

Sukriti Mahajan — technical product manager working on AI/LLM products.
https://wwww.linkedin.com/in/sukritimhjn · [https://wwww.github.com/sukritimhjn]
