# Step 2 (Training): SNV Perturbation Modeling

`prismsnv snv_effect` trains the SNV perturbation model on top of RNA backbone representations and exports ranking/scoring outputs.

## 1. Run Command

```bash
prismsnv snv_effect -y /path/to/train_config.yaml
```

Multi-GPU distributed mode:

```bash
torchrun --nproc_per_node=4 -m prismsnv.cli snv_effect -y /path/to/train_config.yaml
```

---

## 2. Key YAML Fields

```yaml
result_folder: ./snv_result

snv_eff:
  adata_snv: /path/to/all_samples_merged_barcode_snv_matrix.h5ad
  ann_csv: /path/to/annotation.csv  # optional
  min_cnt: 100
  n_top: 50
  batch_size_train: 256
  num_epochs: 500
  latent_dim: 128
  snv_emb_dim: 64
  top_k_attention: 2000
  attn_batch: 256
  seed: 42
  celltype_key: cell_cluster
  cell_type_free: false
  freeze_backbone: false
  eval_only: false
  pair_chunk: 20000
  batch_key: sample
```

### 2.1 Required Fields

- `snv_eff.adata_snv`

`snv_eff.ann_csv` is optional. If omitted, scoring still runs and annotation metadata columns in the score CSVs are left empty.

Accepted `ann_csv` formats in the current code:

- `.csv` comma-delimited tables
- `.tsv` or `.txt` tab-delimited tables
- some whitespace-delimited text tables (fallback parsing), which is useful for certain ANNOVAR exports

Recommended content patterns:

- Either provide a direct SNV identifier column such as `name`, `snv`, `variant`, or `id`, with values like `chr1:123456_A>T`
- Or provide separate variant columns that can be combined by the script: `chr` / `#chr` / `chrom` / `#chrom`, `pos` / `position` / `start`, plus `ref` and `alt`

If present, the score export will try to carry through annotation fields such as `Func.refGene*`, `Gene.refGene*`, `GeneDetail.refGene*`, `ExonicFunc.refGene*`, `CLNDN`, `CLNALLELEID`, and `CLNSIG`.

### 2.2 Common Parameter Guidance

| Parameter | Role | Tuning Advice |
|---|---|---|
| `min_cnt` | Initial SNV filtering threshold | Lower for small datasets |
| `n_top` | Number of SNVs retained after initial filtering | Scale with dataset complexity |
| `top_k_attention` | Number of top-attention SNVs for scoring | Increase for larger cohorts |
| `attn_batch` | Batch size for attention/scoring | Memory-sensitive |
| `pair_chunk` | Sparse decode chunk size | Lower first when hitting OOM |
| `cell_type_free` | Skip cell-type-aware scoring | Useful for fast global screening |

---

## 3. Internal Stages

1. Load and filter SNV matrix (`min_cnt`, `n_top`).
2. Build `SNVPerturbationModel` (SNV embedding + attention + conditional perturbation).
3. Optionally load pretrained backbone weights.
4. Train and save model checkpoint.
5. Export attention ranking, cell-level scores, cell-type-level scores, and plots.

Additional notes:

- If `freeze_backbone=true`, encoder-side parameters are frozen while SNV-related components remain trainable.
- If `eval_only=true`, `snv_perturbation_model.pt` must already exist.
- RNA input is auto-loaded from `result_folder/finetune_aligned.h5ad`.

---

## 4. Distributed Execution Notes

Distributed mode is enabled when launched by `torchrun` with proper env vars (`RANK`, `WORLD_SIZE`, etc.). With an installed package, use `torchrun -m prismsnv.cli snv_effect ...` so each process enters the PrismSNV CLI.

Behavior highlights:

- Automatic DDP initialization (NCCL backend)
- Rank 0 handles key logging and checkpoint persistence
- Synchronization barriers/object broadcasts keep scoring inputs consistent across ranks

---

## 5. Main Outputs

- `snv_perturbation_model.pt`
- `top_snv_attention.csv`
- `snv_perturbation_scores_by_cell.csv`
- `snv_perturbation_scores.csv` when `cell_type_free=true`
- `top_snv_attention_by_celltype.csv` when `cell_type_free=false`
- `snv_perturbation_scores_by_celltype.csv` when `cell_type_free=false`
- `snv_cooccurrence_dedup_removed.csv` when `cell_type_free=false`
- `cell_cluster_marker_genes_top10.csv` when `cell_type_free=false`
- `adata_rna_latent_labeled.h5ad` when `cell_type_free=false`
- `cell_perturbation_scores.csv` when `cell_type_free=false`
- `cell_perturbation/<celltype>_cell_perturbation.csv` for populated high/low perturbation cell lists when `cell_type_free=false`
- `cell_perturbation_and_celltype_umap.pdf` when `cell_type_free=false`

Output branches by mode:

- `cell_type_free=true`: produces global `snv_perturbation_scores.csv` and skips clustering/aggregation outputs
- `cell_type_free=false`: produces cell-type-aware score files, co-occurrence de-duplication audit, per-cell perturbation tables, and UMAP outputs

---

## 6. Post-run Checks

At minimum, verify:

1. `snv_perturbation_model.pt` was created
2. `top_snv_attention.csv` is non-empty
3. score CSVs contain reasonable SNV/cell counts
4. visualization outputs were generated successfully
