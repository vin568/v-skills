---
name: autopilot-grilling
description: "Autopilot variant of grilling for automated pipelines (自动驾驶 / 低打扰): settle a plan's design tree with as few user interactions as possible, auto-deciding questions that pass a confidence checklist, logging every auto-decision for audit and override, and asking the user only what cannot be safely decided. Invoke explicitly."
disable-model-invocation: true
---

# Autopilot Grilling

Settle a plan, decision, or idea the way a grilling interview does, as a **design tree** worked in **rounds**, but tuned for automated pipelines where every user interaction is expensive and unwelcome. Decide everything you can safely decide yourself, log each of those decisions so it can be audited and overridden, and put to the user only the questions you cannot safely answer.

Goal: **fewest possible user interactions, zero silent assumptions.**

## Design Tree and Frontier

Map the plan as a design tree: every decision branches into the decisions that hang off it. The **frontier** is every decision whose prerequisites are already settled, the questions you can resolve _now_ without guessing at answers you haven't got yet.

## Facts Are Yours

Finding _facts_ is your job, never the user's. Before classifying a question, exhaust lookup: repository files, docs, configs, existing conventions, git history, and everything the user already said in this conversation. When available, dispatch sub-agents with fresh context windows for exploration. Don't block on them: a running exploration is an unsettled prerequisite, so only questions downstream of it wait.

Facts are what make auto-decisions safe. A missing fact you could look up is never a reason to ask the user.

## Classify Every Frontier Question

The bar is "≥90% confident", and this checklist defines it. Self-rated confidence does not count.

**Must ask.** If any of these hold, ask the user however confident you feel. This list overrides everything below:

- Irreversible or destructive: data loss, deletion, migrations, published releases, force pushes.
- Product, business, or personal preference with no evidence in the context.
- External contract: public API, schema, protocol, or file format others consume.
- Security, permissions, credentials, cost or billing.
- Conflicting evidence: two conventions, docs contradicting code, the user saying two different things.
- The user's own stated intent is ambiguous on this point.

**Auto-decide.** Otherwise, decide it yourself if at least one of these holds:

- The answer follows from facts you found (repo, docs, existing conventions, the user's earlier statements).
- Only one option is actually viable.
- It is the established industry default and nothing in the context argues against it.
- It is cheap and reversible to change later.

**Neither** → ask.

## The Loop

1. Compute the frontier.
2. Classify each frontier question. An auto-decided question is settled immediately: log it (see [Decision Log](#decision-log)) and recompute the frontier. Repeat until the frontier holds only questions to ask, or is empty.
3. If questions remain, ask them **all in one round**. A question that depends on another question still open in this round may ride along as a conditional follow-up: assume the open question's recommended answer and phrase it as "If Q1 = yes: …". Only follow-ups that would themselves be asked ride along; auto-decidable ones wait until their prerequisite is answered. Go one level deep only, never conditions on conditions.
4. Apply the answers, including any overrides of auto-decisions. An override or a rejected recommendation unsettles everything downstream of it: re-classify those decisions and drop conditional follow-ups whose assumption failed. Recompute the frontier and loop.

**Interaction budget: at most 2 asking rounds.** Once the budget is spent, settle everything still open with your recommended answer and log each one under `⚠️ Needs review`.

**Headless runs.** If the caller declares the run non-interactive, or the environment offers no way to get a reply, never wait. Settle every question with your recommended answer and log every must-ask question under `⚠️ Needs review`.

## Asking a Round

Every asking round carries the current decision log, so the user can veto an auto-decision in the same reply and no extra round is spent.

If a structured question tool is available, use it: one item per question, with your recommendation marked as the default. Emit the decision log as text before calling the tool. Otherwise, format the round as text:

```
<decision log so far>

❓ **Q1** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>

---

❓ **Q2** - **<question title>**: If Q1 = yes: <question body>

➡️ <your recommended answer>

---

Reply with answers or overrides (e.g. "all yes except Q2; A3 → X"). Auto-decisions not mentioned stand.
```

Word each question so "yes" accepts your recommended answer. Number Q and A IDs continuously across rounds and never renumber, so overrides stay unambiguous.

## Decision Log

```
### ✅ Auto-decisions
- **A1** - **<title>**: <decision>
  - Basis: <fact, file, convention, or user statement relied on>
  - Rule: <which auto-decide criterion it met>

### ⚠️ Needs review
- **A7** - **<title>**: <decision>
  - Why flagged: <must-ask item settled because budget spent / headless run>
```

Omit the `⚠️ Needs review` section when it is empty. In the final output, put it first when it is present.

## Finish

The session is done when the frontier is empty: every branch visited, every decision either answered by the user or logged, nothing silently assumed. There is no separate confirmation round. The last asking round already carries confirmation through "Auto-decisions not mentioned stand." If no question ever needed asking, finish without contacting the user.

Final output, as the hand-off to the next pipeline step:

1. The settled design, stated concisely.
2. User decisions: `Q#`: answer.
3. The full decision log.

Act on the settled design only if the caller's task includes acting on it.
