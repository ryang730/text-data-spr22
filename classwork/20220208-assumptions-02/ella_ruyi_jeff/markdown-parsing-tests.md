---
jupytext:
  formats: ipynb,md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.13.6
kernelspec:
  display_name: Python [conda env:text-data-class]
  language: python
  name: conda-env-text-data-class-py
---

```{code-cell} ipython3
import pandas as pd
import datatest as dt
import re
from pathlib import Path
```

```{code-cell} ipython3
print(Path('fixture.md').read_text())
```

```{code-cell} ipython3
patt = re.compile(
    "(^#+)"
    "\s([\w -:]*)"
    "\s+(.*?)"
    "(?=\n)",
    flags = re.S | re.M
)
```

```{code-cell} ipython3
matches = patt.findall (
    """
# This is a Markdown Title
this is _italicized content_.

## This is a Level 2 Subtitle
What more is there to say?
    """
)

output = pd.DataFrame.from_records(matches, columns=['level', 'title', 'content'])
```

```{code-cell} ipython3
output.to_csv('output.csv')  
```

```{code-cell} ipython3
gold = pd.read_csv('output.csv')
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

```{code-cell} ipython3
test_md_df(gold)
```

```{code-cell} ipython3

```
