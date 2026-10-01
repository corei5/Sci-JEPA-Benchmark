# Sci-JEPA Benchmark — Experiment Plan

Sep 28, 2026 · @Gollam

## The whole thing on one page

Read this. Everything after it is reference — open a section when you reach that part of the work.

**The problem.** Our pilot model passed every standard check and learned nothing. Probe accuracy 0.871, healthy rank, converged loss, and it recovered 0 of 14.379 bits. Accuracy alone cannot tell a model that followed the rule from one that followed a shortcut, because on normal data the two agree.

**The idea.** Build small scientific worlds where we plant the rules ourselves. Then we know the right answer *and* the shortcut's answer for every item. Give each item three versions: shortcut agreeing, shortcut flipped, shortcut removed. If a model's accuracy falls when we flip the shortcut, it was using the shortcut. That drop is the measurement nobody has today.

**The numbers.** 80,000 items, 4 subject areas, 5 question types (predict, design, explain, compare, intervene), 5 shortcut types (identity, words, frequency, nearness, type), each with a strength dial. Three splits: 50k to build with, 20k hidden, 10k for strength sweeps.

**The schema.** Nine unit types — Problem, Idea, Claim, Hypothesis, Method, Material, Result, Metric, Condition — each with a SHACL shape that fixes its properties and units. Papers link through `reuses`, `comparesTo`, `replicates` and `contradicts`. These links depend on content, not node type, which is what the pilot's graph got wrong.

**The order of work.** Decide → build the generator → pass ten gates → validate small → train → sweep → write. The gates are checks on the *data*, run before any training. This order is the single most important thing in the document: every expensive mistake in the pilot came from training first.

**The result we expect.** Models that look identical on accuracy will differ on rule-following, and no standard check will predict the gap.

**The dates.** Roughly sixteen weeks, against a projected ICML deadline of late January 2027. Verify the deadline on the official call.

| If you want | Go to |
| --- | --- |
| The paper's argument | The paper in one page |
| What to build | The generator, and the five reasoning tasks |
| What to check before training | The eight gates, plus G9 and G10 |
| What to measure | How we score things |
| What to do this week | The task list document |

## Two axes, not one

Every item in this benchmark is built with a known answer and at least one planted trap. That lets us score a scientific world model, in our case Graph-JEPA, on two separate questions instead of one:

- **Axis 1 — Is it right?** Standard accuracy. This is all that normal benchmarks measure.
- **Axis 2 — Is it right for the right reason?** How well the model holds up when the shortcut points somewhere else. Nobody measures this today, and that is why models can pass while learning nothing real.

Our pilot paper is the reason we need axis 2. A Graph-JEPA looked healthy on every usual check: probe accuracy 0.871, effective rank in a normal range, loss converged. But it recovered 0 of 14.379 recoverable bits. Nothing. No standard check noticed. Axis 2 is what would have noticed.

**What the paper must be able to say at the end:**

1. For each of the five trap channels, we can say at what dose a model stops following the planted rule and starts following the trap.
2. Two models that look the same on axis 1 can be far apart on axis 2, and nothing in the standard toolkit (accuracy, effective rank, linear probe) predicts the gap.
3. What we measure in the miniature worlds also predicts how models behave on real data.

Point 1 is the smallest publishable result. Point 2 is the contribution. Point 3 is what stops a reviewer dismissing the whole thing as synthetic.

**What we do not claim.** That any model here does real scientific reasoning, that our planted rules look like real science, or that a good axis-2 score says anything about real papers. The pilot was strict about this and we stay strict.

## The paper in one page

**Working title.** *Right for the Wrong Reason: a generative benchmark for shortcut-free scientific representation learning.* Two alternatives if that reads too cute: *Measuring rule-following in self-supervised scientific world models*, or *When every health check passes and nothing was learned*.

**The thesis, in one sentence.** Self-supervised graph models can saturate every standard health check while carrying no usable information about the thing being evaluated; we prove why the loss cannot detect this, and we build a generative benchmark that measures rule-following directly, across five reasoning tasks and five shortcut channels.

**The story the paper tells, in four steps:**

1. **The problem is real and invisible.** Our pilot trained a Graph-JEPA that passed every check and recovered zero of 14.379 bits. We show this is not a bug in the training: the degenerate configuration is a global optimum of the objective. No loss-based criterion can flag it.
2. **Current benchmarks cannot see it.** Accuracy alone cannot separate a model that followed the rule from one that followed a shortcut, because on ordinary data the two agree. And a near-ceiling score can be worthless if the evaluation target is reducible, which we prove happens on a real scholarly graph.
3. **So we build worlds where we know the answer.** A generator produces miniature scientific worlds with planted rules, five toggleable trap channels at controlled doses, and every item in three versions: trap agreeing, trap flipped, trap removed. That design makes rule-following measurable rather than inferred.
4. **And then we measure.** Five model families, five reasoning tasks, five channels. We report, for each, the dose at which the model stops following the rule and starts following the trap. We show accuracy and rule-following come apart, and that no standard diagnostic predicts the gap.

