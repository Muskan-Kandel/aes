# Automated Essay Scoring with Mistral-7B

**Course:** Advanced Python for Data Science  
**Institution:** IOE, Thapathali Campus  
**Team:** Ayushma Devkota · Muskan Kandel · Purnima Bhandari

---

## Overview

This project implements an Automated Essay Scoring (AES) system that fine-tunes `Mistral-7B-Instruct-v0.2` using LoRA on the ASAP++ dataset. The system grades student essays across five traits and generates qualitative feedback.

The model predicts scores for:
- **Content** — relevance of argument and evidence used
- **Language** — grammar, spelling, and sentence structure
- **Prompt Adherence** — whether the student addressed the actual question
- **Narrativity** — logical flow and organisation of ideas
- **Holistic** — overall essay quality

All scores are normalised to a 0–10 scale.

---

## Project Structure

```
aes/
├── notebooks/
│   └── AES.ipynb              # main notebook — run this
├── data/                      # downloaded automatically by the notebook
├── outputs/                   # generated plots and CSVs
├── mistral_final_model.zip    # trained LoRA adapter — upload to Colab
├── requirements.txt
└── README.md
```

> `mistral_final_model.zip` should sit at the root of the `aes/` folder locally. When running on Colab, upload it to `/content/` before running Section 5.

---

## Datasets

| Dataset | Source | Description |
|---------|--------|-------------|
| ASAP | Kaggle 2012 competition | 12,976 essays across 8 prompts. Provides holistic scores |
| ASAP++ | Research extension | Trait-level scores (Content, Language, Adherence, Narrativity) for prompts 3–6 |

Only prompts 3–6 are used (7,097 essays) because ASAP++ annotations only cover these four prompts.

The data files are downloaded automatically inside the notebook via `gdown`. No manual download is needed.

---

## Model Architecture

| Component | Details |
|-----------|---------|
| Base model | `mistralai/Mistral-7B-Instruct-v0.2` |
| Fine-tuning method | QLoRA (4-bit quantized LoRA) |
| LoRA rank | r=16, alpha=32, dropout=0.05 |
| Target modules | q_proj, k_proj, v_proj, o_proj, gate_proj, up_proj, down_proj |
| Quantization | 4-bit NF4 with double quantization |
| Training steps | 200 steps (~0.23 epochs) |
| Loss function | Joint CE + MSE (λ=0.5) |
| Batch size | 2 per device, 4 gradient accumulation steps (effective batch=8) |
| Optimizer | paged_adamw_32bit |
| Learning rate | 1e-4 |

---

## How to Run

### On Google Colab (recommended)

1. Open `AES.ipynb` in Google Colab
2. Set runtime to **GPU** (T4 or better): Runtime → Change runtime type → GPU
3. Upload `mistral_final_model.zip` to `/content/` using the file panel on the left
4. Run all cells from top to bottom
5. Section 7 (training pipeline) is for documentation only — do not re-run it

### Local Setup

```bash
pip install -r requirements.txt
jupyter notebook notebooks/AES.ipynb
```

Note: Running locally requires a CUDA GPU with at least 15 GB VRAM. The notebook is designed for Colab.

---

## Requirements

```
transformers
datasets
peft==0.18.1
bitsandbytes
trl
accelerate
scikit-learn
scipy
matplotlib
seaborn
numpy
pandas
nltk
gdown
```

> `peft==0.18.1` must be this exact version. The saved adapter was trained with this version and will not load correctly with others.

---

## Results

Evaluation was performed on a 20-essay random sample (`random_state=42`) drawn from the 7,097-essay dataset.

| Trait | QWK | MSE | Pearson r |
|-------|-----|-----|-----------|
| Holistic | — | — | — |
| Content | — | — | — |
| Language | — | — | — |
| Adherence | — | — | — |
| Narrativity | — | — | — |

> Results depend on inference output format consistency. Fill in after running the notebook.

**QWK interpretation:**

| Range | Meaning |
|-------|---------|
| > 0.70 | Strong agreement |
| 0.50 – 0.70 | Moderate agreement |
| 0.30 – 0.50 | Weak agreement |
| < 0.30 | Poor agreement |

### Known Limitation

Training was limited to 200 steps (~0.23 epochs) due to Colab free-tier time constraints. Full convergence typically requires 3,000–5,000 steps (3–5 epochs). Current metrics reflect early-stage learning rather than the model's full potential.

---

## Output Files

After running the notebook, the following files are saved to `/content/AES_Project/outputs/`:

| File | Description |
|------|-------------|
| `handoff.csv` | 20 test essays with true and predicted scores |
| `metrics_summary.csv` | QWK, MSE, Pearson per trait |
| `plot1_scatter.png` | True vs predicted holistic scores |
| `plot2_distribution.png` | Score distribution comparison |
| `plot3_qwk.png` | QWK per trait bar chart |
| `plot4_heatmap.png` | Error heatmap by prompt and score range |
| `plot5_errors.png` | Prediction error distribution |
| `plot6_mae_prompt.png` | MAE by prompt |

---

## Notebook Sections

| Section | Description |
|---------|-------------|
| 1 | Environment setup, imports, GPU verification |
| 2 | Data loading from Google Drive |
| 3 | Data exploration and overlap verification |
| 4 | Preprocessing — merge, normalise, clean text, build rubrics |
| 5 | Load base model and attach trained LoRA adapter |
| 6 | Inference on 20 essays, score extraction, results |
| 7 | Training pipeline (documentation only — do not re-run) |
| 8 | Evaluation metrics — QWK, MSE, Pearson |
| 9 | Error analysis — worst/best predictions, bias check |
| 10 | Visualisations — 6 diagnostic plots |
| 11 | Export results |

---

## References

- Stab, C., & Gurevych, I. (2014). ASAP++ dataset
- Hu, E. et al. (2021). LoRA: Low-Rank Adaptation of Large Language Models
- Dettmers, T. et al. (2023). QLoRA: Efficient Finetuning of Quantized LLMs
- Mistral AI. (2023). Mistral-7B
