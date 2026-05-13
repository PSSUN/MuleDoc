# Step 2 (Training): RNA Backbone Pretraining

`prismsnv pre_train` builds the RNA backbone and writes encoder-derived latent representations into the finetuning dataset used by `prismsnv snv_effect`.

## 1. Run Command

```bash
prismsnv pre_train -y /path/to/train_config.yaml
```

Positional config path is also supported:

```bash
prismsnv pre_train /path/to/train_config.yaml
```

---

## 2. Key YAML Fields

```yaml
result_folder: ./snv_result

pre_train:
  align:
    pretrain_adata: /path/to/pretrain.h5ad
    finetune_adata: /path/to/finetune.h5ad
    expected_doublet_rate: 0.05
    remove_scrublet: false
  training:
    num_epochs: 100
    latent_dim: 128
    mu_activation: exp
    use_nb: true
    batch_size: 512
    lr: 0.001
    weight_decay: 1e-5
    kl_warmup_epochs: 20
    grad_clip: 5.0
    device: cuda
    hvg_only: true
    batch_key: batch
    batch_emb_dim: 16
    lambda_adv: 0.0
```

### 2.1 Required Fields

- `pre_train.align.pretrain_adata`
- `pre_train.align.finetune_adata`
- `pre_train.training.num_epochs`

### 2.2 Common Parameter Recommendations

| Parameter | Typical Start | Notes |
|---|---|---|
| `latent_dim` | 128 | Latent representation size |
| `use_nb` | `true` | Recommended for raw UMI count modeling |
| `batch_size` | 256–1024 | Adjust by memory budget |
| `kl_warmup_epochs` | 20 | Helps stabilize KL term learning |
| `lambda_adv` | 0.0–1.0 | Adversarial strength for batch-effect mitigation |

### 2.3 Note

For pretraining, batch metadata must be stored in:

- `pretrain_adata.obs[batch_key]`

By default, `batch_key` is set to:

- `batch`

So the default expectation is:

- `pretrain_adata.obs["batch"]`

If you use a different field name, set it explicitly in YAML:

```yaml
pre_train:
  training:
    batch_key: sample   # example
```

Behavior note:

- If `pretrain_adata.obs[batch_key]` is missing, batch embedding/adversarial batch removal will not be applied.
- If you introduce external data for pretraining, it is recommended that `pretrain_adata` also includes the data represented in `finetune_adata` (or a sufficiently overlapping distribution), so representation transfer remains stable.
- If you do not want to introduce external data, set both `pretrain_adata` and `finetune_adata` to the same `.h5ad` file.

---

## 3. Internal Stages

1. Align gene space between pretrain and finetune RNA datasets.
2. Train `RNAOnlyBackbone` (VAE with NB reconstruction and optional adversarial branch).
3. Save backbone checkpoint.
4. Encode finetune RNA into `obsm["X_latent"]`.

Additional notes:

- If `hvg_only=true` and `highly_variable` exists, training uses HVGs only.
- If `batch_key` exists with >1 levels, batch embedding and adversarial removal can be enabled.
- Learning rate uses warmup + cosine schedule; early stopping is based on validation loss.

---

## 4. Outputs and Downstream Handoff

- `pretrain_aligned.h5ad`
- `finetune_aligned.h5ad`
- `rna_backbone_pretrained.pt`
- `rna_backbone_pretrained.pt.genes.npy`

Downstream usage:

- `finetune_aligned.h5ad` is auto-loaded by `prismsnv snv_effect` from `result_folder`
- `rna_backbone_pretrained.pt` is used for optional weight transfer in `prismsnv snv_effect`

---

## 5. Post-run Checks

At minimum, verify:

1. The four key output files exist in `result_folder`.
2. `finetune_aligned.h5ad` contains `obsm["X_latent"]`.
3. Logs do not report major fallback failures for latent precomputation.