**Why a reviewer will have trouble dismissing it.** Each of the usual objections already has an answer built into the design:

| The objection | The answer in the paper |
| --- | --- |
| "Maybe the task is just hard" | A training-free oracle and a rule-aware oracle bracket the difficulty |
| "Maybe your harness is broken" | A positive control recovers the signal through the identical code |
| "Maybe it is your model only" | Five model families, including three standard non-contrastive methods |
| "Maybe your traps are not really traps" | A trap-only predictor scores exactly at the dose, by construction |
| "It is synthetic, so who cares" | 300 hand-labelled real contributions, and the profiles are compared |
| "You tuned until it worked" | Every threshold pre-registered and written to disk before generation |

**What makes it an ICML paper rather than a resource paper.** The theory. Three formal results, two reused from the pilot and one new, say *why* the failure exists, *why* standard criteria cannot see it, and *why* this design can. The dataset is how we demonstrate the theory, not the contribution by itself.

## Six choices to make in week 1

These six choices change what the data looks like. Once the data is built, changing them means building it all again. So decide first.

| # | Choice | Options | What I suggest |
| --- | --- | --- | --- |
| D1 | What the model is asked to do | Find the hidden piece (same as our last paper), or pick between two answers | Do both. The first keeps us comparable, the second is better for measuring honesty |
| D2 | Do we publish the rules? | Publish everything, or publish most and keep one rule family secret | Publish most, keep one family secret |
| D3 | How to use the 4 subject areas | Mix them together, or train on 3 and test on the 4th | Train on 3, test on the 4th. It is the cheapest answer to "you only used one dataset" |
| D4 | Where the words come from | Copy real word frequencies from SciKU, or make up words | Copy real ones. Made-up words make the text look fake |
| D5 | How hard the task should be | Let it come out however it comes out, or set a target in advance | Set it in advance: a no-training baseline must score under 60% |
| D6 | Which Sci-JEPA to use | The old one, or the fixed one from our last paper | The fixed one. The old one becomes a control |

**D5 is the one that will bite.** In our last paper, a baseline that did no training at all already got 96.4% of the best possible score. That left 0.514 bits for every model we ever trained. A rule-based generator will do the same thing unless we control how much the visible parts give away, and check it before training. If we find this out after building 80,000 items, we build them all again.

**D1 matters more than it looks.** If the task is too easy, nothing can show a difference. That is why one of our earlier effects shrank to +0.071 bits: there was simply no room left to move. A pick-between-two format does not get easy in the same way, because we build the wrong answer ourselves.

## The key property: what we know about every item

For every item in the benchmark we know three things:

- **(a) The true answer.** The generator planted the rule, so the answer is not a label someone guessed.
- **(b) Which traps are planted, and at what dose.** Five channels, each with its own strength dial, set per item.
- **(c) What a trap-following model will answer.** We cannot know what a given model will do, but we can know what each planted trap points at. That is the number we store, and it is what makes axis 2 scoreable.

Property (c) is the one that does the work. Knowing the trap's answer as well as the true answer means we can tell, for every prediction, which one the model followed.

**Each item ships in three versions.** Axis 2 cannot be measured from one answer on one item. It needs the same item with the trap moved:

- **Normal** — trap and rule agree. Accuracy here is axis 1.
- **Flipped** — the trap points at a different answer; the rule and true answer are unchanged. Accuracy here is axis 2.
- **Blank** — the trap is removed. Shows whether the model needed it at all.

How much a model leans on a channel is accuracy on normal minus accuracy on flipped. Same items, same rule, only the trap moved, so the comparison is clean.

**What each item record stores.** Item id, domain, the contribution subgraph, the rule instance, the true answer, the dose of each of the five channels, the trap-implied answer for each channel, which version it is, which triple it belongs to, which split it sits in, and the generator seed. For the holdout, rules and answers live in a sealed file.

**One number to settle in week 1.** Does 80k mean 80,000 contribution subgraphs, or 80,000 scored instances? The plan assumes 80,000 subgraphs, giving up to 240,000 scored instances. Generating the extra versions is cheap and training only reads the base subgraphs.

## The generator: miniature scientific worlds

The generator builds small scientific worlds where we control everything. Vocabulary follows real SciKU word statistics, rules are planted by us, and each trap channel can be switched on at any dose. Output is roughly 80,000 contribution subgraphs across four domains.

**The schema.** Every unit is one of nine types: Problem, Idea, Claim, Hypothesis, Method, Material, Result, Metric, Condition. Each is governed by an ORKG template, a SHACL shape fixing required properties, cardinalities, class ranges and units. Unlike open information extraction, every unit has a schema to validate against, so conformance is a machine check rather than a judgement. Across papers, units are joined by `reuses`, `comparesTo`, `replicates` and `contradicts`.

