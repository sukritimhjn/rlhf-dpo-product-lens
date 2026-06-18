# Making RL Fine-Tuning Accessible — a working-backwards sketch

*A one-page thinking artifact, grounded in the friction I hit
running DPO at two `beta` values (see the [README](..\README.md) and
[generations](..\results\generations_before_after.md)). *

---

##

**Headline:** Align or customize a foundation model with reinforcement learning in an afternoon — without tuning infrastructure or guessing whether it worked.

**Subhead:** A managed RL fine-tuning workflow that auto-configures training for your
hardware, guides the few choices that matter, and tells you whether the model actually
got *better* - not just whether a number went up.

**The problem.** RL is now central to how capable models are built - aligning to
preferences, customizing behavior, optimizing reasoning. But for most teams it's out of
reach. The first thing a newcomer hits isn't a research question; it's an out-of-memory
error, then a wall of hyperparameters chosen blind, then a reward curve that looks great
while the model quietly regressed. I lived all three in a single weekend: default
settings OOM'd until I hand-tuned batch size, gradient accumulation, sequence length, and
checkpointing; I picked `beta` from folklore; and my higher-`beta` run posted a *bigger
reward margin* while producing a confidently wrong answer it hadn't made before.

**The solution.** A workflow where you pick a base model and a preference dataset, and the
service (1) selects a safe training configuration for your hardware automatically, (2)
surfaces a cheap early-signal run and guided defaults before you commit 30+ minutes of
compute, and (3) reports a **behavior-level evaluation** — side-by-side generations and a
win-rate on a held-out set — as a first-class output alongside the reward curves.

**Customer quote (imagined).** "I went from never having run RL to a tuned model the same afternoon — and for the first time I could actually tell whether it improved, instead of trusting a curve."

**How it works.**
- Auto-configures memory-bound settings (batch / accumulation / length / checkpointing) from model + GPU.
- Recommends `beta` / learning rate / LoRA rank with guided defaults and a fast preview run.
- Runs held-out generation + win-rate evaluation automatically and flags regressions.
- Versions the whole environment so a working run stays working.

**Availability.** Concept only - this repo is the customer-discovery work behind it.

---

## 

**Why is this hard today?**
Four recurring walls, all of which I hit by hand: (1) **memory** - defaults don't fit
real GPUs and the fixes aren't discoverable; (2) **blind hyperparameters** - no feedback
until a full run finishes; (3) **version fragility** - pinned TRL/transformers/peft, with
silent breakage otherwise; (4) **evaluation ambiguity** - the trainer reports reward
curves, but nothing tells you the model got *worse* on factuality.

**Who is this for?**
ML and applied teams that want to customize a model's behavior but don't have a dedicated
RL-infrastructure group - from startups to enterprise application teams.

**How would we measure success?**
Time-to-first-successful-run; share of runs that converge without manual config; share of runs where a regression is caught *before* the model is shipped; repeat usage. Notably *not* "did the job complete" - completion is the low bar that hides the real failures.

**What would you NOT build, and what are the risks?**
Don't over-automate the choices that carry real trade-offs — `beta` is the clearest
example: in my runs, a higher value produced a larger (partly mechanical) reward margin
but a *worse* model, so a system that silently "optimizes the margin" would ship
regressions. And the evaluation has to reflect real behavior; automating a reward curve
without behavior-level checks just automates a false sense of progress.

**What did doing this by hand teach you that a spec wouldn't?**
That the gap between "the reward curve went up" and "the product is better" is the entire game. My stronger-looking run was the weaker model, and the only thing that revealed it was reading the outputs. Tooling that surfaces only reward curves doesn't just omit a feature — it quietly lets customers ship worse models believing they improved.
