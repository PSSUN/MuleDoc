# Utility: Get Config Template

`prismsnv get_template` writes a `train_config.yaml` template into the current directory (or a specified path). Run this before configuring `prismsnv pre_train` and `prismsnv snv_effect`.

## 1. Run Command

```bash
prismsnv get_template
```

By default this writes `train_config.yaml` in the current working directory. Use `--output` to choose a different path:

```bash
prismsnv get_template --output /path/to/my_train_config.yaml
```

Use `--force` to overwrite an existing file:

```bash
prismsnv get_template --output train_config.yaml --force
```

## 2. Argument Reference

| Argument | Default | Description |
|---|---|---|
| `-o` / `--output` | `train_config.yaml` | Destination path for the generated template |
| `--force` | off | Overwrite the destination file if it already exists |

## 3. Next Steps

After generating the template, fill in the paths and parameters, then run:

```bash
prismsnv pre_train -y train_config.yaml
prismsnv snv_effect -y train_config.yaml
```

Refer to {doc}`train-pre-train` and {doc}`train-snv-eff` for full YAML field references.