This choice does more than tidy the data. Those four cross-paper relations depend on what the units actually say — whether one method genuinely reuses another, whether two results genuinely agree. That is content, not node type, which is exactly what R2 requires and what the pilot's graph lacked. The schema is what lets Gate G1 pass rather than fire.

The matching risk is that a generator can reintroduce the problem in disguise: plant `replicates` for every pair sharing a Method type and its count becomes a function of the census again. Every cross-paper relation must be conditioned on the values inside the units, never on their types, and G1 is re-run after every generator change.

**The one thing the generator must get right.** The edges inside a subgraph have to carry information. If every subgraph of a given shape always has the same edges, those edges tell you nothing and the graph is decoration.

**This is where the pilot corpus failed.** Five of nine relations had a count exactly equal to an endpoint node count: `produces` appeared 57,903 times, once per paper; `grounds` 251,938 times, once per claim. So the edge set never varied. Message passing could only mix each subgraph's own text, which is exactly what a parameter-free average already does with no training.

**How the generator avoids it.** An edge exists because of a latent fact about that item, never because of node type alone:

- A `grounds` edge exists only where the evidence really satisfies the rule's premise.
- A `challenged_by` edge exists only where the claim really violates a planted rule.
- Cross-subgraph edges come from a latent topic process, so they differ item to item.

The test is one line of arithmetic: compare each relation's count to its endpoint node counts. Equal means informationless. This is Gate G1, and this time it has to **pass**.

**The planted rules.** Simple if-then templates, instantiated per domain. Three families, plus a fourth kept back for the holdout:

1. **Compositional** — a property of the whole follows from properties of its parts, above a threshold.
2. **Conditional-exception** — the rule holds unless a named condition is present, so the model must notice an absence.
3. **Transitive chain** — the answer needs two or more edges, so one hop provably cannot be enough.
4. **Hidden family** — a fourth shape, released only after review, so the benchmark cannot be gamed by fitting the published grammar.

Every item records its **minimum hop depth**: how many edges you must follow to derive the answer. Report results split by it. If every item is one hop, the benchmark is a lookup table, and hop depth is the cheapest proof that it is not.

**Vocabulary and surface form.** Names are sampled to match real SciKU unigram and bigram statistics, so the frequency channel means something. Sentence patterns vary per aspect, and we measure how templated the text actually is rather than assuming it is fine.

## The five reasoning tasks

The benchmark asks five kinds of question. Each is a different kind of scientific thinking, and a model can be good at one and bad at another, so we report them separately.

| # | Task | The question | What the model gets, and what it must produce |
| --- | --- | --- | --- |
| T1 | Forward prediction | What will happen next? | Given a method and conditions, predict the result |
| T2 | Experiment design | How can we test this? | Given a claim, pick the method that would actually test it |
| T3 | Abduction | What does this evidence mean? | Given a result, recover the claim it supports |
| T4 | Consistency checking | Do these findings agree? | Given two contributions, say whether they agree, conflict, or are unrelated |
| T5 | Intervention | What if we change something? | Given a contribution and one changed condition, say how the result changes |

**How this fits with everything else.** These five are a separate axis from the rule families above. A rule family is the *shape* of the hidden rule; a task is the *question we ask about it*. So every item carries three labels: which task, which rule family, and which trap channels at which dose. All five tasks are scored on both axes, so each task gets its own accuracy and its own shortcut-robustness number.

**Budget.** Split the 80,000 subgraphs evenly: 16,000 per task, so 4,000 per task per domain. Keep it even even if some tasks turn out easier, because uneven splits make per-task comparisons hard to defend.

**T5 is the one only a generator can give us.** Intervention needs the same world re-run with one condition changed, and the true answer for both versions. No real corpus can supply that, because the alternative experiment was never performed. Our generator can, because it owns the rules. This is the strongest argument for building synthetic worlds at all, and it belongs in the introduction rather than buried in a table.

**T4 needs care.** Consistency checking needs pairs that genuinely conflict under a planted rule, not pairs that merely use different words. In the pilot corpus the contradiction field turned out to be 25.96% identical placeholder text, and a model trained on it would have learned to spot filler rather than disagreement. Here we plant the conflict ourselves, so that problem does not arise — but keep the duplicate and genericness checks running on T4 items anyway, as a guard against the generator producing bland near-copies.

**Expect the tasks to behave differently.** T3 is closest to the pilot's masked-aspect task, so expect high accuracy and little headroom. T1 and T5 need the rule followed forward, so they should be the hardest to shortcut and the most informative on axis 2. If all five give the same numbers, that is itself a finding: it means the tasks are not as distinct as we think. Check it before writing the results section.

## The five trap channels and their doses

Each channel is toggleable and has a dose, drawn independently per item. Dose means: how often the trap happens to point at the true answer. At 0.5 the trap is useless, at 1.0 it is a perfect shortcut.

