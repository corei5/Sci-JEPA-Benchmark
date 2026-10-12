**Contents** [1 What we release](#what) [2 Release gate](#gate) [3 Technical report outline](#report) [4 Benchmark suite](#bench) [5 Models to compare](#baselines) [6 Fair comparison rules](#fair) [7 Use cases](#usecases) [8 Release package](#package) [9 Licensing and responsible use](#license) [10 Work plan](#plan) [References](#refs)

SCI-JEPA · public release

# Releasing the model with a technical report

Everything needed to turn the Paper A model into a public model: what to release, the go/no-go test before release, the report structure, the benchmark suite, the models to compare against, use cases, and the release package.

**Working name** SciKU-JEPA**Prepared** 12 Oct 2026**Companion** Paper A technical guideline

## 1What we release

SciKU-JEPA is a predictive model of scientific knowledge. A paper is split into knowledge units (KUs): context, question, method, procedure, result, interpretation, limitation, claims, evidence and implications. Given some KUs of a paper and its neighbouring papers, the model predicts the representation of a missing one. That gives users three things no general embedding model offers directly:

- **Facet embeddings:** separate vectors for a paper's problem, method, result and claims, in one space.
- **Conditional prediction:** "given this problem and method, what result is expected?" as a vector that can be ranked against real results.
- **Relational signals:** support vs contradiction between claims and evidence, and how a prediction reacts when a condition changes.

The adoption problem to solve first

The model's input is a **structured summary** (the 10 KU types), produced for our corpus by an LLM. A public user has a PDF or an abstract. The release must therefore ship the extraction step: the summary schema, the prompt, and a recommended open LLM to run it. The report must measure how much accuracy drops when KUs come from that open LLM instead of the one that built the training data ("extractor shift").

## 2Release gate: is there something to release?

In the 500-paper pilot, plain cosine similarity of the frozen gte-modernbert embeddings (the model's own input features, no training) was as strong as the trained model on a small test pool. A public model is only worth releasing if it adds something over its own encoder. Decide go / no-go on the D0 tier (60,000 papers, 5 seeds) before any release work:

| Gate | Test | Pass if |
| --- | --- | --- |
| G1 · beats its encoder | Masked-KU balanced retrieval, every KU type, time-split test | higher than frozen gte-modernbert (joined visible text → cosine) on most types, paired test p \< 0.05 over 5 seeds |
| G2 · beats the same data with a standard objective | Same architecture and data trained contrastively (InfoNCE) | JEPA ≥ contrastive on retrieval and better on at least one reasoning task |
| G3 · reasoning is real | Reasoning-task matrix | forward, inverse or abduction ≥ 1.5× chance in the leak-free bins; intervention shift clearly above 0 |
| G4 · transfers | At least one public benchmark (Section 4, tier 2) | competitive with SPECTER2 on faceted retrieval |
| G5 · survives extractor shift | KUs re-extracted with the recommended open LLM | at most a small drop vs the original summaries |

If G1 or G2 fail, the honest release is the **benchmark and the analysis** (a dataset and evaluation paper) rather than a model.

## 3Technical report outline

Modelled on recent open-model reports (Llama, Qwen, Gemma): a self-contained description that lets others reproduce, evaluate and use the model.

| Section | Content | Source in our work |
| --- | --- | --- |
| Abstract and contributions | model, benchmark, key numbers, links | written last |
| 1 · Introduction | why predict knowledge units; what JEPA adds over embedding models; the identity-shortcut problem | Paper A introduction |
| 2 · Data | corpus, KU schema, cleaning, statistics per type and field, splits, licensing, extraction prompt | `ku_schema.py`, `ku_graph.py`, data card |
| 3 · Model | tokens, relation-biased graph transformer, EMA target, predictor, inverse head, sizes | `model.py`, size table |
| 4 · Training | objective, leak-safe masking, schedule, hyperparameters, compute, stability and collapse monitoring, scaling (D0 → D1, S → M) | `train.py`, dashboard logs |
| 5 · Evaluation | 5.1 SciKU benchmark · 5.2 reasoning tasks · 5.3 public benchmarks · 5.4 efficiency | Section 4 below |
| 6 · Analysis | ablations (E10–E18), leak analysis, neuron interpretation, failure cases, extractor shift | Paper A experiments |
| 7 · Use cases | worked examples with numbers | Section 7 below |
| 8 · Limitations and responsible use | what the model cannot do, data biases, contamination, misuse | Section 9 below |
| Appendix | all hyperparameters, prompts, per-task and per-seed tables, model card, data card | result JSONs |

## 4Benchmark suite

### Tier 1 · SciKU benchmark (our own, released with the model)

Frozen from the evaluation code so anyone can score a model on it:

- **Masked-KU retrieval** per KU type: the true KU among 100 same-type candidates from related papers, MRR and Hits@k, with chance.
- **Reasoning-task matrix:** forward prediction, experiment design, abduction, consistency, intervention (as in `tasks.py`).
- **Two test sets:** the time split (test year 2020), and a **fresh set of papers from after the text encoder's training cutoff (2025–2026)**, so the report can make claims free of encoder contamination.
- **Released files:** paper IDs per split, candidate pools, query templates, leak-similarity of every query, and a scorer script. Released KU texts depend on licensing (Section 9).

### Tier 2 · Public benchmarks

| Benchmark | What it tests | Why it fits | How we feed it |
| --- | --- | --- | --- |
| **CSFCube** | faceted query-by-example: background, method, result | its facets map one-to-one onto our context, method and result KUs | abstract sentences already carry facet labels |
| **DORIS-MAE** | complex multi-aspect queries over CS abstracts | tests aspect-level matching | KU extraction on abstracts |
| **LitSearch** | literature search from research questions | question → paper retrieval | query as a question KU |
| **MAPLE (2026)** | multi-aspect, full-paper retrieval | closest to our full-paper KU setting | KU extraction on full papers |
| **SciRepEval** (incl. SciDocs) | 24 tasks: classification, regression, proximity, search | the standard for scientific document embeddings | paper embedding = pooled KU states |
| **SciFact** | claim verification (supports / refutes) | external check of the consistency task | claim and evidence as KUs |
| **MTEB scientific tasks** | SCIDOCS, SciFact retrieval, arXiv clustering | puts the model on the public leaderboard | Sentence-Transformers wrapper |
| **ogbn-arxiv** | node classification on a citation graph | standard for graph and text-graph models | paper embeddings as node features |

Input mismatch

Public benchmarks provide titles and abstracts, not our 10-type summaries. Report every tier-2 result in two input modes: **KU mode** (run the released extractor on the benchmark text) and **abstract mode** (abstract sentences mapped to the closest KU types). The gap between the two is itself a finding.

### Tier 3 · Efficiency

Parameters, embedding size, encoding throughput (papers per second on one GPU and on a laptop), extraction cost per paper, and memory. General embedders now range from 300 M to 8 B parameters; our trainable model is small (5 M local, larger at S), but it depends on a 150 M text encoder and an extraction LLM. Report the full pipeline cost.

## 5Models to compare

| Group | Model | Why it is needed |
| --- | --- | --- |
| **Floors** | Random and BM25 | chance and lexical floor for every retrieval task |
| **Frozen gte-modernbert** (our encoder), cosine on joined visible text and on mean of KU embeddings | the most important baseline: what training adds over the input features |  |
| Majority class / type centroid | floor for consistency and per-type tasks |  |
| **Scientific encoders** | SPECTER2 (proximity and ad-hoc adapters) | the reference scientific paper embedder; SciRepEval baseline |
| SciNCL | citation-neighbourhood contrastive learning |  |
| SPECTER (v1), SciBERT (mean pooled) | older reference points for continuity with prior work |  |
| Aspire (multi-vector, facet-aware) | closest prior model for fine-grained, aspect-level similarity |  |
| **General embedders** | Qwen3-Embedding (0.6 B, 4 B, 8 B) | strongest open-weight family on MTEB; shows the scale trade-off |
| BGE-M3 | widely used, permissive, long context |  |
| multilingual-E5-large, nomic-embed-text, EmbeddingGemma | common mid-size open embedders |  |
| NV-Embed-v2 | strong English model (non-commercial licence; research comparison only) |  |
| One closed API (OpenAI text-embedding-3-large or Gemini embedding) | reference ceiling; small subset only, cost-limited |  |
| **Graph models** | GraphSAGE and GAT on our KU graph, same features | does a standard GNN on the same graph match JEPA? |
| GraphMAE | self-supervised masked reconstruction on graphs |  |
| Graph-JEPA | the existing JEPA for graphs (graph-level) |  |
| **Same-data controls** | Our architecture trained with InfoNCE | isolates the JEPA objective from data and architecture |
| Our architecture trained with masked-feature reconstruction | predictive in latent space vs reconstructive |  |
| gte-modernbert fine-tuned contrastively on KU pairs | the simplest way to use the same supervision |  |
| **LLM references** (reasoning tasks) | An open 7–8 B instruction model as zero-shot reranker over the same candidates | can a general LLM answer the forward / inverse / abduction queries? |
| One frontier API model as reranker on a small subset | upper reference; report cost per query |  |
| LLM-JEPA | the closest JEPA-for-language method |  |

Minimum set if compute is tight: frozen gte-modernbert, SPECTER2, Qwen3-Embedding-0.6B and 8B, BGE-M3, the InfoNCE control, GraphSAGE, and one open LLM reranker.

## 6Fair comparison rules

- Every model sees the **same input text**: the visible KUs joined in a fixed template, or the abstract in abstract mode. No model sees hidden KUs.
- **Same candidate pools**, splits and queries for every model; pools are released.
- Embedders score by cosine between the query text and each candidate's text; our model scores by its prediction against candidate targets. Report both scoring rules for our model.
- Use each baseline's **recommended prompt or instruction** (Qwen3, E5 and nomic need task prefixes) and its native maximum length.
- Give trained baselines the **same tuning budget** (same validation split, same number of trials).
- Report mean ± std over 5 seeds for anything we train, with paired tests; one run for frozen baselines.
- Report results on the **leak-free bins** and on the post-2024 test set, not only overall.
- Report parameters, embedding size and cost next to every score.

## 7Use cases

Each use case ships as a short notebook plus a demo, and gets one number in the report so it is not only an anecdote.

| Use case | User input → output | How it is measured |
| --- | --- | --- |
| **Faceted literature search** | a paper or a query → papers with a similar *method* (or problem, or result) but not necessarily the same topic | CSFCube facets; per-facet MRR vs SPECTER2 |
| **Result anticipation** | problem + method → ranked list of results reported by similar work | forward task on the post-2024 test set |
| **Experiment design support** | problem + desired result → methods and setups that produced similar results | inverse task; expert check on a small sample |
| **Claim–evidence checking** | a claim with its evidence → supports / contradicts score | consistency task and SciFact |
| **Novelty signal** | a paper → how surprising its result is given its problem and method (prediction error) | correlation with expert novelty ratings or later citations; report as exploratory |
| **Completing knowledge records** | a summary with a missing section (e.g. no limitations) → closest existing texts for it | masked-KU retrieval for that type; useful for ORKG curation |
| **Reviewer and collaborator matching** | a submission → people whose methods match its method KU | SciRepEval proximity tasks |
| **KU-level retrieval for assistants (RAG)** | a question → the specific results, methods or claims that answer it | LitSearch, DORIS-MAE; answer quality with a fixed LLM |

Wording in demos

The model predicts a *representation* and ranks existing texts. It does not generate results and cannot replace an experiment. Demos must show retrieved real texts with their sources, never "the model predicts your result will be X".

## 8Release package

| Artifact | Content | Where |
| --- | --- | --- |
| Weights | context encoder, target encoder, predictor, inverse head (safetensors), config, sizes S and M | Hugging Face Hub |
| Inference package | `pip install`; embed a paper, predict a hidden KU, score support / contradiction, intervene | GitHub + PyPI |
| KU extractor | schema, prompt, recommended open LLM, validation that output parses into the 10 types | GitHub |
| SciKU benchmark | splits, candidate pools, templates, scorer, leaderboard table | Hugging Face Datasets + Zenodo DOI |
| MTEB wrapper | Sentence-Transformers-compatible class so `mteb` can run the model | GitHub; leaderboard submission |
| Demo | faceted search and result anticipation on a public corpus slice | Hugging Face Space |
| Model card and data card | intended use, training data, metrics, limitations, licences, contamination statement | with the weights and dataset |
| Technical report | Section 3 outline | arXiv |

Target interface for the inference package:

```
from sciku import SciKU

model = SciKU.from_pretrained("<org>/sciku-jepa-s")
kus = model.extract(pdf_or_abstract)                 # the 10 KU types, via the released extractor
emb = model.embed(kus)                               # {"method": vec, "result": vec, ..., "paper": vec}
pred = model.predict(kus, hide="result")             # predicted result representation
hits = model.search(pred, index="arxiv-2025", k=10)  # real results from similar work, with sources
score = model.consistency(claim, evidence)           # P(supports), P(contradicts)
delta = model.intervene(kus, change={"procedure": other_procedure}, hide="result")
```

## 9Licensing and responsible use

Check before release

- arXiv papers carry different licences; many allow reading but not redistribution. Release summaries only for openly licensed papers; otherwise release IDs plus the extraction code.
- The terms of the LLM that generated the training summaries may restrict redistributing its outputs or training on them.
- Licences of the text encoder and every redistributed component.

State in the model card

- Field coverage (the corpus is arXiv-heavy: physics, CS, maths) and English only.
- The encoder's training data runs to 2024: no foresight claims on earlier test years.
- Outputs rank existing literature; they are not evidence.
- Known failure modes: identity shortcut, paraphrase copying, extractor errors.

Choose

- Model licence: a permissive licence (Apache-2.0) if the data terms allow it.
- Benchmark licence: CC BY 4.0 for anything we create.
- A contact and versioning policy for bug reports and updated releases.

## 10Work plan

In order; each phase ends with a checkable result.

| Phase | Work | Done when |
| --- | --- | --- |
| 0 · Gate | D0 runs (5 seeds) of the reference model, frozen-encoder baseline, InfoNCE control | gates G1–G3 decided |
| 1 · Freeze the model | pick size (S or M) and data tier (D0 or D1); final training; collapse and leak checks | final checkpoints and training logs |
| 2 · Extractor | package the summary prompt; choose the open LLM; measure extractor shift (G5) | extractor release candidate |
| 3 · Benchmark harness | SciKU benchmark files and scorer; post-2024 test set; wrappers for SciRepEval, CSFCube, DORIS-MAE, LitSearch, MAPLE, SciFact, MTEB, ogbn-arxiv | one command scores any model |
| 4 · Baselines | run Section 5 with the Section 6 rules | full results table |
| 5 · Use cases | notebooks and the Space demo; one number per use case | demos reviewed by a domain expert |
| 6 · Report | write Sections 1–8 and the appendix; internal review | arXiv-ready PDF |
| 7 · Release | Hub, PyPI, Datasets, Zenodo DOI, MTEB submission, Space, arXiv | public links work from a clean machine |

**Compute:** measure one D0 run first, then multiply. Runs needed: 205 for the Paper A experiments without E1–E3 (41 arms × 5 seeds; 285 with them), 15 for the three same-data controls, plus the GNN baselines. Frozen embedders need only encoding passes.

## References

- [SciRepEval: a multi-format benchmark for scientific document representations](https://arxiv.org/abs/2211.13308v4)
- [SPECTER2 (model card mirror)](https://huggingface.co/NetworkIsLife/specter2_base)
- [DORIS-MAE: scientific document retrieval with multi-level aspect-based queries](https://arxiv.org/pdf/2310.04678v2)
- [MAPLE: a multi-aspect full-paper scientific retrieval benchmark](https://arxiv.org/pdf/2608.15624)
- [UniFAR: unified facet-aware retrieval for scientific documents](https://arxiv.org/pdf/2602.23766)
- [Facet-aware reranking for paper recommendation](https://arxiv.org/pdf/2604.16329)
- [LLM-JEPA: large language models meet joint embedding predictive architectures](https://arxiv.org/html/2509.14252v2)
- [Graph-JEPA: graph-level representation learning with JEPA](https://arxiv.org/html/2309.16014v2)
- [Embedding model rankings, 2026 (secondary source; verify on the live MTEB leaderboard)](https://blog.premai.io/best-embedding-models-for-rag-2026-ranked-by-mteb-score-cost-and-self-hosting)