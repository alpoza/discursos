TextMining - Discursos Presidenciales tidy way
================

R Markdown
----------

``` r
require(tidyverse)
require(tidytext)
require(ggrepel)
require(tm)
```

``` r
filenames <- list.files(".",pattern="*.txt")
discursos_raw = map(filenames,~readLines(.x,encoding="UTF-8")) %>% 
  map2(filenames, ~tibble(text=.x, docid=.y)) %>% 
  map_df(rbind)

# Replaces varios
discursos_raw = discursos_raw %>% 
  filter(text != "") %>% 
  mutate(text = gsub("SS.SS", "Sus_Señorías", text)
         ,text = gsub("SS. SS", "Sus_Señorías", text)
         )
discursos_raw 
```

    ## # A tibble: 1,978 x 2
    ##                                                                           text
    ##                                                                          <chr>
    ##  1                                                                          X0
    ##  2 Muchas gracias, señor Presidente. Con su venia. Señoras y señores Diputados
    ##  3 El cambio político realizado en nuestro país ha sido profundo y sincero. Pe
    ##  4 Se trata, por  consiguiente, de saber realizar el cambio social con sinceri
    ##  5 El cambio político se verificó en torno a un eje de sensatez consistente en
    ##  6 Con esta voluntad, me  permito invitar a Sus_Señorías. a avanzar en la defi
    ##  7 La cuestión, en la realidad, una vez más, consiste en averiguar si, a uno y
    ##  8 En síntesis, éste es el cuadro y en este gran marco se inscribe la gran tar
    ##  9 Y tenemos voluntad, fortaleza y experiencia política para serlo. La oportun
    ## 10 Nos enfrentamos con una situación nueva porque iniciamos una nueva legislat
    ## # ... with 1,968 more rows, and 1 more variables: docid <chr>

``` r
discursos = discursos_raw %>%
  group_by(docid) %>%
  mutate(linenumber = row_number()) %>% 
  ungroup() %>% 
  unnest_tokens(word, text) 

add.stops=c(
  "vez", "sino", "cada", "ello", "así", "sólo", "digo", 
  "que", "que,", "señorías","gobierno",
  "españa","españoles","país","ser","hacia","años",
  "debe","cualquier","año","manera","todas","mayor",
  "parte","presidenta","ustedes","vista","señora","hecho","sus")
add.stops=c("kk")
stop_words = append(add.stops, tm::stopwords("spanish"))

discursos = discursos %>% 
  filter(!word %in% stop_words) 

discursos_tfidf = discursos %>% 
  count(docid, word, sort = TRUE) %>%
  ungroup() %>% 
  bind_tf_idf(word, docid, n) %>% 
  arrange(desc(tf_idf))

discursos_tfidf %>% arrange(desc(n))
```

    ## # A tibble: 26,342 x 6
    ##     docid      word     n          tf        idf       tf_idf
    ##     <chr>     <chr> <int>       <dbl>      <dbl>        <dbl>
    ##  1 11.txt  gobierno    82 0.016935151 0.00000000 0.0000000000
    ##  2 03.txt  política    69 0.012858740 0.00000000 0.0000000000
    ##  3 11.txt    españa    67 0.013837257 0.00000000 0.0000000000
    ##  4 08.txt  gobierno    65 0.016722408 0.00000000 0.0000000000
    ##  5 06.txt  gobierno    64 0.012398295 0.00000000 0.0000000000
    ##  6 11.txt  señorías    63 0.013011152 0.00000000 0.0000000000
    ##  7 11.txt españoles    59 0.012185048 0.08004271 0.0009753242
    ##  8 00.txt  gobierno    59 0.009525347 0.00000000 0.0000000000
    ##  9 09.txt    españa    56 0.011809363 0.00000000 0.0000000000
    ## 10 12.txt  gobierno    54 0.019128587 0.00000000 0.0000000000
    ## # ... with 26,332 more rows

``` r
discursos_tfidf %>% 
  group_by(docid) %>% 
  top_n(3) %>% 
  ungroup() %>% 
  arrange(docid) %>% 
  ggplot(aes(word,tf_idf, color=docid)) + 
    geom_label_repel(aes(label=word)) +
    theme(axis.text.x = element_text(angle = 90, hjust = 1)) +
    theme(legend.position="bottom")
```

![](tf_idf_files/figure-markdown_github-ascii_identifiers/plots%20words-1.png)

``` r
plots <- discursos_tfidf %>%
  group_by(docid) %>% 
  top_n(5) %>% 
  ungroup() %>% 
  mutate(word = reorder(word, tf_idf)) %>%
  split(.$docid) %>%
  map(~ ggplot(.x, aes(word,tf_idf)) + 
          geom_bar(stat="identity", aes(fill=docid)) + 
          theme(axis.text.x = element_text(angle = 45, hjust = 1)) +
          theme(legend.position="bottom")
  )

walk(plots, print)
```