| Channel | The shortcut it offers | Dose means | Why it is in the set |
| --- | --- | --- | --- |
| Identity | A token that occurs only with this item | How often that token predicts the answer | Catches memorisation and leakage |
| Lexical | Query and gold share surface words | Shared n-gram mass | In the pilot, BM25 alone reached 99.7% of ceiling |
| Frequency | Frequent entities are more often the answer | Planted rank correlation | Catches a popularity prior posing as inference |
| Proximity | The gold node is also the graph-nearest one | How often nearest equals gold | Catches topology standing in for the rule |
| Type-collapse | The answer follows from aspect or node type alone | How often type implies the answer | The exact failure our pilot proof describes |

**Type-collapse is the null hypothesis, not just a fifth channel.** Our pilot proved that when the aspect designator determines the category, ignoring instance identity is not a mistake, it is a global optimum of the objective. So a share of items must sit at dose 0 on this channel, with a designator that genuinely does not give the category away. Otherwise every item is solvable the degenerate way and the other four channels cannot be measured at all. Handle this channel separately in the paper and explain why.

**Channels must be separable.** If identity and lexical nearly always fire together, no result can be attributed to either. Two mechanisms:

- Doses drawn independently, with a pre-registered ceiling of 0.1 on the largest pairwise correlation between channels (Gate G4).
- A reserved **single-channel stratum**: 40% of items have exactly one channel at nonzero dose. Multi-channel items show interactions; the single-channel items carry the attribution.

**A risk the five channels create.** A model could learn to detect that a trap has been planted rather than learn the rules. Test it directly: can a classifier predict which channel is planted, from the item text alone? A high reading means the traps are templated and the doses do not mean what we think. Run it with the gates and report the number.

## The three theory results

This is what lifts the paper from a resource to an ICML submission. Two results come from the pilot and are reused; the third is new and belongs to this paper. Each is stated here in plain words, with the formal version going in the paper.

**R1 — The failure is a global optimum, not a training bug.** When the target is produced by a trailing copy of the encoder being trained, a model that encodes only the category and throws away the instance reaches loss zero. So does a model that keeps the instance. Both are global optima of the same objective, and they differ by the entire recoverable budget. *What it buys:* no loss value, and no criterion computed from the loss, can tell you which one you got. This is why a converged run with a healthy probe can be worthless, and why the fix has to change what the objective asks for rather than how it is optimised.

**R2 — A near-ceiling score can prove nothing.** If a graph's edges are a deterministic function of its node census, the edge set carries no information, and message passing can only recombine each subgraph's own features. *What it buys:* a model scoring 99% on such a benchmark has learned a better metric over features it already had, not structure. We prove the pilot corpus had this property for 5 of 9 relations. It is also why Gate G1 exists, and why our generator conditions edges on latent facts instead of node types.

**R3 — The three-version design identifies shortcut reliance.** This one is new. The claim: if a model's prediction depends only on the rule-relevant part of the subgraph, then its accuracy is unchanged when the trap is flipped. Therefore any drop on flipped items is attributable to trap use, and the size of the drop measures how much.

State the assumptions honestly, because a reviewer will look for the gap:

- The flipped item differs from the normal item **only** in the trap fields. Verify this bitwise, do not assume it.
- The rule and the true answer are identical across the three versions.
- The trap is not itself part of the rule. If a channel is load-bearing for the answer, flipping it changes the answer and the result does not apply. This is a real constraint on the generator, not a formality.

Where it fails is worth saying out loud: if flipping a trap changes the surface text in a way that happens to correlate with the answer, the drop measures that correlation instead. The bitwise check is what rules this out, and it belongs in the self-test.

**One corollary worth stating separately.** The pilot proved that once the query side is degenerate, no transformation applied afterwards can repair it — not whitening, not a learned metric, not a different geometry. Measured: five retrieval frames bought 0.4 bits against a 14.4-bit deficit, while changing the objective moved the entire budget. The practical lesson for anyone using the benchmark is that a low score is not fixed by post-processing, and the paper should say so plainly.

## The three splits

80,000 contribution subgraphs, 20,000 per domain: materials, biomedicine, computer science, environmental.

| Split | Subgraphs | Purpose | What is published |
| --- | --- | --- | --- |
| Model development | 50,000 | Building and tuning models | Items, doses, rule grammar, trap-implied answers |
| Holdout | 20,000 | The score that counts; cannot be gamed | Items only; rules, doses and answers sealed |
| Dose sweeps | 10,000 | Finding each channel's tipping point | Items and doses; single-channel only |

**The holdout is where hidden rules do the work.** Two things change relative to development:

- One rule family is withheld entirely, so a model that fitted the published grammar has nothing left to fit.
- Doses are re-randomised, so traps no longer point at the answer. A model leaning on shortcuts at 0.95 development accuracy should land near chance here.

If no configuration in development shows that drop, the benchmark is not discriminating and we rebuild before writing anything up. Make it an explicit checkpoint, not something noticed afterwards.

**The dose sweeps.** Single-channel items only, on a dense grid: 11 doses per channel from 0.5 to 1.0 in steps of 0.05, roughly 180 items per channel per dose per domain. These produce the paper's central figure.

