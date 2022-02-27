---
jupyter:
  jupytext:
    formats: ipynb,md
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.2'
      jupytext_version: 1.6.0
  kernelspec:
    display_name: Python 3
    language: python
    name: python3
---

# Alexander Adams
# aja149
# Homework #1: Words, Words, Words
# PPOL628 Text as Data


# Homework: State Your Assumptions

>    POLONIUS
>    What do you read, my lord?

>    HAMLET
>    Words, words, words

>    -- Hamlet, Act 2, Scene 2

## This homework deals with the assumptions made when taking text from its original "raw" form into something more computable.

*    Assumptions about the shape of text (e.g. how to break a corpus into documents)
*    Assumptions about what makes a token, an entity, etc.
*    Assumptions about what interesting or important content looks like, and how that informs our analyses.

## There are three parts:

1.    Splitting Lines from Shakespeare
2.    Tokenizing and Aligning lines into plays
3.    Assessing and comparing characters from within each play

*NB
This file is merely a template, with instructions; do not feel constrained to using it directly if you do not wish to.*

---


## Get the Data

Since the class uses dvc, it is possible to get this dataset either using the command line (e.g. dvc import https://github.com/TLP-COI/text-data-course resources/data/shakespeare/shakespeare.txt), or using the python api (if you wish to use python):

```python
#I could not get the dvc.api read function to work. It consistenly produced the following error:
#AttributeError: 'HashFile' object has no attribute 'get'

#I switched to this import method because it was the only way I could actually read in the text file.

#from dvc.api import read,get_url
#import pandas as pd

#txt = pd.read_csv('shakespeare.txt', sep = '\n', header = None)

#print(txt[:250])
```

```python
#Use relative path because shakespeare.txt is in parent directory
with open('..\..\..\shakespeare.txt') as f:
    lines = f.read()
```

Make sure this works before you continue! Either way, it would likely be beneficial to have the data downloaded locally to keep from needing to re-dowload it every time.

---


# Part 1

Split the text file into a table, such that:

    -each row is a single line of dialogue
    -there are columns for:
        *the speaker
        *the line number
        *the line dialogue (the text)

*Hint: you will need to use RegEx to do this rapidly. See the in-class "markdown" example!*

Question(s):

    *What assumptions have you made about the text that allowed you to do this?

---

```python
#Import the re package for regex
import pandas as pd
pd.set_option('display.max_rows', None)
pd.set_option('display.width', None)
pd.set_option('display.max_colwidth', None)
import re
```

The key assumption I am making about this document is that all the chunks of text have the format:

Speaker_Name: Text\n\n

My first step is then to split the text into chunks, using two new lines (\n\n) as the delimiter:

```python
text = re.split(r'\n\n',lines)
```

Next, convert that output to a pandas data frame for easier processing going forward.

```python
text = pd.DataFrame(text, columns = ['text'])
```

Then use the `split` method for the string attribute on the text column to split into two columns using the colon followed by a new line as the delimiter. Setting the 'expand' argument to true creates two columns from the resultant split.

```python
text = text['text'].str.split(':\n', 1, expand=True)
```

Then use the `split` method again to break each chunk up into individual lines, followed by the `explode` function to make each individual line its own row. Call `reset_index` with `drop = True` to drop the index column, which has the same value for all lines which were part of the same original chunk.

```python
text[1] = text[1].str.split('\n')
```

```python
text = text.explode(1).reset_index(drop = True)
```

Finally, reset the index to create a column of integer values for line numbers, and then add one to each item in that column because Python is a zero-indexed language. Rename the columns in the data frame for clarity. 

```python
text = text.reset_index().rename(columns = {'index': 'Line_Number', 
                                            0: 'Speaker',
                                            1: 'Text'})
text['Line_Number'] = text['Line_Number']+1
text.head(10)
```

----
## Part 2

#### You have likely noticed that the lines are not all from the same play! Now, we will add some useful metadata to our table:

    -Determine a likely source title for each line.
    -Add the title as a 'play' column in the data table.
    -Make sure to document your decisions, assumptions, external data sources, etc.

This is fairly open-ended, and you are not being judged completely on accuracy. Instead, think outside the box a bit as to how you might accomplish this, and attempt to justify whatever approximations or assumptions you felt were appropriate.

---

```python
#Just for fun, let's see a list of all unique speakers in the table:
text.Speaker.unique()
```

Some of the names are in all caps, while some are in sentence case. I'll adjust so they're all in sentence case.

```python
text['Speaker'] = text['Speaker'].str.title()
```

Through Googling, I found a table of all characters in all Shakepeare plays, along with the number of lines spoken by that character and the play they appear in. When I tried to scrape the table using the `read_html` function from `pandas`, I received a 403 error code, indicating forbidden access. Fortunately, I was able to take a much cruder approach and copy/paste the contents of the table into an Excel file, which I've committed to my branch/folder for this homework assignment. This dataset comes from playshakespeare.com, specifically this page: [https://www.playshakespeare.com/study/complete-shakespeare-character-list](https://www.playshakespeare.com/study/complete-shakespeare-character-list). All credit goes to the original compilers of the data.

```python
#Unsuccessful scraping attempt:
#characters = pd.read_html('https://www.playshakespeare.com/study/complete-shakespeare-character-list')
```

```python
characters = pd.read_csv('shakespeare_characters.csv')
```

```python
characters.head(10)
```

I need to remove the notations in parentheses in order to properly join my tables, so I'll adapt the string method I used in part 1.

```python
characters[['cter_clean','unneeded']] = characters['cter'].str.split(' \(', 1, expand = True)
```

```python
characters.head(5)
```

```python
#Drop unneeded columns
characters = characters[['cter_clean', 'Play', 'Lines']]
```

```python
text_play = pd.merge(text, characters, left_on = ['Speaker'], right_on = ['cter_clean'], how = 'left')
```

```python
text_play.head(30)
```

This probably achieved a decent amount of matching, but checking the head and tail of the data set reveals two problems:

1) Some lines are spoken by characters with ambiguous names ("All" for the ensemble in response to the First Citizen in Richard III), and so have no match in the character data set.

