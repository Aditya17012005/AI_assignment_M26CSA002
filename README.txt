AI ASSIGNMENT 2026 - CLASSICAL REASONING AGENT
Roll number: M26CSA002

CONTENTS
  reasoning_agent.ipynb  Complete implementation, tests, public data, executed results
  report.pdf            Technical report generated from measured results
  submission.csv        250 public RuleTaker test demonstration predictions
  README.txt            This guide

QUICK START
1. Open reasoning_agent.ipynb in Jupyter, VS Code, or another Python notebook runner.
2. Select a Python 3.10+ kernel.
3. Run every cell in order.
4. Read the final accuracy/timing table and generated submission.csv.

The notebook's default mode requires only the Python standard library. It contains the complete
engine and compressed public evaluation snapshots, so it does not need a src/
directory, network access, Hugging Face login, an API key, or pip installs.
To install a notebook UI if needed, install Jupyter separately in your environment.

GENERATE PREDICTIONS FOR NEW INPUT
In the final cell set:
  INPUT_PATH = 'test.csv'
  OUTPUT_PATH = 'submission.csv'
  SEMANTICS = 'owa'
  USE_FORMAL_ANNOTATIONS = False
  PARSER_MODE = 'deterministic'  # alternatives: local, adaptive (requires local server)

Accepted formats: CSV, JSON list, JSONL.
Required columns: context, statement.
Optional column: id. If omitted, sequential IDs are used.
Aliases: premises/given_info/theory for context;
         question/conclusion/query for statement.
Output columns: id,predicted_label. Values are the strings True and False.
Do not require a label column for inference. Reference reasoning and answer labels
are ignored. If INPUT_PATH remains None, the public RuleTaker demo is regenerated.
The included CSV is not a prediction file for an unavailable private test set.

OPTIONAL FORMAL INPUT
Set USE_FORMAL_ANNOTATIONS=True only when the input supplies both premises-FOL
and conclusion-FOL. Formal-annotation performance is reported separately from
natural-language-only performance. The ordinary input path uses context/query.

METHODS AND SEMANTICS
- Natural language: deterministic compositional parser, indexed Horn closure,
  unification over all alternative bindings, proof provenance and bounded caching.
  Relative/quantified clauses, shared subjects, scoped negation and elliptical
  neither are supported. Parsed-context caches isolate state between queries.
- Default OWA: negative facts are explicit; absent evidence is not proof of negation.
- Optional CWA: negative bodies/queries use closed-world negation. Stratified rules
  use their completed strata; cycles use a finite well-founded model, and undefined
  queries stay unsupported. OWA remains the default for general entailment.
- General formulas: scoped syntax tree, NNF, Skolemization, CNF, bounded ground
  instances, equality congruence and our own DPLL search. UNSAT proves entailment;
  a bounded SAT result means not proved, not a complete FOL countermodel.
  Lifted unit resolution tries proofs before grounding. Definitional predicates
  control XOR/iff expansion. Free query variables are universally closed first.
  Categorical English can use classical refutation beyond Horn closure.
- Counterfactuals: Boolean SCM, exact exogenous enumeration, factual conditioning,
  intervention and downstream recomputation. The posterior probability is kept;
  the binary causal convention is probability > 0.5. Unspecified independent root
  priors default to Bernoulli(0.5). Cycles/contradictory observations are diagnosed.
- The optional local model translates English into formulas only. Our classical
  code makes the final truth decision. No learned answer classifier, external
  theorem prover, answer table or public-reference rationale decides the label.

PUBLIC RESULTS FROM THE INCLUDED EXECUTED NOTEBOOK
  Linked RuleTaker test                 97.60% (244/250)
  Linked RuleTaker OOD                  99.47% (373/375)
  Alternative RuleTaker depth-1 CWA    100.00% (200/200)
  CounterBench alpha                    95.00% (950/1000)
  CounterBench backdoor                 90.00% (180/200)
  FOLIO NL, binary entailment           69.95% (142/203)
  FOLIO formal, binary entailment       83.74% (170/203)
  FOLIO formal, True/False-only         76.12% (102/134)
  FOLIO NL, True/False-only             56.72% (76/134)

counterbench_alpha_heldout: 95.00% (380/400).
counterbench_backdoor_heldout: 87.50% (70/80).
Five name substitutions of each template stay in the same partition.
Public data had been examined during the preceding audit; these are public
benchmarks, not blind private-test results. All original labels are retained.

FOLIO has 69 Uncertain examples. For binary entailment these map to False; the
134-row subset is also reported for comparison with the original solution.
Unsupported syntax and proof limits map to False but retain diagnostic statuses.
Timing values are measured and printed in the notebook and report. Every batch
starts with a fresh agent, with context caching enabled within that batch.

VALIDATION
122 regression tests pass, including an independent exhaustive truth-table check
on 150 SAT instances and 40 complete Boolean entailment problems. The notebook
was executed by a fresh Python
interpreter in an isolated temporary directory. Every code cell has saved output.