**Domains as a transfer axis.** Train on three domains, evaluate on the fourth, rotating all four ways. Four training runs instead of one, and the cheapest available answer to the single-corpus limitation the pilot had to concede. It also shows whether channel effects hold across fields or are artefacts of one.

## Eight checks that run before any training

No training starts until all eight checks pass on the built data. The pass marks are written down before the data is made, and never changed afterwards.

| Gate | What we measure | Pass mark | What happened last time |
| --- | --- | --- | --- |
| G1 | Do connection counts match node counts? | No connection type matches | Failed: 5 of 9 connections |
| G2 | How well does a no-training baseline do? | Under 60% of the best score | It got 96.4% |
| G3 | Can a model that knows the rules solve it? | 95% or better | Our control got 98.9% |
| G4 | Do any two traps travel together? | Under 0.10 | New check |
| G5 | How much of the target text is copy-pasted? | Under 5% | Failed: 25.96% was filler |
| G6 | Is one class of text blander than the other? | Gap under 0.15 | Failed: gap of 0.24 |
| G7 | Can you guess the type from filler words alone? | Under 0.75 | It was 0.937, versus 0.333 by luck |
| G8 | Can a classifier spot which trap is planted? | Under 0.50 | New check |
| G9 | Do all units conform to their SHACL shape? | 100%, by construction | New check |
| G10 | Are units and magnitudes legal and in range? | 100% of Metric and Condition units | New check |

**The order is the whole point, and it is our own lesson.** The two checks that changed our last paper's conclusions took under a minute between them. The experiments they overturned took 4.6 GPU-hours. Every check above is cheap. Run them after training and a finding turns into an excuse.

**When a check fails.** That is a bug in the generator, not a result to work around. Fix it, rebuild the data, run all eight again. Write down every failure and what we changed. That list is worth publishing: it shows the checks actually do something.

**Watch out with G2.** The no-training baseline is just the average of the visible text, exactly as in our last paper. It has to be measured with the same code that will later score the models. If the check uses different code from the evaluation, the comparison means nothing.

## Training Sci-JEPA, and the controls that validate the benchmark

Sci-JEPA is our existing Graph-JEPA pipeline, trained on the four-domain dataset with no architectural changes. What changes is the data it trains on and the way it is scored. The controls below run first, because the benchmark is only trustworthy if it gives the expected verdict on systems whose behaviour we already know.

| System | Role | Expected axis 1 | Expected axis 2 |
| --- | --- | --- | --- |
| Sci-JEPA, repaired objective | The model under study | High | This is the measurement |
| Sci-JEPA, regression loss | Negative control | Near chance | Near chance |
| Training-free oracle | Headroom reference | Under 60% of ceiling | Follows traps by construction |
| Rule-aware oracle | Solvability reference | Over 95% | Perfect rule adherence |
| BM25 over raw text | Lexical bound | Moderate | Collapses on flipped lexical items |
| Trap-only predictor | Dose calibration | Tracks the dose exactly | Zero on flipped items |

**The negative control is the validity argument.** In the pilot, changing only the loss moved the pipeline by 14.05 bits with frame, cue, budget, schedule and seed held fixed. If this benchmark does not cleanly separate the repaired pipeline from the regression one on axis 1, it is not measuring anything. The answer is already known, so this tests the instrument rather than producing a finding.

**The trap-only predictor calibrates axis 2.** A system that consults only the planted trap and ignores the subgraph should score exactly at the dose on normal items and zero on flipped ones. If it does not, the doses are mislabelled and the whole sweep is uninterpretable. Same idea as the pilot's cheat query that correctly returned MRR 1.000.

**Nuisance factors get swept, not fixed.** The pilot's largest positive effect was the learning-rate schedule at +1.337 bits, larger than any architectural factor and larger than the effect it was first confused with. Sweep schedule and seeds and report the spread. Better still, make reporting them a condition of submitting to the benchmark.

**Keep a second metric.** Alongside the optimised score, keep a reasoning-relevant probe and audit its floor. Across ten converged cells in the pilot, the two moved independently at Spearman 0.24, n=10, and we drew no directional conclusion. Expect the same here and claim nothing from it.

## How we score things

The accuracy side stays exactly as in our last paper. The honesty side is new, so its formulas need to be fixed before we measure anything.

**Accuracy.** We report bits recovered compared to random guessing, with the exact guessing level and the highest score actually reachable. Do not report ratios against chance: at these sizes they are mostly noise.

**Honesty.** Three numbers, one set per trap:

```latex
\text{leaning} = \mathrm{acc}(\text{normal}) - \mathrm{acc}(\text{flipped})
```

```latex
\text{needs the cue} = \mathrm{acc}(\text{normal}) - \mathrm{acc}(\text{blank})
```

And the **tipping point**: how strong a trap has to be before accuracy on flipped items falls below an agreed line. Fit a curve across the 11 strength levels rather than reading off the nearest point, and give an uncertainty range.

