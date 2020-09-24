Lab 06 - Text Mining
================

# Learning goals

  - Use `unnest_tokens()` and `unnest_ngrams()` to extract tokens and
    ngrams from text.
  - Use dplyr and ggplot2 to analyze text data

# Lab description

For this lab we will be working with a new dataset. The dataset contains
transcription samples from <https://www.mtsamples.com/>. And is loaded
and “fairly” cleaned at
<https://raw.githubusercontent.com/USCbiostats/data-science-data/master/00_mtsamples/mtsamples.csv>.

This markdown document should be rendered using `github_document`
document.

# Setup the Git project and the GitHub repository

1.  Go to your documents (or wherever you are planning to store the
    data) in your computer, and create a folder for this project, for
    example, “PM566-labs”

2.  In that folder, save [this
    template](https://raw.githubusercontent.com/USCbiostats/PM566/master/content/assignment/06-lab.Rmd)
    as “README.Rmd”. This will be the markdown file where all the magic
    will happen.

3.  Go to your GitHub account and create a new repository, hopefully of
    the same name that this folder has, i.e., “PM566-labs”.

4.  Initialize the Git project, add the “README.Rmd” file, and make your
    first commit.

5.  Add the repo you just created on GitHub.com to the list of remotes,
    and push your commit to origin while setting the upstream.

### Setup packages

You should load in `dplyr`, (or `data.table` if you want to work that
way), `ggplot2` and `tidytext`. If you don’t already have `tidytext`
then you can install with

``` r
library(tidytext)
library(tidyverse)
```

    ## -- Attaching packages ------------------------------------------------------------------------------------------------ tidyverse 1.3.0 --

    ## √ ggplot2 3.3.2     √ purrr   0.3.4
    ## √ tibble  3.0.3     √ dplyr   1.0.1
    ## √ tidyr   1.1.1     √ stringr 1.4.0
    ## √ readr   1.3.1     √ forcats 0.5.0

    ## -- Conflicts --------------------------------------------------------------------------------------------------- tidyverse_conflicts() --
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

``` r
library(ggplot2)
library(readr)
library(dplyr)
```

### read in Medical Transcriptions

Loading in reference transcription samples from
<https://www.mtsamples.com/>

``` r
mt_samples <- read_csv("https://raw.githubusercontent.com/USCbiostats/data-science-data/master/00_mtsamples/mtsamples.csv")
mt_samples <- mt_samples %>%
  select(description, medical_specialty, transcription)

head(mt_samples)
```

    ## # A tibble: 6 x 3
    ##   description                  medical_specialty   transcription                
    ##   <chr>                        <chr>               <chr>                        
    ## 1 A 23-year-old white female ~ Allergy / Immunolo~ "SUBJECTIVE:,  This 23-year-~
    ## 2 Consult for laparoscopic ga~ Bariatrics          "PAST MEDICAL HISTORY:, He h~
    ## 3 Consult for laparoscopic ga~ Bariatrics          "HISTORY OF PRESENT ILLNESS:~
    ## 4 2-D M-Mode. Doppler.         Cardiovascular / P~ "2-D M-MODE: , ,1.  Left atr~
    ## 5 2-D Echocardiogram           Cardiovascular / P~ "1.  The left ventricular ca~
    ## 6 Morbid obesity.  Laparoscop~ Bariatrics          "PREOPERATIVE DIAGNOSIS: , M~

``` r
mt_samples$transcription[1]
```

    ## [1] "SUBJECTIVE:,  This 23-year-old white female presents with complaint of allergies.  She used to have allergies when she lived in Seattle but she thinks they are worse here.  In the past, she has tried Claritin, and Zyrtec.  Both worked for short time but then seemed to lose effectiveness.  She has used Allegra also.  She used that last summer and she began using it again two weeks ago.  It does not appear to be working very well.  She has used over-the-counter sprays but no prescription nasal sprays.  She does have asthma but doest not require daily medication for this and does not think it is flaring up.,MEDICATIONS: , Her only medication currently is Ortho Tri-Cyclen and the Allegra.,ALLERGIES: , She has no known medicine allergies.,OBJECTIVE:,Vitals:  Weight was 130 pounds and blood pressure 124/78.,HEENT:  Her throat was mildly erythematous without exudate.  Nasal mucosa was erythematous and swollen.  Only clear drainage was seen.  TMs were clear.,Neck:  Supple without adenopathy.,Lungs:  Clear.,ASSESSMENT:,  Allergic rhinitis.,PLAN:,1.  She will try Zyrtec instead of Allegra again.  Another option will be to use loratadine.  She does not think she has prescription coverage so that might be cheaper.,2.  Samples of Nasonex two sprays in each nostril given for three weeks.  A prescription was written as well."

-----

## Question 1: What specialties do we have?

We can use `count()` from `dplyr` to figure out how many different
catagories do we have? Are these catagories related? overlapping? evenly
distributed?

``` r
mt_samples %>%
  count(medical_specialty, sort = TRUE)
```

    ## # A tibble: 40 x 2
    ##    medical_specialty                 n
    ##    <chr>                         <int>
    ##  1 Surgery                        1103
    ##  2 Consult - History and Phy.      516
    ##  3 Cardiovascular / Pulmonary      372
    ##  4 Orthopedic                      355
    ##  5 Radiology                       273
    ##  6 General Medicine                259
    ##  7 Gastroenterology                230
    ##  8 Neurology                       223
    ##  9 SOAP / Chart / Progress Notes   166
    ## 10 Obstetrics / Gynecology         160
    ## # ... with 30 more rows

Here, these categories may overlapping.

-----

## Question 2

  - Tokenize the the words in the `transcription` column
  - Count the number of times each token appears
  - Visualize the top 20 most frequent words

Explain what we see from this result. Does it makes sense? What insights
(if any) do we get?

``` r
mt_samples %>%
  unnest_tokens(output=token, input=transcription) %>%
  count(token, sort = T) %>%
  top_n(n=50, wt=n) %>%
  ggplot(aes(x=n,y=fct_reorder(token,n))) + 
  geom_col()
```

![](lab6_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

-----

## Question 3

  - Redo visualization but remove stopwords before
  - Bonus points if you remove numbers as well

<!-- end list -->

``` r
mt_samples %>%
  unnest_tokens(output=token, input=transcription) %>%
  anti_join(stop_words, by= c("token"="word")) %>%
  filter(! (token %in% as.character(seq(0,100, by=1)))) %>%
  count(token, sort = T) %>%
  top_n(n=50, wt=n) %>%
  ggplot(aes(x=n,y=fct_reorder(token,n))) + 
  geom_col()
```

![](lab6_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

What do we see know that we have removed stop words? Does it give us a
better idea of what the text is about?

-----

# Question 4

repeat question 2, but this time tokenize into bi-grams. how does the
result change if you look at tri-grams?

``` r
mt_samples %>%
  unnest_ngrams(output=token, input=transcription, n=2) %>%
  count(token, sort = T) %>%
  top_n(n=50, wt=n) %>%
  ggplot(aes(x=n,y=fct_reorder(token,n))) + 
  geom_col()
```

![](lab6_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

``` r
mt_samples %>%
  unnest_ngrams(output=token, input=transcription, n=3) %>%
  count(token, sort = T) %>%
  top_n(n=50, wt=n) %>%
  ggplot(aes(x=n,y=fct_reorder(token,n))) + 
  geom_col()
```

![](lab6_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

-----

# Question 5

Using the results you got from questions 4. Pick a word and count the
words that appears after and before it.

``` r
mt_bi = mt_samples %>%
  unnest_ngrams(output=token, input=transcription, n=2) %>%
  
  separate(token, into =c("word1","word2"),sep=" " ) %>%
  select(word1, word2)

mt_bi %>%
  filter(word1=="patient") %>%
  count(word2, sort = T)
```

    ## # A tibble: 588 x 2
    ##    word2         n
    ##    <chr>     <int>
    ##  1 was        6291
    ##  2 is         3330
    ##  3 has        1417
    ##  4 tolerated   992
    ##  5 had         888
    ##  6 will        616
    ##  7 denies      552
    ##  8 and         377
    ##  9 states      363
    ## 10 does        334
    ## # ... with 578 more rows

``` r
mt_bi %>%
  filter(word2=="patient") %>%
  count(word1, sort = T)
```

    ## # A tibble: 269 x 2
    ##    word1         n
    ##    <chr>     <int>
    ##  1 the       20301
    ##  2 this        470
    ##  3 history     101
    ##  4 a            67
    ##  5 and          47
    ##  6 procedure    32
    ##  7 female       26
    ##  8 with         25
    ##  9 use          24
    ## 10 old          23
    ## # ... with 259 more rows

``` r
mt_bi %>%
  count(word1,word2, sort=T)
```

    ## # A tibble: 301,399 x 3
    ##    word1   word2       n
    ##    <chr>   <chr>   <int>
    ##  1 the     patient 20301
    ##  2 of      the     19050
    ##  3 in      the     12784
    ##  4 to      the     12372
    ##  5 was     then     6952
    ##  6 and     the      6346
    ##  7 patient was      6291
    ##  8 the     right    5509
    ##  9 on      the      5241
    ## 10 the     left     4858
    ## # ... with 301,389 more rows

-----

# Question 6

Which words are most used in each of the specialties. you can use
`group_by()` and `top_n()` from `dplyr` to have the calculations be done
within each specialty. Remember to remove stopwords. How about the most
5 used words?

``` r
mt_samples %>%
  unnest_tokens(token, transcription) %>%
  anti_join(tidytext::stop_words, by=c("token"="word")) %>%
  group_by(medical_specialty) %>%
  count(token, sort=T) %>%
  top_n(5,n)
```

    ## # A tibble: 209 x 3
    ## # Groups:   medical_specialty [40]
    ##    medical_specialty          token         n
    ##    <chr>                      <chr>     <int>
    ##  1 Surgery                    patient    4855
    ##  2 Surgery                    left       3263
    ##  3 Surgery                    procedure  3243
    ##  4 Consult - History and Phy. patient    3046
    ##  5 Consult - History and Phy. history    2820
    ##  6 Surgery                    0          2094
    ##  7 Surgery                    1          2038
    ##  8 Orthopedic                 patient    1711
    ##  9 Cardiovascular / Pulmonary left       1550
    ## 10 Cardiovascular / Pulmonary patient    1516
    ## # ... with 199 more rows

# Question 7 - extra

Find your own insight in the data:

Ideas:

  - Interesting ngrams
  - See if certain words are used more in some specialties then others

<!-- end list -->

``` r
mt_samples %>%
  unnest_ngrams(output=token, input=transcription, n=3) %>%
  group_by(medical_specialty) %>%
  count(token, sort=T) %>%
  top_n(1,n)
```

    ## # A tibble: 44 x 3
    ## # Groups:   medical_specialty [40]
    ##    medical_specialty          token               n
    ##    <chr>                      <chr>           <int>
    ##  1 Surgery                    the patient was  2139
    ##  2 Consult - History and Phy. the patient is    638
    ##  3 Orthopedic                 the patient was   572
    ##  4 Cardiovascular / Pulmonary the patient was   422
    ##  5 Urology                    the patient was   280
    ##  6 General Medicine           the patient is    275
    ##  7 Obstetrics / Gynecology    the patient was   245
    ##  8 Gastroenterology           the patient was   231
    ##  9 Discharge Summary          the patient was   196
    ## 10 Radiology                  there is no       188
    ## # ... with 34 more rows

The most common phrases for specialties like surgery and consult are
“the patient was” and “the patient is”. The office note specialty has
“there is no” as the most commom phrase.
