# Mule User Documentation

This documentation describes the complete Mule pipeline:
**BAM-to-VCF Calling → SNV-to-Barcode Matrix Construction → RNA Backbone Pretraining → SNV Perturbation Modeling**.

## Reader Guide

- If this is your first time using Mule, read in this order:
  1. `Introduction`
  2. `Getting Started`
  3. `Workflow Overview`
  4. `End-to-End Example`
- If you already know the basics, jump directly to module pages:
  - Preprocessing: BAM-to-VCF Calling, SNV-to-Barcode Matrix Construction
  - Training: RNA Backbone Pretraining, SNV Perturbation Modeling

```{toctree}
:maxdepth: 2
:caption: Documentation

introduction
getting-started
workflow-overview
preprocess-bam2vcf
preprocess-snv2barcode
train-pre-train
train-snv-eff
end-to-end
outputs-and-faq
contact
```
