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
import re
```

# Get the Data

```python
from dvc.api import read, get_url
import pandas as pd

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

```python
# Function to get the dialogue from the text
def get_dialogue(text):
    
    '''Input: the full text 
    Returns: a dictionary of dialogue number and content'''

    # Create enumerate object to count line number
    enum = enumerate(re.split(r"\n\n", text))
    
    # Dict comprehension 
    dict_dia = dict((i,j) for i,j in enum)

    return dict_dia
```

```python
get_dialogue(sample)
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

```python
# Function to split a dialogue into speaker name & dialogue content
def split_dialogue(dialogue):
    '''Split a dialogue into speaker name and dialogue content
    Returns a tuple'''
    speaker_name = re.split(":", dialogue)[0]
    dialogue_content = re.split(":", dialogue)[1]
    return speaker_name, dialogue_content
```

```python
# Test the function out
split_dialogue(sample)
```

### Step 3: Put it all together


# Part 2

You have likely noticed that the lines are not all from the same play! Now, we will add some useful metadata to our table:

* Determine a likely source title for each line

* add the title as a 'play' column in the data table.

* make sure to document your decisions, assumptions, external data sources, etc.

This is fairly open-ended, and you are not being judged completely on accuracy. Instead, think outside the box a bit as to how you might accomplish this, and attempt to justify whatever approximations or assumptions you felt were appropriate.

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