2) Shakespeare reused some character names across multiple plays. For example, there are characters named Antonio in The Merchant of Venice, Much Ado About Nothing, The Tempest, Twelfth Night, and The Two Gentlemen of Verona, and simply joining on character name is not enough to specify which one of those plays corresponds to that particular Antonio. 

Issue (1) has a fairly straightforward-ish solution which should appropriately label many of the NAs. If I assume that these lines are not in a random order, and consist of the full (or partial) dialogue of each play in the order that dialogue is spoken (i.e. a line from Romeo and Juliet is surrounded by other lines from Romeo and Juliet, and is not intermingled with lines from The Taming of the Shrew or Hamlet or King Lear), then I can compare the values of each NA line to the closest non-NA values preceding and following it, and use those to assign a play. For example, line 2, spoken by the ensemble (labeled "All") is preceded by a line identified as matching a character in Richard III, and is followed by a line matching a character in Richard III, and so is almost certainly itself a line from Richard III. I can also fill in the character name with the value from the 'Speaker' column in cases where that would be necessary. 

```python
text_play['prev_play'] = text_play['Play'].ffill()
text_play['next_play'] = text_play['Play'].bfill()
```

```python
import numpy as np
text_play['Play'] = np.where(text_play['prev_play'] == text_play['next_play'],
                             text_play['prev_play'],
                             text_play['Play'])
```

```python
text_play
```

```python
text_play['line_counts'] = text_play['Line_Number'].value_counts(sort = False)
text_play['line_counts'] = np.where(text_play['line_counts'] != 1,
                                    np.nan,
                                    text_play['Play'])
```

```python
text_play = text_play.drop(columns = ['prev_play', 'next_play'])
```

```python
text_play['prev_play'] = text_play['line_counts'].ffill()
text_play['next_play'] = text_play['line_counts'].bfill()
```

```python
text_play['Play1'] = np.where(text_play['prev_play'] == text_play['next_play'],
                                    text_play['prev_play'],
                                    text_play['Play'])
```

```python
text_play1 = text_play[['Line_Number','Speaker','Text','cter_clean']]
chars_na = text_play[text_play['Play'].isnull()]['Speaker'].unique()
possible_list = []
```

```python
for char in chars_na:
    possible = characters[characters['cter_clean'].str.contains(char)]
    #possible = pd.DataFrame(possible)
    possible['char'] = char
    possible_list.append(possible)
```

```python
possible = pd.concat(possible_list)
```

```python
text_play[text_play['Play'].isnull()]['Speaker'].unique()
```