**Uncertainty.** Bootstrap over items, 2,000 resamples. When two systems see the same items, compare them item by item, never with separate ranges.

**Too many tests.** Five traps times four subject areas times several systems adds up fast. Decide in advance which comparisons go in the abstract. Everything else is exploration and must be labelled as such. Reviewers at these venues will check.

**What not to do.** Do not report one headline number per model. Our last paper's whole point was that one number is not enough in either direction. The output here is a profile: five leaning scores, five tipping points, and the drop on the hidden test set.

**Self-test.** Extend the 20 checks from last time with new ones: the shortcut-only guesser scores exactly at the trap strength; the rule-knowing baseline is unaffected by trap strength; the three versions of an item differ only in the trap and nowhere else. Nothing gets measured until these pass.

## Phase plan

&#91;embedded content: 14-week phase plan · 7 phases, 3 gates\]

If a gate fails, the work goes back to the data generator, not forward to training. Each rebuild means running every check again, plus every number that depends on the data. The 12 to 14 week slots assume a deadline about three months away. If time gets tight, cut the writing weeks, never the checking weeks.

## How much computing time this needs

Our last diagnosis took 0.77 GPU-hours and the full set of experiments 4.44. This project is bigger, but the pattern is the same: building and checking the data is nearly free, training is what costs.

| Stage | Estimate | Note |
| --- | --- | --- |
| Building 80,000 items in 4 areas | 3 to 5 CPU-hours | Mostly turning text into vectors |
| Making the extra versions | 2 to 3 CPU-hours | Same step |
| All eight checks | Under 10 GPU-minutes | Runs again after every rebuild |
| Sci-JEPA, 20k steps, 3 seeds | 8 to 10 GPU-hours | Multiplied by the 4 rotations |
| The 4 subject-area rotations | 32 to 40 GPU-hours | The biggest line |
| The failing-model control | 3 GPU-hours | Needed, not optional |
| Strength sweeps | 2 GPU-hours | Scoring only, no retraining |
| Schedule and seed sweep | 6 to 8 GPU-hours | Our own advice says to do this |
| Real-data check | Under 1 GPU-hour | Plus the time to label by hand |

Around 55 to 70 GPU-hours in total. Book one machine for eight weeks, with room for two full rebuilds.

**Rebuilds are the real risk, not training.** Every failed check means rebuilding the data and recomputing every number that came from it. We saw this last time: two sets of results shared exactly the same cached data, which is what made four numbers comparable, and any fix would have forced us to redo all four. Build the pipeline so a rebuild automatically throws away and recomputes everything downstream.

**Label your data.** Give every built dataset a fingerprint, and store that fingerprint with every result file. If a result's fingerprint does not match the current data, the tool should refuse to use it rather than quietly reusing an old number.

## Writing decisions down in advance

Every number that decides pass or fail is written to a file before the data is built, the same way we did last time. That file is never edited afterwards.

**What goes in it:**

- All eight pass marks, and why each one is where it is.
- The list of trap strengths, how flipped items are made, and the line that defines a tipping point.
- Which comparisons will go in the abstract, kept separate from everything we are just exploring.
- The training budget, schedule, seeds, and what counts as a finished run.
- The give-up conditions from the risk table below.

**What we release.** The generator with its seed, the decisions file, the eight checks, the self-test, the raw logs behind every table, and the hidden answers behind a separate door. Last time we also published a table linking every number in the abstract to the table it came from, with each number stored once so the text and the tables cannot disagree. Do that again.

**Publish the failures too.** The most convincing part of our last paper was what we admitted: a number that turned out to be 17 times smaller once measured properly, a self-test that passed because it checked a hard-coded value and so hid a real problem, and the discovery that a quarter of one field was filler text. A benchmark paper that claims everything worked first time reads as either lucky or not curious. Keep a build diary and publish it.

**Version the data.** The benchmark will change after release. Give each version a number, freeze it, and say clearly that scores only compare within one version.

## What could go wrong

| Problem | How we spot it | What we do | When we stop and change plan |
| --- | --- | --- | --- |
| "It is all made up, so who cares?" | A reviewer says it, and we have no answer | Label 300 real items by hand and check our numbers predict real behaviour | No connection at all: drop the generality claim, present it as a diagnostic tool |
| Connections carry no information | Check G1 fails | Make connections depend on real facts, rebuild | Fails three times: the rules themselves are wrong, not the code |
| Task too easy | Check G2 above 60% | Make the answer need more steps | Cannot get under 80%: switch to the pick-between-two format only |
| Benchmark does not tell models apart | Nobody drops on the hidden test | Widen the trap strengths, check flipped items really are flipped | Nothing separates: rebuild before writing |
| Traps are too obvious | Check G8 above 0.50 | Vary how each trap looks | Still obvious: report it as a known weakness, with the number |
| Traps travel together | Check G4 above 0.10 | More single-trap items, redraw strengths | Cannot fix: report per-trap results only on single-trap items |
| Running out of compute | A third full rebuild | Cut from 4 rotations to 2 | A fourth rebuild: shrink to 2 areas and 40,000 items |
| Student stuck on the old code | Week 2 and still no run matching last paper | Debug together against the released code | Week 4: treat it as new code and add 3 weeks |

