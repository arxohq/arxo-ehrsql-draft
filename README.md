# Arxo Materia EHRSQL 0.1: a 4B model reaches 67.5% on EHRSQL in MedHELM

EHRSQL asks a model to answer a clinician's question about hospital records by writing one SQL query against the eICU database: lab results, prescriptions, diagnoses, costs, lengths of stay. MedHELM uses it as its benchmark for querying patient records.

Arxo Materia EHRSQL 0.1, `Arxo-Materia-4B-EHRSQL-0.1`, is a 4B model based on Qwen3.5-4B. It reads the database schema and the question and writes the query on its own. It is the second Arxo Materia release, after [Arxo Materia 0.1 on MedCalc-Bench](https://github.com/arxohq/arxo-medcalc-bench).

On the 1,959 evaluation questions of EHRSQL as MedHELM runs it, scored by MedHELM's own evaluator:

| Model | Execution accuracy |
|---|---|
| Arxo Materia EHRSQL 0.1 (4B), based on Qwen3.5-4B | 1322 / 1959 (67.5%) |
| Arxo Materia EHRSQL 0.1 (0.8B), based on Qwen3.5-0.8B | 1309 / 1959 (66.8%) |
| Claude Opus 5.5, best result in the EHRSQL column of the MedHELM leaderboard | 63.2% |
| Qwen3.5-4B, no training | 324 / 1959 (16.5%) |
| Qwen3.5-0.8B, no training | 65 / 1959 (3.3%) |

Each Arxo Materia row is one training run. The untrained rows use the same prompt and the same token limit as the trained ones. They were scored with the evaluator's comparison outside `helm-run`, with a limit of 60 seconds per query: `helm-run` sets no time limit, and some queries of the untrained models do not finish.

## By split

MedHELM evaluates on the validation and test splits of EHRSQL together.

| Model | Validation (755 questions) | Test (1,204 questions) |
|---|---|---|
| Arxo Materia EHRSQL 0.1 (4B) | 526 (69.7%) | 796 (66.1%) |
| Arxo Materia EHRSQL 0.1 (0.8B) | 521 (69.0%) | 788 (65.4%) |

Share of answers that execute without an error: 99.8% for the 4B model, 99.6% for the 0.8B model.

## Arxo Materia EHRSQL 0.1 (0.8B)

Arxo Materia EHRSQL 0.1 (0.8B), `Arxo-Materia-0.8B-EHRSQL-0.1`, is the same model built on Qwen3.5-0.8B. With a fifth of the parameters it scores 13 questions fewer than the 4B model.

## Results on the MedHELM leaderboard

EHRSQL column of the MedHELM leaderboard, release v6.0.0 of 9 November 2026. These rows use the same 1,959 questions as this repository.

| System | Trained on the task | Execution accuracy | Source |
|---|---|---|---|
| **Arxo Materia EHRSQL 0.1 (4B)** | yes | **67.5%** | this repository |
| **Arxo Materia EHRSQL 0.1 (0.8B)** | yes | **66.8%** | this repository |
| Claude Opus 5.5 | no | 63.2% | [MedHELM leaderboard](https://leaderboard.medhelm.org/) |
| GPT-6.1 Sol | no | 42.8% | MedHELM leaderboard |
| Qwen3.8 Max | no | 42.7% | MedHELM leaderboard |
| Gemini 3.8 Flash | no | 41.0% | MedHELM leaderboard |

The Arxo Materia models were trained on the training split of EHRSQL. The leaderboard models are general models that answer from the prompt alone.

Other rows of the same column were measured on a different number of questions and are listed for reference.

| System | Questions | Execution accuracy |
|---|---|---|
| Gemini 3.1 Pro (Preview) | 2,909 | 44.3% |
| GPT-5.4 mini | 2,909 | 34.4% |
| GPT-5.4 | 2,909 | 34.4% |
| Claude 4.6 Opus | 2,909 | 26.5% |
| Gemini 3.5 Flash | 2,909 | 21.7% |
| Gemini 2.5 Pro | 1,000 | 21.3% |
| Muse Spark | 2,909 | 19.2% |
| Gemini 2.0 Flash | 1,000 | 16.5% |
| Claude 3.7 Sonnet | 1,000 | 8.1% |
| DeepSeek R1 | 1,000 | 7.6% |
| Llama 3.3 Instruct (70B) | 1,000 | 7.4% |

## How the score was produced

- The prompt is MedHELM's own: its instructions, the `CREATE TABLE` statements of the eICU schema and the question, with no examples. Generation settings are MedHELM's: temperature 0, at most 1,024 new tokens. Thinking is disabled.
- The model writes one comment line that restates the question and then the query inside `<sql>` tags. MedHELM takes the query from the tags.
- The score is `ehr_sql_execution_accuracy` from `helm-run` with the unmodified `ehr_sql` run entry of [PacificAI/medhelm](https://github.com/PacificAI/medhelm) at commit `8843816c33`. The model's answers were recorded once and served to `helm-run` through an OpenAI-compatible endpoint. HELM sent 1,956 distinct prompts, and each is identical to the prompt the model had answered.
- The evaluator ran on SQLite 3.46.1.

## Training and evaluation questions

EHRSQL builds its training, validation and test questions from the same set of question templates. Of the 1,959 evaluation questions, 641 have a training question with the same template and the same values, and 8 are word for word identical to a training question. These questions were kept in the training data.

| Evaluation questions | Arxo Materia EHRSQL 0.1 (4B) | Arxo Materia EHRSQL 0.1 (0.8B) |
|---|---|---|
| Same template and values as a training question (641) | 448 (69.9%) | 434 (67.7%) |
| All other questions (1,318) | 874 (66.3%) | 875 (66.4%) |

## What we did not do

- No validation or test question was used to train the model. The two splits were opened only to count the overlap above and to measure.
- No prompt was tuned: the prompt and the generation settings are MedHELM's.
- No score of an Arxo Materia model comes from our own grader: each is MedHELM's `ehr_sql_execution_accuracy`.

## Scope of the numbers

The scores refer to EHRSQL as MedHELM runs it: the validation and test splits of the eICU part, 1,959 questions that have a reference query.

For 302 of these questions the reference query returns no rows, and the evaluator credits them only to an empty answer. The Arxo Materia models write a query for every question. On the other 1,657 questions the 4B model scores 79.8% and the 0.8B model 79.0%.

The evaluator compares result rows for exact equality, so a query that sums decimal values can score differently on another SQLite version. On SQLite 3.53.1 the 4B model scores 1,322 again and the 0.8B model 1,283.

## Reproducing the result

The adapters are in the Release `v0.1`, the model outputs and per-question scores in `answers/` and `eval/`. To measure a model yourself, merge the adapter into the base model, serve it with thinking disabled and run `helm-run --run-entries "ehr_sql:model=<your deployment>"` from PacificAI/medhelm.

Arxo is in early access: to run it on your own data, or to verify the result independently, write to hi@arxo.io.

## What is in this repository

| Path | Contents |
|---|---|
| `answers/4b.jsonl`, `answers/0.8b.jsonl` | Model outputs for the 1,959 evaluation questions, keyed by the question id of EHRSQL |
| `answers/untrained-4b.jsonl`, `answers/untrained-0.8b.jsonl` | Outputs of the base models without training |
| `eval/` | Per-question scores of the two Arxo Materia models from MedHELM's evaluator |
| `helm/` | `stats.json` written by `helm-run` for the two Arxo Materia models |
| `adapters/` and the Release `v0.1` | Adapter configs and LoRA weights: about 300 MB for the 4B model, about 100 MB for the 0.8B model |
| `SHA256SUMS` | Checksums of every file, weights included |

Load an adapter with `AutoModelForImageTextToText`; it does not attach to `AutoModelForCausalLM`. Save a weights file from the Release as `adapter_model.safetensors` next to its `adapter_config.json`.

Licences: the adapters are released under Apache-2.0, the licence of the base models Qwen/Qwen3.5-4B and Qwen/Qwen3.5-0.8B. Answers and evaluation outputs refer to EHRSQL questions and are released under CC-BY-4.0, the licence of the benchmark.
