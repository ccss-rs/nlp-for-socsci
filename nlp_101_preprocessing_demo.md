---
title: "Summarise Tables"
output: 
  learnr::tutorial:
    progressive: true
    allow_skip: true
runtime: shiny_prerendered
description: Learn core commands for text pre-processing.
---

```{r setup, include=FALSE}
library(readr)
library(dplyr)
library(stringr)
library(tidytext)
library(SnowballC)
library(learnr)
tutorial.options(exercise.completion = FALSE)

nyc <- read_csv("nyc_10k.csv")
```

In this demo, we'll be walking through how to perform the fundamentals of text 
pre-processing within R. Let's briefly go over the core packages that I've pre-downloaded 
into our session:
* **readr** facilitates the easy uploading of external data files such as CSVs
* **dplyr** promotes streamlined data frame creation and manipulation
* **stringr** is all about working with text character strings
* **tidytext** provides more specialized functions to prepare text data for various NLP use cases
* **SnowballC**

These libraries are all popular choices for performing essential text pre-processing
functions. I'll also load in our working data set of 10,000 comments scraped from
r/nyc.

```{r, eval=FALSE}
library(readr)
library(dplyr)
library(stringr)
library(tidytext)
library(SnowballC)

nyc <- read_csv("nyc_10k.csv")
```

Let's check the character encoding of our r/nyc text data. The "body" column of the
loaded data frame features the text of the scraped r/nyc comments. 

``` {r encoding, exercise = TRUE, exercise.setup="setup"}
guess_encoding(nyc$body)
```

Our 1 confidence score on the UTF-8 encoding for the text column of our data frame 
assures us that the following pre-processing methods should work without 
problems around character encoding. Let’s now inspect our data set in more detail.

```{r data}
nyc 
```

This provides us with the first 10 records of our data set. “id” is the unique 
identifier for each comment, “author” provides the Reddit username who wrote the 
comment, “body” features the comments themselves, “score” refers to the 
Reddit system of being able to “upvote” or “downvote” comments similar to a 
like/dislike framework, and “date” as the year, month, and day the comment was 
posted. 

You'll notice with a preliminary view on the comments that there's unwanted noise
within the text, such as '`\n`' referring to line breaks written within HTML. Let's
use a stringr function to remove all instances of '`\n`' from the comment text. 

```{r remove, exercise = TRUE, exercise.setup="setup"}
nyc$body <- str_remove_all(nyc$body, "\n")
```
This successfully removes all instances of “`\n`” within the comments. There's a 
wide range of additional types of noise a researcher may want to remove from text. 
While our last str_remove_all command is quite effective at removing exact matches
with '`\n`', regexes are a particularly helpful alternative for matching a wider range
of character types to delete within one function call. 

One common use case for regexes within text pre-processing is to remove all types 
of punctuation, since many NLP methods would otherwise consider tokenized punctuation
to be there own words. Luckily, there's special regex character classes that specialize
in matching with groups of characters. Let's use one to identify and remove all 
punctuation from our text as follows. 

```{r regex, exercise = TRUE, exercise.setup="remove"}
nyc$body <- str_remove_all(nyc$body, '[:punct:]')
```

It’s common within text pre-processing to convert all words to lowercase since 
capitalization may lead an NLP method to consider the same words as separate 
entities based on casing alone. You may also consider combining n-grams into 
single word units, such as concatenating all instances of the bi-gram “New York” 
into the single token of “New_York.” Let's learn two more stringr functions to 
accomplish both of these goals within our comment data. 

```{r lowerreplace, exercise = TRUE, exercise.setup="regex"}
nyc$body <- str_to_lower(nyc$body)
nyc$body <- str_replace_all(nyc$body, "New York", "New_York")
```

*Tokenization* refers to the segmentation of text components into unit such as 
individual words, sentences, paragraphs, characters, or n-grams. Let's use
the tidytext package to create a new data frame of our r/nyc comments split
into word tokens. 

```{r unnest, exercise = TRUE, exercise.setup="lowerreplace"}
tokens <- nyc %>%
    unnest_tokens("word", body)
```
Our 100,000 original comments are now almost 3.5 million rows of individual word 
tokens. By default, unnest_tokens lowercases the tokens and removes punctuation 
from the original string, which would allow you to skip our previous regex string
removal step.

Let's now consider the optional pre-processing steps we covered in the slides. 
First is *stopword removal*, which refers to common words such as "and", "the", 
"but", and so forth that we'd like to remove from our text data since we consider
them semantically irrelevant to our core NLP interests. Tidytext comes with a 
pre-made common stopwords list that we can call and apply directly to our generated
tokens data frame. 

```{r stopwords, exercise = TRUE, exercise.setup="unnest"}
data(stop_words)
    
tokens <- tokens %>%
  anti_join(stop_words)    
```

*Stemming* is used to manipulate text into its root word form, such as "swimmers"
and "swimming" being stemmed to "swim". There’s a variety of different stemming 
algorithms available with the most commonly used being the Porter stemmer. We can
use the methods from the SnowballC package to stem our own tokenized r/nyc comments 
as follows. 

```{r stemming, exercise = TRUE, exercise.setup="unnest"}
stems <- tokens %>%
  mutate(stem = wordStem(word)) %>%
  count(stem, sort = TRUE)
```

Finally, parts-of-speech (POS) tags identifies the grammatic and lexical classes 
that words in your text data belong to. There's a variety of different libraries 
that follow different POS tag rules, with the "udpipe"- referring to the Universal
Dependencies tagging schema- being a popular library choice for R. 

We won't run this code chunk within the demo itself since it usually takes some
time to process, but feel free to experiment with POS tags on your own time 
following the below work flow. 

```{r partsofspeech, eval=FALSE}
library("udpipe")

## Donwload & load the POS tags for English
model <- udpipe_download_model(language = "english")
model <- udpipe_load_model(model)

## Annotation will take quite some time, particularly if you're using the 
## non-stopword version of our tokenized dataset 
tags <- udpipe_annotate(model, x = tokens$word)

## The generated "upos" column in the following dataframe will have each token's 
## identified POS tags.
pos <- as.data.frame(tags)
```

You've now completed the pre-processing demo and established a working knowledge
of how to perform the fundamentals of text data pre-processing in R. Congrats!