LIMITATIONS
Natural-language-only FOLIO coverage and recall remain limited. Annotated FOL is
not guaranteed to be present in unseen input. Grounding/equality/time budgets can
miss valid proofs. Some public annotations are malformed and some labels disagree
with short derivations. Whole unsupported English/formal premises can be omitted with a
diagnostic; this never drops individual antecedents from a retained rule. The
report distinguishes incomplete parsing, resource limits and an unproved query.
No private test set was accessed, used for tuning, or manually labeled.
Public evaluation examples were inspected during the improvement pass. Reported
scores are after public-benchmark error analysis, not untouched validation scores.
Language translation is heuristic; broader recall brings extra false positives.

FULL PROJECT COMMANDS (AVAILABLE IN THE SEPARATE SOURCE ARCHIVE)
  python tests/test_engine.py
  python evaluate.py --phase dev
  python evaluate.py --phase final
  python predict.py test.csv --output submission.csv
  python download_data.py --manifest-only
  python build_submission.py

The source project includes engine.py, the test suite, evaluator, CLI, cached
public snapshots, checksums, per-example results and the notebook/report builder.
Only rebuilding report.pdf needs an extra package: reportlab.
Refreshing data with download_data.py needs internet access and may change the
upstream snapshot; use the included cached files to reproduce submitted results.

DATA SOURCES
  https://huggingface.co/datasets/jise/ruletaker
  https://huggingface.co/datasets/tasksource/ruletaker
  https://huggingface.co/datasets/CounterBench/CounterBench
  https://huggingface.co/datasets/tasksource/folio
The assignment's original FOLIO link is https://huggingface.co/datasets/yale-nlp/FOLIO
(gated). The public mirror used here is named explicitly. CounterBench question
JSON files are downloaded separately from its metadata files.

DEVELOPMENT NOTES
This version was developed from the two supplied prototypes' design ideas, with
new correctness checks and repaired/replaced parsing and inference components.
All final decisions are produced by the executable classical reasoning code.
The original archives were preserved separately.

OPTIONAL LOCAL SEMANTIC PARSER
The notebook includes Qwen3-4B-Instruct-2507 translation, structural validation,
up to two validator-feedback repairs, a classical proof pipeline and an optional
general resolution implementation. The local parser accepts only source premises
and queries; it never outputs an answer label. Translation may still be wrong.

The notebook embeds setup_local_parser and start_local_parser helpers. To use
the local model on this Windows machine with an NVIDIA GPU, run:
  setup_local_parser(cuda=True)
  parser_process, parser_endpoint = start_local_parser(cuda=True)
Then set PARSER_MODE='local' in the final CSV cell. For CPU use cuda=False.
Stop the server afterward:
  parser_process.terminate()
  parser_process.wait()
Downloads are optional, not automatic: about 2.5 GB of model weights plus a pinned
llama.cpp runtime, saved outside the submission in the user's cache. No API key
is needed. Other platforms can run a compatible llama.cpp loopback server using
the same GGUF model. CUDA/CPU generation can differ; measured translations are
retained for auditable proof replay.

The notebook includes matched development (21 queries), reserved (48 queries)
and public-validation (203 queries) study evidence. Parser settings/code were
frozen before reserved evaluation. The reserved stories were excluded from the
previously inspected training prefix; public validation had already been examined
in earlier passes. Read the study table and report for the measured results.

By default the notebook REPLAYS recorded translations and recomputes every proof.
It prints original translation time separately from current proof-replay time.
Set LIVE_LOCAL_STUDY=True with the server running for fresh model generation.
The replay data is never used by the new-input prediction path.

Source-archive commands:
  python experiments/setup_local_parser.py --cuda
  python experiments/serve_local_parser.py --cuda
  python predict.py test.csv --parser local --output predictions.csv
The last command runs in a second terminal while the server is active, and writes
a diagnostics JSON with statuses, translated formulas and proof traces.
Omit --cuda for CPU. General resolution and portfolio reasoning are optional CLI
choices; the existing ground solver remains selected because broader resolution
did not improve the formal-input benchmark. Causal and CWA inputs retain their
established classical paths. Full evidence is documented in APPROACH_STUDY.md.

LOCAL PARSER STUDY RESULTS
Matched binary-entailment comparisons (local mode versus deterministic mode):
  dev          deterministic 14/21 (66.67%), local 19/21 (90.48%)
  reserved     deterministic 32/48 (66.67%), local 33/48 (68.75%)
  validation   deterministic 142/203 (69.95%), local 162/203 (79.80%)
The development score is a tuning result. Reserved improvement is modest.
Public validation was inspected in earlier passes; this is not private-test accuracy.
Detailed precision, recall, runtime and failed-translation counts are in the report.

GENERALIZATION AUDIT (FROZEN PUBLIC SPLIT)
  Fresh final: 2938 RuleTaker/FOLIO questions; 1434 True, 1504 False.
  Overall accuracy: baseline 79.37% (2332/2938), improved 85.94% (2525/2938).
  RuleTaker: 81.23% -> 88.61%; FOLIO: 64.29% -> 64.29%.
  Paired bootstrap over 320 knowledge-base/story groups: gain 6.57 points; 95% interval [4.34, 8.97].
  Human-written paraphrases: 51.32%; controlled language: 99.80%.
  CounterBench: 97.32% on validation only; all source questions were previously examined.
  Public data only. See report.pdf and GENERALIZATION_AUDIT.md.
