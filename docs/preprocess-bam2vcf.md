# Step 1 (Preprocessing): BAM-to-VCF Calling

`prismsnv bam2vcf` calls the packaged BAM-to-VCF pipeline from an installed PrismSNV environment. It calls SNVs from BAM files and removes known RNA editing sites in the final filtering step.

## 1. Command and Arguments

```bash
prismsnv bam2vcf \
  --outer-jobs <OUTER_JOBS> \
  --inner-threads <INNER_THREADS> \
  --reference <reference.fa> \
  --varscan-jar <VarScan.jar> \
  --rna-edit-bed <RNA_editing.bed> \
  --out-dir <output_dir> \
  --bam-files <bam1> [bam2 ...]
```

### Argument Reference

| Argument | Meaning | Typical Values |
|---|---|---|
| `--outer-jobs` | Number of BAMs processed in parallel (sample-level) | 4 / 6 / 8 |
| `--inner-threads` | samtools threads per BAM | 2 / 4 / 8 |
| `--reference` | Reference genome FASTA | `hg38.fa` |
| `--varscan-jar` | Path to VarScan JAR | `/path/VarScan.jar` |
| `--rna-edit-bed` | Known RNA editing BED file | `RNA_editing.bed` |
| `--out-dir` | Output directory | `./snv_call_out` |
| `--bam-files` | One or more BAM files | `sample1.bam sample2.bam` |

### Example

```bash
prismsnv bam2vcf \
  --outer-jobs 6 \
  --inner-threads 4 \
  --reference genome.fa \
  --varscan-jar VarScan.jar \
  --rna-edit-bed RNA_edit.bed \
  --out-dir ./out \
  --bam-files sample1.bam sample2.bam
```

The command requires `bash`, `samtools`, `bedtools`, `java`, and `awk` in `PATH`. On Windows, run it in an environment where Bash can access the input files, such as WSL or Git Bash.

---

## 2. Per-sample Processing Steps

1. `samtools view -F 1804 -q 20` filters BAM reads.
2. `samtools index` creates index for the filtered BAM.
3. `samtools mpileup -B -q 20 -Q 20` generates pileup.
4. `VarScan mpileup2snp --min-coverage 8 --min-var-freq 0.01 --min-reads2 3` calls SNVs.
5. `bedtools intersect -v` removes known RNA editing sites.

### Threshold Summary

| Threshold | Step | Effect |
|---|---|---|
| `-q 20` | samtools view/mpileup | Filters low mapping-quality reads |
| `-Q 20` | samtools mpileup | Filters low base-quality observations |
| `--min-coverage 8` | VarScan | Excludes low-depth loci |
| `--min-var-freq 0.01` | VarScan | Keeps variants with allele frequency ≥ 1% |
| `--min-reads2 3` | VarScan | Requires at least 3 ALT-supporting reads |

---

## 3. Parallelization and Performance Notes

The script uses two levels of parallelism:

- Outer level: `--outer-jobs` (how many BAM files run simultaneously)
- Inner level: `--inner-threads` (threads used by samtools per BAM)

Practical guidance:

- If machine CPU core count is `N`, keep `outer_jobs * inner_threads` near `N`.
- On slow storage, prefer moderate `outer_jobs` and slightly higher `inner_threads`.

---

## 4. Built-in Input Validation

Before execution, the script checks:

1. Existence of `reference.fa`, `VarScan.jar`, and `RNA_editing.bed`
2. Existence of each BAM input
3. Existence of BAM index (`sample.bam.bai` or `sample.bai`)
4. Chromosome naming compatibility between FASTA and BED (`chr*` vs non-`chr*`)

If required files are missing, the script exits early.

---

## 5. Output Naming (Per Sample)

- `{sample}.f1804q20.bam`
- `{sample}.f1804q20.bam.bai`
- `{sample}.f1804q20.mpileup`
- `{sample}.f1804q20.vcf`
- `{sample}.f1804q20.no_rna_editing.vcf` (recommended downstream input)

In most workflows, `no_rna_editing.vcf` is used as `samples.<name>.vcf` in the `prismsnv snv2barcode` configuration.

---

## 6. Common Failure Cases

- Missing `.bai` index for BAM files
- Mismatched chromosome naming between reference FASTA and RNA editing BED

Recommended troubleshooting order:

1. Confirm command has all required arguments
2. Confirm all file paths exist
3. Confirm BAM index presence and naming compatibility
4. Confirm runtime tools are available: `samtools`, `bedtools`, `java`