![](tf_idf_files/figure-markdown_github-ascii_identifiers/td-idf%20words%20by%20doc-1.png)![](tf_idf_files/figure-markdown_github-ascii_identifiers/td-idf%20words%20by%20doc-2.png)![](tf_idf_files/figure-markdown_github-ascii_identifiers/td-idf%20words%20by%20doc-3.png)![](tf_idf_files/figure-markdown_github-ascii_identifiers/td-idf%20words%20by%20doc-4.png)![](tf_idf_files/figure-markdown_github-ascii_identifiers/td-idf%20words%20by%20doc-5.png)![](tf_idf_files/figure-markdown_github-ascii_identifiers/td-idf%20words%20by%20doc-6.png)![](tf_idf_files/figure-markdown_github-ascii_identifiers/td-idf%20words%20by%20doc-7.png)![](tf_idf_files/figure-markdown_github-ascii_identifiers/td-idf%20words%20by%20doc-8.png)![](tf_idf_files/figure-markdown_github-ascii_identifiers/td-idf%20words%20by%20doc-9.png)![](tf_idf_files/figure-markdown_github-ascii_identifiers/td-idf%20words%20by%20doc-10.png)![](tf_idf_files/figure-markdown_github-ascii_identifiers/td-idf%20words%20by%20doc-11.png)![](tf_idf_files/figure-markdown_github-ascii_identifiers/td-idf%20words%20by%20doc-12.png)![](tf_idf_files/figure-markdown_github-ascii_identifiers/td-idf%20words%20by%20doc-13.png)

``` r
discursos_bigram_raw = discursos_raw %>%     
  unnest_tokens(bigram, text, token = "ngrams", n = 2)

add.bigramsstops=c("señora presidenta", "sus señorías")
discursos_bigram_raw = discursos_bigram_raw %>% 
  filter(!bigram %in% add.bigramsstops)
  
discursos_bigram = discursos_bigram_raw %>% 
  separate(bigram, c("word1", "word2"), sep = " ") %>% 
  filter(!word1 %in% stop_words) %>%
  filter(!word2 %in% stop_words) %>% 
  unite(bigram, word1, word2, sep = " ") 

discursos_bigram_tfidf = discursos_bigram %>% 
  count(docid, bigram, sort = TRUE) %>%
  ungroup() %>% 
  bind_tf_idf(bigram, docid, n) %>% 
  arrange(desc(tf_idf))

discursos_bigram
```

    ## # A tibble: 21,985 x 2
    ##     docid              bigram
    ##  *  <chr>               <chr>
    ##  1 00.txt           x0 muchas
    ##  2 00.txt      muchas gracias
    ##  3 00.txt      gracias seã±or
    ##  4 00.txt   seã±or presidente
    ##  5 00.txt      venia seã±oras
    ##  6 00.txt  seã±ores diputados
    ##  7 00.txt                 s m
    ##  8 00.txt      rey comparezco
    ##  9 00.txt           acto cuya
    ## 10 00.txt cuya significaciã³n
    ## # ... with 21,975 more rows

``` r
discursos_bigram_tfidf %>% 
  group_by(docid) %>% 
  top_n(3) %>% 
  ungroup() %>% 
  mutate(bigram = reorder(bigram, tf_idf)) %>%
  ggplot(aes(bigram,tf_idf, color=docid)) + 
    geom_label_repel(aes(label=bigram)) +
    theme(axis.text.x = element_text(angle = 90, hjust = 1)) +
    theme(legend.position="bottom")
```

![](tf_idf_files/figure-markdown_github-ascii_identifiers/plots%20bigrams-1.png)

``` r
plots <- discursos_bigram_tfidf %>%
  group_by(docid) %>% 
  top_n(5) %>% 
  ungroup() %>% 
  mutate(bigram = reorder(bigram, tf_idf)) %>%
  split(.$docid) %>%
  map(~ ggplot(.x, aes(bigram,tf_idf)) + 
          geom_bar(stat="identity", aes(fill=docid)) + 
          theme(axis.text.x = element_text(angle = 45, hjust = 1)) +
          theme(legend.position="bottom")
  )

walk(plots, print)
```

![](tf_idf_files/figure-markdown_github-ascii_identifiers/td-idf%20bigrams%20by%20doc-1.png)![](tf_idf_files/figure-markdown_github-ascii_identifiers/td-idf%20bigrams%20by%20doc-2.png)![](tf_idf_files/figure-markdown_github-ascii_identifiers/td-idf%20bigrams%20by%20doc-3.png)![](tf_idf_files/figure-markdown_github-ascii_identifiers/td-idf%20bigrams%20by%20doc-4.png)![](tf_idf_files/figure-markdown_github-ascii_identifiers/td-idf%20bigrams%20by%20doc-5.png)![](tf_idf_files/figure-markdown_github-ascii_identifiers/td-idf%20bigrams%20by%20doc-6.png)![](tf_idf_files/figure-markdown_github-ascii_identifiers/td-idf%20bigrams%20by%20doc-7.png)![](tf_idf_files/figure-markdown_github-ascii_identifiers/td-idf%20bigrams%20by%20doc-8.png)![](tf_idf_files/figure-markdown_github-ascii_identifiers/td-idf%20bigrams%20by%20doc-9.png)![](tf_idf_files/figure-markdown_github-ascii_identifiers/td-idf%20bigrams%20by%20doc-10.png)![](tf_idf_files/figure-markdown_github-ascii_identifiers/td-idf%20bigrams%20by%20doc-11.png)![](tf_idf_files/figure-markdown_github-ascii_identifiers/td-idf%20bigrams%20by%20doc-12.png)![](tf_idf_files/figure-markdown_github-ascii_identifiers/td-idf%20bigrams%20by%20doc-13.png)
