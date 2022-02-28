---
jupytext:
  formats: ipynb,md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.11.5
kernelspec:
  display_name: Python [conda env:root]
  language: python
  name: conda-root-py
---

# Homework: _State Your Assumptions_ 

> POLONIUS\
> _What do you read, my lord?_
> 
> HAMLET\
> _Words, words, words_
> 
>  -- _Hamlet_, Act 2, Scene 2

This homework deals with the assumptions made when taking text from its original "raw" form into something more _computable_.

- Assumptions about the _shape_ of text (e.g. how to break a corpus into documents)
- Assumptions about what makes a _token_, an _entity_, etc. 
- Assumptions about what interesting or important _content_ looks like, and how that informs our analyses.


There are three parts: 
1. Splitting Lines from Shakespeare
2. Tokenizing and Aligning lines into plays
3. Assessing and comparing characters from within each play

**NB**\
This file is merely a _template_, with instructions; do not feel constrained to using it directly if you do not wish to.

+++

## Get the Data

Since the class uses `dvc`, it is possible to get this dataset either using the command line (e.g. `dvc import https://github.com/TLP-COI/text-data-course resources/data/shakespeare/shakespeare.txt`), or using the python api (if you wish to use python)

```{code-cell} ipython3
from dvc.api import read,get_url
import pandas as pd

txt = read('resources/data/shakespeare/shakespeare.txt', 
           repo='https://github.com/TLP-COI/text-data-course')

print(txt[:250])
```

Make sure this works before you continue! 
Either way, it would likely be beneficial to have the data downloaded locally to keep from needing to re-dowload it every time.

+++

## Part 1

Split the text file into a _table_, such that 
- each row is a single _line_ of dialogue
- there are columns for
  1. the speaker
  1. the line number
  1. the line dialogue (the text)

_Hint_: you will need to use RegEx to do this rapidly. See the in-class "markdown" example!

Question(s): 
- What assumptions have you made about the text that allowed you to do this?

```{code-cell} ipython3
import pandas as pd
import re
```

```{code-cell} ipython3
# Use the first 250 characters as a sample
sample = txt[:250]
sample
```

Assumptions:
- the speaker begins with a capitalized letter and ends with a colon
- the dialogue begins with \n (new line) and ends with \n\n

```{code-cell} ipython3
patt_try = re.compile(
    "(^[A-Z].+?):"  # the speaker
    "\n{1}(.*?)\n{2}",  # the dialogue
    flags = re.S | re.M
)
```

```{code-cell} ipython3
# Find `patt_try` such that:  
matches_try = patt_try.findall(sample)
pd.DataFrame.from_records(matches_try, columns=['speaker', 'dialogue'])
```

```{code-cell} ipython3
# Check the last several lines to see whether they meet the assumptions
print(txt[-330:])
```

```{code-cell} ipython3
txt[-330:]
```

Here, the last line will be omitted according to the current assumptions because it ends with \n instead of \n\n.

```{code-cell} ipython3
# Fix the last line by adding an extra \n
new_line = '\n'
txt = txt + new_line
```

```{code-cell} ipython3
patt = re.compile(
    "(^[A-Z].+?):"  # the speaker
    "\n{1}(.*?)\n{2}",  # the dialogue
    flags = re.S | re.M
)
```

```{code-cell} ipython3
# Find `patt` such that: 
matches = patt.findall(txt)
df = pd.DataFrame.from_records(matches, columns=['speaker', 'dialogue'])
df.tail()
```

```{code-cell} ipython3
# Create a new column for line number
df['line'] = df.index + 1
```

```{code-cell} ipython3
# Reorder columns in the dataframe
df = df[['speaker','line','dialogue']]
df
```

## Part 2

You have likely noticed that the lines are not all from the same play!
Now, we will add some useful metadata to our table: 

- Determine a likely source title for each line
- add the title as a 'play' column in the data table. 
- make sure to document your decisions, assumptions, external data sources, etc. 

This is fairly open-ended, and you are not being judged completely on _accuracy_. 
Instead, think outside the box a bit as to how you might accomplish this, and attempt to justify whatever approximations or assumptions you felt were appropriate.

+++

## Part 3

Pick one or more of the techniques described in this chapter: 

- keyword frequency
- entity relationships
- markov language model
- bag-of-words, TF-IDF
- semantic embedding

make a case for a technique to measure how _important_ or _interesting_ a speaker is. 
The measure does not have to be both important _and_ interesting, and you are welcome to come up with another term that represents "useful content", or tells a story (happiest speaker, worst speaker, etc.)

Whatever you choose, you must
1. document how your technique was applied
2. describe why you believe the technique is a valid approximation or exploration of how important, interesting, etc., a speaker is. 
3. list some possible weaknesses of your method, or ways you expect your assumptions could be violated within the text. 

This is mostly about learning to transparently document your decisions, and iterate on a method for operationalizing useful analyses on text. 
Your explanations should be understandable; homeworks will be peer-reviewed by your fellow students.

```{code-cell} ipython3

```
