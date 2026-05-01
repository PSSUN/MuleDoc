# PrismSNV User Documentation

This documentation describes the complete PrismSNV pipeline:
**BAM-to-VCF Calling → SNV-to-Barcode Matrix Construction → RNA Backbone Pretraining → SNV Perturbation Modeling**.

## Reader Guide

- If this is your first time using PrismSNV, read in this order:
  1. [Introduction](introduction.md)
  2. [Getting Started](getting-started.md)
  3. [Workflow Overview](workflow-overview.md)
  4. [End-to-End Example](end-to-end.md)
- If you already know the basics, think of the runtime flow as just two steps:
  - Step 1 (Preprocessing): BAM-to-VCF Calling, then SNV-to-Barcode Matrix Construction
  - Step 2 (Training): RNA Backbone Pretraining, then SNV Perturbation Modeling

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
