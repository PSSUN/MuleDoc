# End-to-End Execution Example

## 1. End-to-End Commands

```bash
# 1) BAM -> VCF
bash src/preprocess/bam2vcf.sh \
  6 4 \
  /path/to/genome.fa \
  /path/to/VarScan.jar \
  /path/to/RNA_editing.bed \
  ./snv_call_out \
  /path/to/sample1.bam /path/to/sample2.bam

# 2) VCF/BAM/CB -> merged barcode×SNV h5ad
python src/preprocess/snv2barcode.py src/preprocess/snv2barcode_config.yaml

# 3) Pretrain RNA backbone
python src/train/pre_train.py -y src/train/train_config.yaml

# 4) Train and score SNV perturbation effects
python src/train/snv_effect.py -y src/train/train_config.yaml
```

## 2. Checkpoints After Each Step

### After Step 1

- Each sample should have: `*.f1804q20.no_rna_editing.vcf`

### After Step 2

- Expected output: `all_samples_merged_barcode_snv_matrix.h5ad`

### After Step 3

- Expected output: `finetune_aligned.h5ad`
- Expected output: `rna_backbone_pretrained.pt`

### After Step 4

- Expected output: `snv_perturbation_model.pt`
- Expected output: attention/score CSV files

## 3. Common Bottlenecks

- Step 1 is slow: usually large BAM files or storage I/O bottlenecks; reduce `OUTER_JOBS` first.
- Step 2 is slow: often caused by a very large SNV union or overly aggressive `threads`.
- Step 4 OOM: reduce `batch_size_train` and `pair_chunk` first.

Before running step 4, confirm that both `all_samples_merged_barcode_snv_matrix.h5ad` and `finetune_aligned.h5ad` are available.
