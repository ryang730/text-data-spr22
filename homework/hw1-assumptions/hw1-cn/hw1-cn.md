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
    display_name: Python [conda env:text-data-class]
    language: python
    name: conda-env-text-data-class-py
---

```python
# Libraries
from dvc.api import read, get_url
import pandas as pd
import numpy as np

import re

import seaborn as sns


```

# Get the Data

```python
# Read from repo using DVC
txt = read('resources/data/shakespeare/shakespeare.txt', 
           repo='https://github.com/TLP-COI/text-data-course')

# # Can also read from local file as wel
# f = open('/Users/chau/text-data-course/shakespeare.txt','r')
# txt = f.read()

print(txt[:250])
```

# Part 1

Split the text file into a table, such that

* each row is a single line of dialogue
* there are columns for

1. the speaker

2. the line number

3. the line dialogue (the text)

Hint: you will need to use RegEx to do this rapidly. See the in-class "markdown" example!

#### Question(s): What assumptions have you made about the text that allowed you to do this?


```python
# Print first 250 letters
txt[:250]
```

### Step 1: Split into individual dialogues

**Assumption 1a:** A dialogue looks like a paragraph - meaning it ends with a double line break <code>\n\n</code>. 

I can use <code>re.split</code> to split the text into dialogues.

```python
# Print first dialogue
re.split(r"\:\n", txt)[0]
```

```python
#Print last dialogue
re.split(r"\n\n", txt)[-1]
```

I will use the text of the last dialogue as a sample for the functions I'll define

```python
# Use last dialogue as sample
sample = re.split(r"\n\n", txt)[-1]

# Pretty print
print(sample)
```

```python
# Ugly print
sample
```

### Step 2: Split dialogue into speaker and dialogue text

```python
# Count number of dialogues in the text
num_dia = len(re.split(r"\n\n", txt))

num_dia
```

**Assumption 1b:** Within a dialogue, speaker name is something that comes before <code>:\n</code> in each split dialogue. I can use <code>^[^:]*</code> to extract it

```python
# Function to get speaker name
def get_speaker_name(dialogue):
    '''Get the speaker name from each dialogue
    Speaker name is defined as anything between the first instance of colon'''
    speaker_name = re.compile(r'^[^:]*').match(dialogue).group()
    return speaker_name
```

```python
# Test the function out
get_speaker_name(sample)
```

**Assumption 1c:** The dialogue content is anything that's not the speaker name in a dialogue. I can try to use a negative look ahead to solve this with regex.

```python
# Function to get dialogue content:
def get_dialogue_content(dialogue):
    '''Get content of the dailogue'''
    dialogue_content = re.split(":", dialogue)[1]
    return dialogue_content
```

**Hacky solution** Since I can't figure out a negative look ahead to get the dialogue content and want to move on with the rest of the homework, I will use <code>re.split</code> to get the speaker name & dialogue. If I have more time next week I will revisit <code>get_dialogue_content</code>


### Step 3: Put it all together

```python
# Function to get the dialogue from the text
def get_dialogue(text):
    '''Input: the full text 
    Returns: a list of list'''
    # Create enumerate object to count line number
    enum = enumerate(re.split(r"\n\n", text))
    
    # List comprehension to put the line number and the 2 custom functions above together 
    res = [[i,get_speaker_name(j),get_dialogue_content(j)] for i,j in enum]
    return res
```

```python
# Check if it works
dia_list = get_dialogue(txt)
dia_list[:3]
```

```python
# Convert to pd
df = pd.DataFrame(dia_list, columns =['line_no','speaker','dialogue'])
# Check 
df[:3]
```

```python
# Remove linebreaks
df = df.replace(r'\n',' ', regex=True) 
```

```python
# Remove trailing/ leading white space from speaker and dialogue
df.speaker = df.speaker.str.strip()
df.dialogue = df.dialogue.str.strip()
```

```python
df[:5]
```

# Part 2

You have likely noticed that the lines are not all from the same play! Now, we will add some useful metadata to our table:

* Determine a likely source title for each line

* add the title as a 'play' column in the data table.

* make sure to document your decisions, assumptions, external data sources, etc.

This is fairly open-ended, and you are not being judged completely on accuracy. Instead, think outside the box a bit as to how you might accomplish this, and attempt to justify whatever approximations or assumptions you felt were appropriate.

```python
# Number of unique speakers
df.speaker.nunique()
```

```python
# List of unique speakers
speaker_list = df.speaker.unique()
```

```python
speaker_list
```

**Assumption 2a** It looks like names written in all caps might be main characters?

```python
# Create a column in df to determine if the speaker name is all caps
df['name_cap']=df.apply(lambda x: x['speaker'].isupper(), axis=1)
```

```python
# 20 speakers with the most lines
df.speaker.value_counts().head(20)
```

```python
# 20 spekers with the fewest line
df.speaker.value_counts().tail(20)
```

```python
# Add number of line count for each speaker in df
df['line_count'] = df.groupby(['speaker'])['line_no'].transform('count')
```

```python
df.head()
```

```python
# Distribution of line count
sns.histplot(df,kde=True, x='line_count')
```

Mahority of characters have very few lines. Next I want to see the distribution split by name capitalization

```python
# Distribution of line count 
sns.histplot(df,kde=True, x='line_count', hue = 'name_cap')
```

It looks like there are characters with a significant number of lines whose name where not capitalized. Check to see who they are.

```python
# Speakers with more than 50 lines & non-cap names:
df.loc[(df.line_count>=50)&(df.name_cap==False)].speaker.unique()
```

Capitalized names are for named characters (not necessarily main?)

```python
# Print the non-cap names
df.loc[(df.name_cap==False)].speaker.unique()
```

There are a few "ghost of" named charactes that are not all caps


For each speaker, I want to get the max line number and min line number - this should give me a general idea of which speaker belongs in which play

```python
# Get min and max line number
df['line_min']=df.groupby(['speaker'],sort=False)['line_no'].transform(min)
df['line_max']=df.groupby(['speaker'],sort=False)['line_no'].transform(max)
```

```python
df[40:70]
```

Generic characters without capped names are not unique to each stories, so their line min & line max are not important in determining the play title

```python
# Get a subset of dialogues with named characters only
df_main = df[df.name_cap==True]
```

```python
df_main.head(20)
```

```python

```

**Assumption 2b** Named-characters whose line_max and line_min overlap are from the same play


# Part 3

Pick one or more of the techniques described in this chapter:

* keyword frequency

* entity relationships
    
* markov language model
    
* bag-of-words, TF-IDF
    
* semantic embedding

make a case for a technique to measure how important or interesting a speaker is. The measure does not have to be both important and interesting, and you are welcome to come up with another term that represents "useful content", or tells a story (happiest speaker, worst speaker, etc.)

Whatever you choose, you must

1. document how your technique was applied

2. describe why you believe the technique is a valid approximation or exploration of how important, interesting, etc., a speaker is.

3. list some possible weaknesses of your method, or ways you expect your assumptions could be violated within the text.

This is mostly about learning to transparently document your decisions, and iterate on a method for operationalizing useful analyses on text. Your explanations should be understandable; homeworks will be peer-reviewed by your fellow students.




```python

```

```python

```

```python

```

```python

```

```python
df
```

```python

```

```python

```

```python

```

```python

```

```python

```