**Only the first row can actually sink the paper.** Every other row is an engineering problem with a known fix. "It is synthetic" cannot be won by arguing, only by measuring. Plan the hand-labelling from week 1, not during the rebuttal when there is no time left.

**One more habit.** Our most useful corrections last time came from re-reading logs we had already paid for, not from new runs. Once a week, read one finished run's log from top to bottom. The correction that saved us from printing a wrong number in the abstract cost nothing but attention.

## Getting this to ICML 2027 standard

The official ICML 2027 call for papers is not published yet. Deadline trackers project late January 2027, with the conference in July. **Check the official call before planning around any date.** Working backwards from a late-January deadline gives us about sixteen weeks from today.

**ICML has no datasets track.** Unlike some venues, there is no separate place to submit a dataset. That changes what the paper has to be: the contribution is the *measurement framework and the theory*, and the dataset is how we demonstrate it. A paper that reads as "here is a new dataset" will struggle. A paper that reads as "here is why standard evaluation cannot see this failure, here is a proof, here is an instrument that can see it" is a main-track paper.

**The biggest gap right now: we only test our own model.** A reviewer will ask whether this is a property of Graph-JEPA or of self-supervised graph learning in general. We need at least four model families on the benchmark:

| Family | Example | Why it is in the set |
| --- | --- | --- |
| Latent prediction | Sci-JEPA (ours), repaired and broken | The model under study |
| Masked reconstruction | GraphMAE-style | Reconstructs inputs instead of latents |
| Bootstrap / momentum | BYOL-style | Same philosophy, no negatives |
| Redundancy reduction | VICReg or Barlow Twins | The standard anti-collapse approach |
| Supervised GNN | Plain trained classifier | Shows what supervision buys |

If the shortcut reliance appears across all of them, the finding is about the paradigm and the paper is much stronger. If it appears only in ours, that is still publishable but it is a smaller claim, and we should say so plainly.

**Theory is what makes this ICML rather than a workshop paper.** Our pilot already proves two things we reuse: that a category-measurable solution is a global optimum, and that a census-determined graph carries no structural information. Add one new result for this paper: under stated assumptions, the three-version design identifies shortcut reliance, in the sense that no model following the rule can lose accuracy on flipped items. State the assumptions honestly, including where they fail.

**The checklist ICML reviewers actually use.** None of this is optional:

- Error bars on every number, from multiple seeds. A table of single-run numbers gets rejected.
- Code in the supplementary at submission time, not promised for later.
- A real limitations section. Ours is strong already; do not weaken it.
- Broader impact and reproducibility statements.
- Eight pages of main text. Everything else goes to the appendix, which has no limit.
- Full anonymisation, including the repository link.

**Hold one experiment back for the rebuttal.** Reviewers reliably ask for something. The dose–response experiment on real features, which our pilot named but never ran, is the ideal reserve: it is about one GPU-hour, it answers the most likely objection, and it is far more convincing delivered during rebuttal than mentioned as future work.

**Backwards schedule from a late-January deadline:**

| Dates | What happens |
| --- | --- |
| Oct 1 to 10 | Six choices settled, decisions file written |
| Oct 12 to 25 | Generator, rules, five tasks |
| Oct 26 to Nov 1 | Eight gates on a probe pool; fix and rebuild as needed |
| Nov 2 to 8 | Pilot-scale validation; reproduce the old numbers |
| Nov 9 to Dec 6 | Full 80k pool; train Sci-JEPA across 4 rotations |
| Dec 7 to 20 | Dose sweeps; the other model families |
| Dec 21 to Jan 3 | Hand-label 300 real contributions; start writing |
| Jan 4 to 17 | Full draft, all figures final |
| Jan 18 to deadline | Internal review, polish, submission |

The writing weeks are the compressible ones. The gate weeks are not. If something slips, cut the number of model families before cutting a gate.

**One compute warning.** Four extra model families times four domain rotations times three seeds is a large multiplier. Run the full rotation only for Sci-JEPA. For the other families, use two rotations and three seeds, and say so in the paper.

## The paper itself

| Section | What goes in it | Needs which phase |
| --- | --- | --- |
| 1 Introduction | The two axes, the five reasoning tasks, and why intervention needs a generator | Pilot |
| 2 Related work | Shortcut learning, adversarial tests, graph evaluation | Reading |
| 3 The generator | Rules, edges that carry information, SciKU word statistics | Weeks 2 to 3 |
| 4 The five reasoning tasks | T1 to T5, what each asks and how it is scored | Weeks 2 to 3 |
| 5 The five trap channels | Each channel and what dose means | Weeks 2 to 3 |
| 6 The eight gates | What they check, and which ones fired while building | Week 4 |
| 7 Does the benchmark work? | Oracles, the trap-only predictor, the deliberately broken model | Weeks 5, 6 to 9 |
| 8 Sci-JEPA results | Per task and per channel: accuracy, leaning, tipping points, holdout drop | Weeks 6 to 11 |
| 9 Does it match reality? | The 300 hand-labelled real contributions | Week 11 |
| 10 Weaknesses | Synthetic data, planted rules, one generator family | All |

