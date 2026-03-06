# Automated Essay Scoring (AES)

## Current Project Status

### Already done (before this update)
- A prototype notebook exists: `AES.ipynb`.
- The notebook demonstrates:
  - a small dummy dataframe for testing essays and scores,
  - loading `mistralai/Mistral-7B-Instruct-v0.2` with 4-bit quantization,
  - prompting the model for rubric-style scores (`Content`, `Language`, `Adherence`, `Narrativity`, `Holistic`) on a 0-10 scale,
  - regex extraction of numeric scores from generated text.
- The current notebook is a proof of concept; it is not yet wired to an ASAP++ preprocessing pipeline.

### Added in this update
- `scripts/prepare_asap_prompts_3_6.py`
  - Filters dataset to prompts `3, 4, 5, 6`.
  - Trims to a target row count (default `7,133`).
  - Removes HTML tags and non-ASCII artifacts from text columns.
  - Normalizes score columns to unified `0-10` scale using prompt-specific score ranges.
- `config/prompt_rubrics_3_6.json`
  - Prompt-wise rubric criteria mapping for prompts `3`, `4`, `5`, `6`.
  - Designed to be injected into model prompts or evaluation templates.
- `requirements.txt`
  - Added core dependencies for notebook + preprocessing pipeline.

## Setup

```bash
pip install -r requirements.txt
```

## Preprocess ASAP++ (Prompts 3-6)

Run:

```bash
python scripts/prepare_asap_prompts_3_6.py --input data/asap_plus_plus.csv --output data/asap_pp_3_6_prepared.csv
```

Useful options:

```bash
python scripts/prepare_asap_prompts_3_6.py \
  --input data/asap_plus_plus.csv \
  --output data/asap_pp_3_6_prepared.csv \
  --prompt-col prompt_id \
  --target-rows 7133 \
  --text-columns essay \
  --score-columns domain1_score \
  --sampling random \
  --seed 42
```

If your score ranges differ by prompt, pass your own JSON:

```bash
python scripts/prepare_asap_prompts_3_6.py \
  --input data/asap_plus_plus.csv \
  --output data/asap_pp_3_6_prepared.csv \
  --ranges-json config/score_ranges.json
```

Example `score_ranges.json`:

```json
{
  "3": { "min": 0, "max": 3 },
  "4": { "min": 0, "max": 3 },
  "5": { "min": 0, "max": 4 },
  "6": { "min": 0, "max": 4 }
}
```

## Rubric Mapping File

Path: `config/prompt_rubrics_3_6.json`

Intended use:
- Load by prompt id.
- Inject the `criteria` list into your model instruction template so scoring logic is prompt-aware.

## Recommended Next Steps

1. Add raw ASAP++ data to `data/` and run preprocessing script to generate the 7,133-row prepared subset.
2. Validate cleaned output:
   - confirm only prompts 3-6 are present,
   - confirm row count is exactly 7,133,
   - spot-check text cleaning and normalized score columns.
3. Update notebook to read the prepared CSV instead of dummy data.
4. Build a deterministic prompt template that includes:
   - prompt id,
   - prompt-specific rubric criteria from JSON,
   - strict output format for score parsing.
5. Run baseline evaluation (for example QWK or MAE on held-out split) and log prompt-wise performance.
6. Add a training/evaluation script so the workflow is reproducible outside the notebook.
