# Data card

## Contents

| File | What it is |
|---|---|
| `dataset_v1.parquet` / `.jsonl.gz` | 11,700 documents: 5,850 human + 5,850 machine, topic-paired |
| `attacks_v1.parquet` | transformed **test** texts (paraphrase, back-translation, homoglyphs, typos, defence variants) |
| `splits/` | train / val / test document IDs and per-language counts |
| `build_logs/` | generation yield, removed items, attack quality (LaBSE similarity, language ID) |

### Languages and splits (documents)

| Language | train | val | test |
|---|---|---|---|
| en | 1386 | 194 | 370 |
| kk | 1378 | 214 | 358 |
| ky | 1384 | 200 | 366 |
| ru | 1380 | 184 | 386 |
| tr | 1358 | 176 | 416 |
| uz | 1334 | 194 | 422 |

### Generators (machine documents)

| Generator | Documents |
|---|---|
| dsv3 | 1170 |
| gem25lite | 1170 |
| gpt41mini | 1170 |
| haiku45 | 1170 |
| llama33_70b | 1170 |

Model identifiers, as requested through OpenRouter:

| ID | Model | Family |
|---|---|---|
| `gpt41mini` | openai/gpt-4.1-mini | openai |
| `haiku45` | anthropic/claude-haiku-4.5 | anthropic |
| `gem25lite` | google/gemini-2.5-flash-lite | google |
| `llama33_70b` | meta-llama/llama-3.3-70b-instruct | meta |
| `dsv3` | deepseek/deepseek-chat-v3-0324 | deepseek |

## Fields (`dataset_v1`)

| Field | Description |
|---|---|
| `doc_id` | `{pair_id}:h` (human) or `{pair_id}:m` (machine) |
| `pair_id` | topic pair; each pair has one human and one machine paragraph on the same Wikipedia article |
| `lang` | ISO 639-1 code |
| `title`, `qid` | Wikipedia article title and Wikidata ID |
| `split` | `train` / `val` / `test`, assigned by hashing the QID. No test QID occurs in training in any language. |
| `label` | 0 = human, 1 = machine |
| `generator`, `family` | generator ID and family (`human` for human texts) |
| `pair_generator`, `pair_family` | generator of the pair's machine text (also set on the human row) |
| `prompt_id` | which of the two COLING-2025-style prompt templates was used |
| `text` | normalised paragraph (40–200 words) |
| `n_words` | word count |
| `source_meta` | JSON. Human: Wikipedia `revid` and timestamp. Machine: requested model, served model and provider. |

`attacks_v1` adds `orig_doc_id` (the clean test document), `eval_set` (e.g. `T1_para`, `T2_bt`, `T3_h15`, `T3_h15both`, `T3_h15_def`, `T4_t05`, `test_def`), `attack`, `level`, `labse_sim` and `lid_ok`. Sets named `…both` contain transformed **human** texts as well.

## Construction

1. **Human texts.** The lead paragraph of each Wikipedia article, from the revision current on 2022-11-01 (before ChatGPT). Paragraphs of 40–200 words whose language GlotLID confirms; bot stub templates are removed.
2. **Machine texts.** One paragraph per topic, generated from the article title alone with two prompt templates (temperature 0.7, top-p 0.95). Outputs that are too short, in the wrong language, or contain meta-commentary are rejected.
3. **Pairing and normalisation.**
   - Pairs whose lengths differ by more than 10 words or 25 % are dropped.
   - Both classes go through the same normaliser: NFC, zero-width and stress-mark removal, removal of headings and Markdown, quote and dash unification, and symmetric removal of parenthetical spans.
   - Near-duplicates are removed with MinHash.
   - Every (language, generator) cell is balanced to 195 pairs.
4. **Transformations** (test machine texts; homoglyphs and typos also on human texts):
   - paraphrase by Qwen3-235B-A22B;
   - back-translation with NLLB-200-distilled-600M;
   - Cyrillic↔Latin homoglyphs at 5 % / 15 %;
   - character typos at 5 % / 15 % of words.

## Known limitations

- **Single domain:** Wikipedia lead paragraphs only.
- **Stylometric signal:** punctuation and sentence-length features alone separate the classes with AUROC 0.78–0.90 (`results/08_results/data_audit.csv`).
- **Contamination:** the human texts are public and are likely part of the pre-training data of the generators and scorers.
- **API generators** may change behind the same model name.

## Licences and attribution

- **Human texts** and their transformed versions are adapted from Wikipedia. Wikipedia text is licensed under CC BY-SA (revisions from 2022 under CC BY-SA 3.0, which permits adaptations under CC BY-SA 4.0). They are released under **CC BY-SA 4.0**.
  - Attribution: each human text links to its source revision via `https://{lang}.wikipedia.org/w/index.php?oldid={revid}` (from `source_meta`). Authors are listed in each article's history.
- **Machine texts** are outputs of the listed commercial and open-weight models. Their use may be subject to the respective providers' terms. They are released together with the dataset under CC BY-SA 4.0 for research on detecting machine-generated text.
- **Back-translations** (`eval_set == "T2_bt"`) were produced with NLLB-200, whose model licence is CC BY-NC 4.0. Treat this subset as **non-commercial research use only**.
- **COLING-2025 Kazakh data** are **not included**. Predictions on them are referenced by ID only.

This card describes the data as released; it is not legal advice.
