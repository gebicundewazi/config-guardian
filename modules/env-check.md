---
module: env-check
description: "Environment availability check"
inputs: []
outputs: [environment_issues]
parallel: true
---

## Execution Logic

### Parallel Environment Checks

Commands run in parallel:

```bash
Rscript --version &
Rscript -e "for(pkg in c('ggplot2','survival','survminer','pROC','epiR','lme4','svglite')) { if(requireNamespace(pkg, quietly=TRUE)) cat(pkg, ': OK\n') else cat(pkg, ': MISSING\n') }" &
python --version &
python -c "import pandas; print('pandas:', pandas.__version__)" &
python -c "import matplotlib; print('matplotlib:', matplotlib.__version__)" &
python -c "import openpyxl; print('openpyxl:', openpyxl.__version__)" &
python -c "import docx; print('python-docx:', docx.__version__)" &
git --version &
git config user.name &
git config user.email &
wait
```

## Output Format

| Component | Status | Details |
|-----------|--------|---------|

## Error Handling

- Command not found → report as MISSING, don't error
- Version check fails → report as WARNING
