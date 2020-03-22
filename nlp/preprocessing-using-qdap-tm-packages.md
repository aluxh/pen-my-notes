# Cheatsheet about pre-processing text using qdap and tm packages

## Useful functions for cleaning the text

**qdap** package offers some text cleaning functions, which I find them useful.

- bracketX(): Remove all text within brackets (e.g. "It's (so) cool" becomes "It's cool")
- replace_number(): Replace numbers with their word equivalents (e.g. "2" becomes "two")
- replace_abbreviation(): Replace abbreviations with their full text equivalents (e.g. "Sr" becomes "Senior")
- replace_contraction(): Convert contractions back to their base words (e.g. "shouldn't" becomes "should not")
- replace_symbol() Replace common symbols with their word equivalents (e.g. "$" becomes "dollar")

**tm** package offers other useful preprocessing functions:

- removePunctuation(): Remove all punctuation marks
- removeNumbers(): Remove numbers
- stripWhitespace(): Remove excess whitespace


## Stopwords

To list the standard English stop words from **tm** package

```r
stopwords('en')
```

And, if you wish to expand or customize the stopword list:

```r
customized_stopwords <- c("word1", "word2", stopwords("en"))
```

Then use `removeWords()` function to remove the stopwords

```r
# Remove stop words from text
removeWords(text, new_stops)
```

## Word stemming

Word stemming and stem completion. For example,

```r
stemDocument(c("computational", "computers", "computation"))
#returns "comput" "comput" "comput".
```

Use `stemCompletion()` to reconstruct these word roots back into a known term. `stemCompletion()` accepts a character vector and a completion dictionary.

## Pre-processing workflow

### Learnings for tm package

```r
clean_corpus <- function(corpus){
  corpus <- tm_map(corpus, removePunctuation)
  corpus <- tm_map(corpus, stripWhitespace)
  corpus <- tm_map(corpus, removeNumbers)
  corpus <- tm_map(corpus, content_transformer(tolower))
  corpus <- tm_map(corpus, removeWords, c(stopwords("en"), "amp", "glass", "chardonnay", "coffee"))
  return(corpus)
}
```

### Learnings for qdap package

```r
Cleaning Data using QDAP:
qdap_clean <- function(x){
  x <- replace_abbreviation(x)
  x <- replace_contraction(x)
  x <- replace_number(x)
  x <- replace_ordinal(x)
  x <- replace_ordinal(x)
  x <- replace_symbol(x)
  x <- tolower(x)
  return(x)
}
```

## Terms Frequency

Using term-document matrix in the TM library, as well as its transpose, the document-term matrix, we will use it as the basis for some analysis. In order to analyze it we need to change it to a simple matrix like we did in chapter 1 using `as.matrix()`.

Calling `rowSums()` on your newly made matrix aggregates all the terms used in a passage. Once you have the rowSums(), you can sort() them with decreasing = TRUE, so you can focus on the most common terms.

Lastly, you can make a `barplot()` of the top 5 terms of term_frequency with the following code.

```r
barplot(term_frequency[1:5], col = "#C0DE25")
```

### Frequent Terms with qdap library

If you are OK giving up some control over the exact preprocessing steps, then a fast way to get frequent terms is with `freq_terms()` from qdap.
