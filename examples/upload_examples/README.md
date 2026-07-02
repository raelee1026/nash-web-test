# Upload Result Formats

These files are completed NASH evaluation result examples for the Batch Upload page.

Batch Upload only accepts these three completed evaluation result formats. Other JSON shapes and CSV files are treated as format errors.

Use one `rows` item per evaluation unit:

- `triplet`: one row per `{anchor, s1, s2}` group.
- `crosspair`: one row per four-sentence group.
- `listwise`: one row per anchor plus candidate list.

Each row should include model metadata and `pair_results`. Each `pair_results` item is one scored sentence pair.

## Triplet

```json
{
  "kind": "nash_evaluation_results",
  "task": "triplet",
  "rows": [
    {
      "id": "triplet_001",
      "title": "Temporal triplet 001",
      "task": "triplet",
      "anchor": "...",
      "s1": "...",
      "s2": "...",
      "model": "sentence-transformers/all-MiniLM-L6-v2",
      "baseline_label": "Sentence-BERT / all-MiniLM-L6-v2",
      "backend": "sentence-transformer",
      "pair_results": [
        { "label": "S1", "sentence1": "...", "sentence2": "...", "baseline_score": 0.958, "nash_score": 0.941, "text_score": 0.963, "numeric_score": 0.912 },
        { "label": "S2", "sentence1": "...", "sentence2": "...", "baseline_score": 0.951, "nash_score": 0.602, "text_score": 0.947, "numeric_score": 0.183 }
      ]
    }
  ]
}
```

## Cross-pair

```json
{
  "kind": "nash_evaluation_results",
  "task": "crosspair",
  "rows": [
    {
      "id": "crosspair_001",
      "title": "Temporal crosspair 001",
      "task": "crosspair",
      "model": "sentence-transformers/all-MiniLM-L6-v2",
      "baseline_label": "Sentence-BERT / all-MiniLM-L6-v2",
      "backend": "sentence-transformer",
      "pair_results": [
        { "label": "Pair A", "sentence1": "...", "sentence2": "...", "baseline_score": 0.969, "nash_score": 0.948, "text_score": 0.966, "numeric_score": 0.921 },
        { "label": "Pair B", "sentence1": "...", "sentence2": "...", "baseline_score": 0.944, "nash_score": 0.586, "text_score": 0.941, "numeric_score": 0.158 }
      ]
    }
  ]
}
```

## Listwise

```json
{
  "kind": "nash_evaluation_results",
  "task": "listwise",
  "rows": [
    {
      "id": "listwise_001",
      "title": "Temporal listwise 001",
      "task": "listwise",
      "model": "sentence-transformers/all-MiniLM-L6-v2",
      "baseline_label": "Sentence-BERT / all-MiniLM-L6-v2",
      "backend": "sentence-transformer",
      "pair_results": [
        { "label": "Candidate 1", "sentence1": "...", "sentence2": "...", "baseline_score": 0.971, "nash_score": 0.943, "text_score": 0.968, "numeric_score": 0.906 },
        { "label": "Candidate 2", "sentence1": "...", "sentence2": "...", "baseline_score": 0.964, "nash_score": 0.733, "text_score": 0.961, "numeric_score": 0.452 }
      ]
    }
  ]
}
```

Optional pair fields for matrix display:

- `masked_sentence1`
- `masked_sentence2`
- `numbers_sentence1`
- `numbers_sentence2`
- `alignment.matrix`
- `alignment.selected_alignment_edges`
