---
jupyter:
  jupytext:
    formats: ipynb,md
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.3'
      jupytext_version: 1.13.6
  kernelspec:
    display_name: Python 3 (ipykernel)
    language: python
    name: python3
---

Alex Adams, Madeline Kinnaird, Vince Egalla

```python
import pandas as pd
from pathlib import Path
import re
```

#### regex

^#+

^#+\s([\w:" ]*)\s([\w:' -]*)\s

^#+\s([\w:" ]*)\s([\w:' -]*)\s+(.*?)

```python
from pathlib import Path
```

```python
gold = pd.read_csv('../output.csv')
def test_md_df(df):
    levels=gold.level.tolist()
    contents = gold.content.tolist()
    titles = gold.title.tolist()
    
    dt.validate(df.columns, gold.columns.tolist())
    dt.validate(df.level, levels)
    dt.validate(df.title, titles)
    dt.validate(df.content, contents)
    
    print('all good!')
gold
```
