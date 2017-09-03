Discursos Presidenciales
================

GitHub Documents
----------------

This is an R Markdown format used for publishing markdown documents to GitHub. When you click the **Knit** button all R code chunks are run and a markdown file (.md) suitable for publishing to GitHub is generated.

Including Code
--------------

You can include R code in the document as follows:

``` r
txt <- readLines("data/00.txt",encoding="UTF-8")
#txt = iconv(txt, to="ASCII//TRANSLIT")
#corpus <- Corpus(VectorSource(txt))
txt00 = readLines("data/00.txt",encoding="UTF-8") %>% tibble()
txt00 = txt00 %>% filter(txt != "")
#txt = iconv(txt, to="ASCII//TRANSLIT")
corpus <- Corpus(VectorSource(txt00))
d  <- tm_map(corpus, tolower)
d  <- tm_map(d, stripWhitespace)
d <- tm_map(d, removePunctuation)
d <- tm_map(d, removeWords, stopwords("spanish"))
tdm <- TermDocumentMatrix(d)

findFreqTerms(tdm, lowfreq=20)
```

    ##  [1] "acciÃ"       "constituciÃ" "derecho"     "econÃ"       "espaÃ"      
    ##  [6] "gobierno"    "hacer"       "ley"         "libertad"    "mica"       
    ## [11] "paÃ"         "polÃ"        "prÃ"         "seguridad"   "sistema"    
    ## [16] "social"      "sociedad"    "tica"        "tico"

Including Plots
---------------

You can also embed plots, for example:

``` r
m <- as.matrix(tdm)
v <- sort(rowSums(m),decreasing=TRUE)
df <- data.frame(word = names(v),freq=v)
```

![](exploratory_files/figure-markdown_github-ascii_identifiers/wordcloud-1.png)

Note that the `echo = FALSE` parameter was added to the code chunk to prevent printing of the R code that generated the plot.