**The main figure.** Accuracy on flipped items against trap strength, one line per trap, with the tipping point marked. It carries the whole contribution in one picture, so give it real design time instead of making it in the last week.

**About the venues.** Please check this year's calls for papers, since tracks change and my information may be out of date.

- **ICML** — best fit for the theory side, which is what makes this different from other shortcut benchmarks. Check whether a datasets track is running; if not, submit to the main track with the measurement idea as the contribution.
- **KDD** — likes benchmarks and resources, and the scientific knowledge-graph angle fits. Expect questions about real-world use, which section 8 answers.
- **IJCAI** — widest audience, least patience for the information-theory details. If we go here, lead with the two questions and move the maths to an appendix.

**One warning about wording.** Saying "for the first time" invites a reviewer to go and find prior work. Similar benchmarks exist in language and vision. What is new here is the combination: a made-up world with known rules, traps with adjustable strength, and three versions of every item, on scientific graphs. Say that, rather than claiming a first.

## How to actually work through this

The plan above is long. Here is how to use it without getting lost.

**The one rule that matters.** Never run a training job on data that has not passed all eight gates. Every expensive mistake in the pilot came from training first and checking afterwards. If you remember nothing else from this document, remember that order.

**What a normal week looks like.**

1. Monday: look at the phase diagram, find where we are, pick the one thing that has to finish this week.
2. During the week: build it, and write down any number you measure, even if it looks wrong. Especially if it looks wrong.
3. Friday: read one finished run's log from top to bottom. Not skim, read. This is where the pilot's two most important corrections came from, and both were free.
4. Friday: 30-minute check-in against the deliverables list.

**How to know a phase is really done.** Each phase has one question it has to answer, and "the code runs" is never the answer:

| Phase | The question it must answer |
| --- | --- |
| Generator | Does Gate G1 pass, so the edges actually carry information? |
| Gates | Did all eight pass on a probe pool, with the thresholds written beforehand? |
| Pilot validation | Do the controls behave the way the theory says they must? |
| Training | Is the broken model clearly worse than the fixed one on axis 1? |
| Dose sweeps | Does each channel have a tipping point with a usable error bar? |
| Calibration | Does the synthetic profile predict anything about the real items? |

**When something does not work.** The order to check things, cheapest first:

1. Run the self-test. It catches most problems in 20 seconds.
2. Check the data fingerprint matches the results you are comparing against.
3. Check whether a gate that passed earlier still passes, since generator changes break things silently.
4. Only then suspect the model.

**Three habits that are worth more than they look.**

- **Write the number down before you interpret it.** A measurement you have already explained is hard to re-examine.
- **A failed gate is a result, not an obstacle.** The pilot's placeholder-text discovery started as an annoying blocked experiment and ended as one of the paper's strongest sections.
- **Say what you do not know.** Every limitation we wrote down in the pilot made the paper stronger, not weaker. Reviewers trust a paper that found its own problems.

**When to come and ask.** Straight away, if: a gate fails three times in a row; a result looks too good (a perfect score is usually a leak, not a success); the timeline slips by more than a week; or you find yourself about to explain away a number instead of investigating it. None of these are things to push through alone.

## Things to finish

- [ ] Week 1 note answering the six choices, agreed before any code
- [ ] Decisions file on disk, with all eight pass marks and the planned comparisons
- [ ] The generator, with a fixed seed, producing items in the agreed format
- [ ] Generator supports all five tasks T1 to T5, including re-running a world for intervention
- [ ] The rule families written up, with the hidden family stored separately
- [ ] The tool that makes the three versions, plus a check that they differ only in the trap
- [ ] All eight gates, running in under 10 GPU-minutes
- [ ] The self-test, extended with the new axis-2 checks
- [ ] Pilot pipeline reproduced on 10,000 items from one domain, matching the published numbers
- [ ] The full 80,000 subgraphs across 4 domains and 5 tasks, all gates passing, build diary written
- [ ] Sci-JEPA trained, repaired objective, 4 domain rotations, 3 seeds each
- [ ] The regression control and the trap-only predictor, both reported
- [ ] Dose sweeps with tipping points and bootstrap intervals, per task and per channel
- [ ] 300 hand-labelled real contributions and the comparison result
- [ ] A table linking every number in the abstract to where it came from
- [ ] Public release: generator, gates, self-test, per-seed logs, sealed holdout

**Every week.** A 30-minute check against this list, and one finished run's log read end to end. Both are cheap, and both are where our best corrections came from last time.
