# Making RL Fine-Tuning Accessible — a working-backwards sketch

*A one-page PR/FAQ written in the Amazon working-backwards style, grounded in the
friction I hit doing DPO by hand (see the [README](../README.md)). This is a thinking
artifact, not a real launch.*

---

## Press release (mock)

**Headline:** Teams can now align and customize a foundation model with reinforcement
learning in an afternoon — no infra tuning, no guesswork.

**Subhead:** A managed RL fine-tuning workflow that auto-configures training, guides
the key choices, and tells you whether the model actually got better.

**The problem.** Reinforcement learning is now central to how capable models are built
— aligning to preferences, optimizing reasoning, customizing behavior. But for most
teams it's out of reach: it demands deep expertise, brittle infrastructure, and days
of trial and error. The first thing a newcomer hits isn't a research question — it's
an out-of-memory error. [FILL: tie to your own OOM/version experience]

**The solution.** [FILL: describe the product in 2–3 sentences — e.g., a workflow where
you pick a base model and a preference dataset, and the system selects a safe training
configuration for your hardware, surfaces an early-signal run before committing compute,
and reports a behavior-level evaluation, not just reward curves.]

**Customer quote (imagined).** "[FILL: e.g., 'I went from never having run RL to a
preference-tuned model the same afternoon — and I could actually tell it improved.']"

**How it works.** [FILL: 3–4 bullets mapping to your friction → opportunity table.]

**Availability.** [FILL]

---

## FAQ

**Why is this hard today?**
[FILL: summarize the four friction points from your README — memory/config, blind
hyperparameters, version fragility, no behavior-level eval — in your own words.]

**Who is this for?**
[FILL: e.g., ML teams that want to customize a model's behavior but don't have a
dedicated RL infra team — from startups to enterprise app teams.]

**How would we measure success?**
[FILL: e.g., time-to-first-successful-run; share of runs that converge without manual
config; adoption / repeat usage; not just whether a job completes.]

**What would you NOT build / what are the risks?**
[FILL — shows judgment: e.g., over-automating hyperparameters can hide the tradeoffs
that matter; pushing alignment too hard (low beta) degrades the base model; the eval
has to reflect real behavior, or you've automated a false sense of progress.]

**What did doing this by hand teach you that a spec wouldn't?**
[FILL: your most honest takeaway — e.g., the gap between "the reward curve went up" and
"the product is better" is the whole game, and tooling that only shows reward curves
quietly lets customers ship worse models.]
