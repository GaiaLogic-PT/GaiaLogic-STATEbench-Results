# A Small Model and a Stack of Rules: Our Run at STATE-Bench

There is a benchmark out of Microsoft called [STATE-Bench](https://github.com/microsoft/STATE-Bench),
and before we tell you what we did with it we should tell you, plainly, what it
is. Picture a customer on the other end of a chat — only the customer is not a
person, it is a language model playing a person, patient and sometimes difficult,
asking to cancel an order, to match a price, to rebook a flight, to return a pair
of shoes. The agent being tested sits across from that simulated user with a set
of tools — look up the order, check the policy, issue the refund — and it has to
listen, gather the facts, propose the right action, and only then act. When the
conversation ends a third model, the judge, reads the whole transcript and asks
two quiet questions: did the right things actually change in the world (the
*state* requirements), and did the agent do what the user needed (the *task*
requirements). It also rates the experience. Three domains — customer support,
travel, shopping — fifty held-out tasks each, run five times over, scored by
GPT-5.4 acting as both the user and the judge.

That is the thing. It is harder than it sounds, because the tasks are built to
punish the agent that acts too soon, discloses the wrong number, skips the policy
gate, or confuses one customer's order for another's.

And here is a detail worth saying out loud: STATE-Bench was really designed to
test *large, complex agent architectures* — systems with memory, with planning
loops, with the whole apparatus that lets an agent learn across tasks and carry
what it learned forward. The Agent Learning Track, the one we entered, rewards
exactly that kind of learning and remembering.

## What we built

We went the other way. We built something small, on purpose, and we called it
**gaia-nano**.

The whole system runs on one model, `gpt-5.4-nano` — a small one, nothing larger
anywhere in the pipeline. There is no second model doing the thinking, no big
planner behind the curtain. The learning part uses no language model at all: we
read the *training* conversations the benchmark provides, the ones the agent is
allowed to see, and we mine them — deterministically, by hand of code, not by
prompting — into plain human-readable rules. If/then rules. The preview-then-
confirm discipline. The order you consult the tools in. The policy branches the
environment itself hands you. Then at test time the little agent carries that
frozen rulebook in its pocket and nothing more; a thin retrieval hook slips it the
three most relevant pages for whatever the user just said.

No ground-truth labels ever touch the agent or the learner. Not from the training
set, not from the test set. That boundary is enforced in code and machine-checked:
the training examples our rules lean on have zero overlap with the test tasks. We
wrote the proof down and shipped it alongside the results so anyone can check.

We are submitting two versions, and the difference between them is a matter of
honesty more than cleverness:

- **gaia-nano** — our stronger run. It is label-free, but we will not pretend it
  never saw the test set. An earlier iteration was run against the test tasks, our
  engineers read those results, and from reading them we changed the *design* —
  mostly prompt and conduct-rail wording. No labels, no test fields, ever went
  into the machine. But the design was informed by looking. We say so plainly.
- **gaia-nano-v3** — the clean one. Rules authored from a held-out slice of the
  training data only, frozen before any evaluation, any judge, any simulator call,
  and never touched again. It never saw the test set in any way. It is the honest
  baseline: what a small model learns purely from reading allowed demonstrations.

## The results

Five runs per task, fifty tasks per domain, judged by the locked GPT-5.4. *pass@1*
is how often a single attempt fully passes, averaged across the runs. *pass^5* is
the strict one — all five attempts pass, the measure of consistency. *UX* is the
judge's rating of the interaction, on its own one-to-five-ish scale.

| System | pass@1 | pass^5 | Mean UX |
|---|---|---|---|
| **gaia-nano** (design-informed by test) | 0.64 | 0.39 | 3.71 |
| **gaia-nano-v3** (never saw the test set) | 0.62 | 0.38 | 3.68 |

Per domain, for the curious:

| Domain | gaia-nano (pass@1 / pass^5 / UX) | gaia-nano-v3 (pass@1 / pass^5 / UX) |
|---|---|---|
| customer_support | 0.61 / 0.38 / 3.92 | 0.57 / 0.32 / 3.87 |
| travel | 0.65 / 0.32 / 3.40 | 0.66 / 0.34 / 3.38 |
| shopping_assistant | 0.66 / 0.48 / 3.82 | 0.64 / 0.48 / 3.79 |

Read those numbers soberly. The agent solves somewhere around six in ten tasks on
a single try, and it is consistent across all five tries on roughly four in ten.
That is decent work for a small model carrying nothing but a rulebook. The most
interesting line is the gap that is barely a gap: the clean, never-looked version
trails the design-informed version by about two points on average, and on travel
it actually edges ahead. The looking bought us a little. Honesty cost us almost
nothing.

## Where that sits on the leaderboard

STATE-Bench keeps a public
[leaderboard for the Agent Learning Track](https://microsoft.github.io/STATE-Bench/leaderboard/?track=agent-learning),
and it is worth laying our numbers beside the ones already verified there —
carefully, because versions matter. The benchmark has moved through releases, and
only entries on the same version, v0.8.0, make a fair side-by-side. Here are the
verified Agent Learning Track entries on v0.8.0, with ours beneath them:

| System | Organization | pass@1 | pass^5 | UX | Status |
|---|---|---|---|---|---|
| GPT-5.4 + Foundry Memory | Microsoft Foundry | 54.5% | 33.6% | 3.75 | verified |
| GPT-5.4, no memory (baseline) | OpenAI | 51.3% | 29.9% | 3.31 | verified |
| **gaia-nano** (ours) | GaiaLogic | **64.0%** | **39.3%** | 3.71 | not verified |
| **gaia-nano-v3** (ours) | GaiaLogic | **62.3%** | **38.0%** | 3.68 | not verified |

Read that the right way. On the same benchmark version our runs land above the
verified entries — above a full-size GPT-5.4 carrying a dedicated memory system,
above the no-memory GPT-5.4 baseline — and from a model a fraction of the size. For
a wider bearing: the strongest full-size model on the separate Main Track, GPT-5.5
at high reasoning, sits at 58.9% pass@1 on the same version.

Now the honesty, because this is the part that matters. A nano model sitting at the
top of a leaderboard built for heavy architectures is exactly the kind of result
that should make you raise an eyebrow — us included. These are our own runs on the
locked protocol, scored by the benchmark's own judge; they are **not yet verified
by Microsoft**. Extraordinary numbers earn extraordinary scrutiny, and we would
rather you hold them as a claim awaiting a check than as a trophy on the shelf. And
even taken at face value the consistency line is sobering: a pass^5 near 39% means
that on something like six tasks in ten the agent still slips on at least one of
its five tries. The ceiling is a long way up.

## What it means, and what it doesn't

We think the quiet result here is the one worth keeping. A benchmark built for big
architectures with memory can be met — and, on our own unverified runs, met with a
little room to spare — by a small model and a stack of rules you can read with your
own eyes. That is not a claim that the small way is better; a single benchmark does
not settle that. It is a claim that the small way is cheap, auditable, and further
along than you might expect — and that there is clearly more road left, in the six
tasks of ten the strict measure still misses.

## Links

- **Full submissions, scored trajectories, and compliance documents:**
  [GaiaLogic-STATEbench-Results](https://github.com/GaiaLogic-PT/GaiaLogic-STATEbench-Results)
- **Our official submission thread (open, unverified):**
  [microsoft/STATE-Bench · issue #51](https://github.com/microsoft/STATE-Bench/issues/51)
- **The benchmark itself:**
  [microsoft/STATE-Bench](https://github.com/microsoft/STATE-Bench)

---

*Status: both results are **submitted** to the STATE-Bench Agent Learning Track and
are **currently under verification**. Until Microsoft confirms them, treat the
numbers above as self-reported on the locked protocol — published in full so they
can be checked, and awaiting that check.*
