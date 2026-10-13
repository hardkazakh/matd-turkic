# Cross-lingual generalisation and robustness of multilingual AI-text detectors (Turkic languages)

Code, data and results for the paper **“Evaluating the Cross-Lingual Generalisation and Robustness of Multilingual AI Text Detectors: Evidence from Turkic Languages”** (under review).

We evaluate twelve detectors of machine-generated text on a new topic-paired corpus of **11,700 Wikipedia-style paragraphs**, written by humans or by five LLM families. The corpus covers four Turkic languages (Kazakh, Kyrgyz, Uzbek, Turkish) and two Indo-European reference languages (Russian, English). The language set crosses **language family × script**, so the two can be separated as drivers of cross-lingual transfer.

| Code | Language | Family | Script |
|---|---|---|---|
| kk | Kazakh | Turkic | Cyrillic |
| ky | Kyrgyz | Turkic | Cyrillic |
| uz | Uzbek | Turkic | Latin |
| tr | Turkish | Turkic | Latin |
| ru | Russian | Indo-European | Cyrillic |
| en | English | Indo-European | Latin |

## Main findings

The full analysis, with confidence intervals and multiplicity correction, is in the paper.

- **Fine-tuned multilingual encoders** (mDeBERTa-v3-base, XLM-R-base) reach test AUROC ≥ 0.987 in every language. Pooled AUROC is 0.995 for mDeBERTa and 0.992 for XLM-R.
- **Transfer pathway depends on the representation.** Transfer of monolingual detectors follows **language family** for the encoders and **script** for character n-gram models.
- **Unseen generator families** cost little (|ΔAUROC| ≤ 0.01), except Llama-3.3. Its drop of 0.04–0.05 is confounded with its atypical outputs.
- **Zero-shot detectors** (Binoculars, Fast-DetectGPT, log-likelihood, log-rank) are weaker, with pooled AUROC 0.76–0.88.
  - They fall below chance under 15 % homoglyph or typo perturbation.
  - Unicode NFKC normalisation with confusable mapping recovers most of the homoglyph loss.
- **A Kazakh-adapted scoring LM** does not help zero-shot detection. Much of its log-likelihood loss comes from per-token normalisation with its extended tokenizer.
- **Thresholds do not transfer across languages.** At a nominal 5 % FPR set on the training language, realised FPR in other languages reaches 89 %.

## Repository layout

```
notebooks/   00–14 Colab notebooks (the only code; outputs cleared)
config/      config.json – single source of truth for every setting
data/        dataset, splits, attack sets, build logs, data card (data/README.md)
results/     08_results/ metrics · 09_statistics/ RQ tests + posthoc/ · run_registry.csv · predictions/
figures/     all figures (PDF/PNG) and their source data
tables/      LaTeX tables
env/         package and GPU snapshots of the runs
```

## Reproducing the study

All code is in Colab notebooks that read and write a Google Drive folder, `MyDrive/multilingual-ai-text-detection/`. The folder layout is given in `config.json → paths`.

1. Copy `notebooks/` to `MyDrive/multilingual-ai-text-detection/notebooks/`.
2. Add Colab secrets:
   - `OPENROUTER_API_KEY` (needed for notebooks 03 and 05);
   - `HF_TOKEN` (optional).
3. Run the notebooks in order. Each one skips work that is already done, and every run is recorded in the run registry.

| # | Notebook | Runtime |
|---|---|---|
| 01 | setup, config, smoke tests | CPU |
| 02 | human texts from Wikipedia (revision as of 2022-11-01) | CPU |
| 03 | machine texts via OpenRouter | CPU (paid API) |
| 04 | pairing, normalisation, splits, shortcut audit, external set | CPU |
| 05 | attack sets (paraphrase, back-translation, homoglyphs, typos, defence) | GPU (T4+) + API |
| 06 | KerasHub fine-tuning (60 runs, batch 32, bf16) | A100 |
| 07 | character n-gram and stylometric baselines | CPU |
| 08 | zero-shot detectors (generic and Kazakh-adapted scorers) | A100 |
| 09–11 | evaluation, statistics, error analysis | CPU |
| 12 | figures and tables | CPU |
| 13 | exploratory robustness checks | CPU |
| 14 | this release | CPU |

**Re-running only the analysis (no GPU, no API):**

1. Put `config/config.json` in `00_admin/config/`, `data/dataset_v1.parquet` in `03_processed/` and `data/attacks_v1.parquet` in `05_attacks/`.
2. Put `data/splits/*` in `04_splits/`.
3. Put `results/predictions/*` in `07_predictions/` and `results/run_registry.csv` in `00_admin/registry/`.
4. Run notebooks 09–13.

The COLING-2025 Kazakh external test set is not redistributed. To reproduce those results, obtain it from the shared-task organisers and place it at `data/data/coling2025_kazakh.jsonl`.

**Frameworks:** Keras 3 and KerasHub (TensorFlow backend) for fine-tuning; Hugging Face Transformers only for the zero-shot scoring LMs; scikit-learn for the shallow models. Exact versions are in `env/`.

## Data

See [`data/README.md`](data/README.md) for fields, construction and licences. The data are released **for research on detecting machine-generated text**.

## Responsible use

Detector scores are probabilistic and can be wrong. Error rates depend on the language, the generator and on thresholds, which do not transfer between languages. Do not use any detector from this repository as the sole basis for decisions about individuals, for example in academic-misconduct cases.

## Licences

- **Code** (notebooks, config): Apache License 2.0 (`LICENSE`).
- **Data:** see `data/LICENSE.md`. Human texts are from Wikipedia and are released under CC BY-SA 4.0.

## Citation

See `CITATION.cff`. A full reference will be added when the paper is published.
