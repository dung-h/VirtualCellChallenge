# Virtual Cell Challenge — scGPT Embedding Diffusion Baseline

A simple, reproducible baseline for the **Virtual Cell Challenge (VCC)** that generates perturbation-specific single-cell gene expression predictions using:

- a **streamed training mean profile** (memory-friendly)
- **scGPT gene embeddings** to propagate perturbation effects via **cosine similarity**
- **log-normal per-cell sampling** + **median UMI matching**
- export to **AnnData (.h5ad)** and packaging to **.vcc** via `cell-eval prep`

> This is intentionally a lightweight baseline (not a full generative model). Great as a starting point for ablations and rapid iteration.

## Method (high level)
Given a target gene `t` and the global mean expression vector `μ`:

1. Compute `μ` by streaming over the training AnnData (no dense load).
2. Align dataset gene names to scGPT vocab; missing genes use the **mean embedding** fallback.
3. Compute cosine influence of target gene embedding against all genes:
   - `influence_i = cos(e_i, e_t)` (clipped)
4. Apply embedding-based modulation and knockdown:
   - `μ' = μ * (1 + α * influence)`
   - `μ'_t = k * μ'_t`
5. Sample `n_cells` with log-normal noise and rescale to match the required **median UMI per cell**.
6. Concatenate all perturbations + optional `non-targeting` controls; write `.h5ad`.
7. Convert `.h5ad` → `.vcc` using `cell-eval prep`.

## Repository contents
- `baseline_with_scgpt_embedding.ipynb`: end-to-end notebook
- `scgpt_runs/`: generated predictions
- `submissions/`: final `.vcc` files (optional)

## Setup
### Environment
Python >= 3.10 recommended.

Install dependencies (minimal):
```bash
pip install numpy pandas scanpy anndata scipy torch cell-eval gdown
