<!--
%\VignetteEngine{knitr::knitr}
%\VignetteIndexEntry{aRxiv tutorial}
-->

# aRxiv tutorial

[arXiv](http://arxiv.org) is a repository of electronic preprints for
computer science, mathematics, physics, quantitative biology,
quantitative finance, and statistics. The
[aRxiv package](https://github.com/ropensci/aRxiv) provides an
[R](http://www.r-project.org) interface to the
[arXiv API](http://arxiv.org/help/api/index).

Note that the arXiv API _does not_ require an API key.





## Installation

You can install the [aRxiv package](https://github.com/rOpenSci/aRxiv)
via [CRAN](http://cran.r-project.org):


```r
install.packages("aRxiv")
```

Or use `devtools::install_github()` to get the (more recent) version
at [GitHub](https://github.com/rOpenSci/aRxiv):


```r
install.packages("devtools")
library(devtools)
install_github("ropensci/aRxiv")
```

## Basic use

Use `arxiv_search()` to search [arXiv](http://arxiv.org),
`arxiv_count()` to get a simple count of manuscripts matching a
query, and `arxiv_open()` to open the abstract pages for a set of
results from `arxiv_search()`.

We'll get to the details in a moment. For now, let's look at a few
examples.

Suppose we wanted to identify all arXiv manuscripts with &ldquo;`Peter
Hall`&rdquo; as an author. It is best to first get a count, so that we have
a sense of how many records the search will return. (Peter Hall is
&ldquo;[among the world's most prolific and highly cited authors in both probability and statistics](http://en.wikipedia.org/wiki/Peter_Gavin_Hall).&rdquo;)
We first use `library()` to load the aRxiv package and then
`arxiv_count()` to get the count.


```r
library(aRxiv)
arxiv_count('au:"Peter Hall"')
```

```
## [1] 51
```

The `au:` part indicates to search the author field; we use double
quotes to search for a _phrase_.

To obtain the actual records matching the query, use `arxiv_search()`.


```r
rec <- arxiv_search('au:"Peter Hall"')
nrow(rec)
```

```
## [1] 10
```

The default is to grab no more than 10 records; this limit can be
changed with the `limit` argument. But note that the arXiv API will
not let you download more than 50,000 or so records, and even in that
case it's best to do so in batches; more on this below.

Also note that the result of `arxiv_search()` has an attribute
`"total_results"` containing the total count of search results; this
is the same as what `arxiv_count()` provides.


```r
attr(rec, "total_results")
```

```
## [1] 51
```

The following will get us all 51
records.


```r
rec <- arxiv_search('au:"Peter Hall"', limit=50)
nrow(rec)
```

```
## [1] 50
```

`arxiv_search()` returns a data frame with each row being a single
manuscript. The columns are the different fields (e.g., `authors`, `title`,
`abstract`, etc.). Fields like `authors` that
contain multiple items will be a single character string with the
multiple items separated by a vertical bar (`|`).

We might be interested in a more restrictive search, such as for Peter
Hall's arXiv manuscripts that have `deconvolution` in the title. We
use `ti:` to search the title field, and combine the two with `AND`.


```r
deconv <- arxiv_search('au:"Peter Hall" AND ti:deconvolution')
nrow(deconv)
```

```
## [1] 4
```

Let's display just the authors and title for the results.


```r
deconv[, c('title', 'authors')]
```

```
##                                                                             title
## 1                                     A ridge-parameter approach to deconvolution
## 2                                     On deconvolution with repeated measurements
## 3 Estimation of distributions, moments and quantiles in deconvolution\n  problems
## 4   Kernel methods and minimum contrast estimators for empirical\n  deconvolution
##                                        authors
## 1                 Peter Hall|Alexander Meister
## 2 Aurore Delaigle|Peter Hall|Alexander Meister
## 3               Peter Hall|Soumendra N. Lahiri
## 4                   Aurore Delaigle|Peter Hall
```

We can open the abstract pages for these 4 manuscripts
using `arxiv_open()`. It takes, as input, the output of
`arxiv_search()`.


```r
arxiv_open(deconv)
```


## Forming queries

The two basic arguments to `arxiv_count()` and `arxiv_search()` are
`query`, a
character string representing the search, and `id_list`, a list of
[arXiv manuscript identifiers](http://arxiv.org/help/arxiv_identifier).

- If only `query` is provided, manuscripts matching that query are
  returned.
- If only `id_list` is provided, manuscripts in the list are
  returned.
- If both are provided, manuscripts in `id_list` that match `query`
  will be returned.

`query` may be a single character string or a vector of character
strings. If it is a vector, the elements are pasted together with
`AND`.

`id_list` may be a vector of character strings or a single
comma-separated character string.

### Search terms

Generally, one would ignore `id_list` and focus on forming the `query`
argument. The aRxiv package includes a dataset `query_terms` that
lists the terms (like `au`) that you can use.


```r
query_terms
```

```
##               term                                      description
## 1               ti                                            Title
## 2               au                                           Author
## 3              abs                                         Abstract
## 4               co                                          Comment
## 5               jr                                Journal Reference
## 6              cat                                 Subject Category
## 7               rn                                    Report Number
## 8              all                                 All of the above
## 9    submittedDate Date/time of initial submission, as YYYYMMDDHHMM
## 10 lastUpdatedDate        Date/time of last update, as YYYYMMDDHHMM
```

Use a colon (`:`) to separate the query term from the actual query.
Multiple queries can be combined with `AND`, `OR`, and `ANDNOT`. The
default is `OR`.


```r
arxiv_count('au:Peter au:Hall')
```

```
## [1] 14860
```

```r
arxiv_count('au:Peter OR au:Hall')
```

```
## [1] 14860
```

```r
arxiv_count('au:Peter AND au:Hall')
```

```
## [1] 72
```

```r
arxiv_count('au:Hall ANDNOT au:Peter')
```

```
## [1] 1292
```

It appears that in the author field (and many other fields) you must
search full words, and that wild cards not allowed.


```r
arxiv_count('au:P* AND au:Hall')
```

```
## [1] 0
```

```r
arxiv_count('au:P AND au:Hall')
```

```
## [1] 567
```

```r
arxiv_count('au:"P Hall"')
```

```
## [1] 37
```

### Subject classifications

arXiv has a set of 127 subject classifications,
searchable with the prefix `cat:`. The aRxiv package contains a
dataset `arxiv_cats` containing the abbreviations and descriptions.
Here are the statistics categories.


```r
arxiv_cats[grep('^stat', arxiv_cats$abbreviation),]
```

```
##   abbreviation                   description
## 1      stat.AP     Statistics - Applications
## 2      stat.CO      Statistics - Computation
## 3      stat.ML Statistics - Machine Learning
## 4      stat.ME      Statistics - Methodology
## 5      stat.TH           Statistics - Theory
```

To search these categories, you need to include either the full term
or use the `*` wildcard.


```r
arxiv_count('cat:stat')
```

```
## [1] 0
```

```r
arxiv_count('cat:stat.AP')
```

```
## [1] 3360
```

```r
arxiv_count('cat:stat*')
```

```
## [1] 17853
```

### Dates and ranges of dates

The terms `submittedDate` (date/time of first submission) and
`lastUpdatedDate` (date/time of last revision) are particularly
useful for limiting a search with _many_ results, so that you may
combine multiple searches together, each within some window of time,
to get the full results.

The date/time information is of the form `YYYYMMDDHHMMSS`, for
example `20071018122534` for `2007-10-18 12:25:34`. You can use `*`
for a wildcard for the times. For example, to get all manuscripts
with initial submission on 2007-10-18:


```r
arxiv_count('submittedDate:20071018*')
```

```
## [1] 196
```

But you can't use the wildcard within the _dates_.


```r
arxiv_count('submittedDate:2007*')
```

```
## [1] 0
```

To get a count of all manuscripts with original submission in 2007,
use a date range, like `[from_date TO to_date]`. (If you give a partial
date, it's treated as the earliest date/time that matches, and the
range appears to be up to but not including the second date/time.)


```r
arxiv_count('submittedDate:[2007 TO 2008]')
```

```
## [1] 55749
```

## Search results

The output of `arxiv_search()` is a data frame with the following
columns.


```r
res <- arxiv_search('au:"Terry Speed"')
names(res)
```

```
##  [1] "id"               "submitted"        "updated"         
##  [4] "title"            "abstract"         "authors"         
##  [7] "affiliations"     "link_abstract"    "link_pdf"        
## [10] "link_doi"         "comment"          "journal_ref"     
## [13] "doi"              "primary_category" "categories"
```

The columns are described in the help file for `arxiv_search()`. Try
`?arxiv_search`.

A few short notes:

- Each field is a single character string. `authors`, `link_doi`, and
  `categories` may contain multiple items, separated by a vertical bar
  (`|`).
- Missing entries will have an empty character string (`""`).
- The `categories` column may contain not just the aRxiv categories
  (e.g., `stat.AP`) but also codes for the
  [Mathematical Subject Classification (MSC)](http://www.ams.org/mathscinet/msc/msc2010.html) (e.g., 14J60)
  and the
  [ACM Computing Classification System](http://www.acm.org/about/class/1998/)
  (e.g., F.2.2). These are not searchable with `cat:` but are
  searchable with a general search.


```r
arxiv_count("cat:14J60")
```

```
## [1] 0
```

```r
arxiv_count("14J60")
```

```
## [1] 367
```


## Sorting results

The `arxiv_search()` function has two arguments for sorting the results,
`sort_by` (taking values `"submitted"`, `"updated"`, or
`"relevance"`) and `ascending` (`TRUE` or `FALSE`). If `id_list` is
provided, these sorting arguments are ignored and the results are
presented according to the order in `id_list`.

Here's an example, to sort the results by the date the manuscripts
were last updated, in descending order.


```r
res <- arxiv_search('au:"Terry Speed"', sort_by="updated",
                    ascending=FALSE)
res$updated
```

```
## [1] "2012-01-31 05:54:46" "2008-06-27 08:25:01"
```


## Technical details

### Metadata limitations

The [arXiv metadata](http://arxiv.org/help/prep) has a number of
limitations, the key issue being that it is author-supplied and so not
necessarily consistent between records.

Authors' names may vary between records (e.g., T. P. Speed vs. Terry
Speed vs. Terence P. Speed). Further, arXiv provides no ability to
distinguish multiple individuals with the same name (c.f.,
[ORCID](http://orcid.org)).

Authors' institutional affiliations are mostly missing.  The arXiv
submission form does not include an affiliation field; affiliations
are entered within the author field, in parentheses.  The
[metadata instructions](http://arxiv.org/help/prep#author) may not be
widely read.

There are no key words; you are stuck with searching the free text in
the titles and abstracts.

Subject classifications are provided by the authors and may be
incomplete or inappropriate.


### Limit time between search requests

Care should be taken to avoid multiple requests to the arXiv API in a
short period of time. The
[arXiv API user manual](http://arxiv.org/help/api/user-manual) states:

> In cases where the API needs to be called multiple times in a row,
> we encourage you to play nice and incorporate a 3 second delay in
> your code.

The aRxiv package institutes a delay between requests, with the time
period for the delay configurable with the R option
`"aRxiv_delay"` (in seconds). The default is 3 seconds.

To reduce the delay to 1 second, use:


```r
options(aRxiv_delay=1)
```

**Don't** do searches in parallel (e.g., via the parallel
package). You may be locked out from the arXiv API.


### Limit number of items returned

The arXiv API returns only complete records (including the entire
abstracts); searches returning large numbers of records can be very
slow.

It's best to use `arxiv_count()` before `arxiv_search()`, so that you have
a sense of how many records you will receive. If the count is large,
you may wish to refine your query.

arXiv has a hard limit of around 50,000 records; for a query that
matches more than 50,000 manuscripts, there is no way to receive the
full results. The simplest solution to this problem is to break the
query into smaller pieces, for example using slices of time, with a
range of dates for `submittedDate` or `lastUpdatedDate`.

The `limit` argument to `arxiv_search()` (with default `limit=10`)
limits the number of records to be returned. If you wish to receive
more than 10 records, you must specify a larger limit (e.g., `limit=100`).

To avoid accidental searches that may return a very large number of
records, `arxiv_search()` uses an R option, `aRxiv_toomany` (with a
default of 15,000), and refuses to attempt a search that will return
results above that limit.

### Make requests in batches

Even for searches that return a moderate number of records (say
2,000), it may be best to make the requests in batches: Use a smaller
value for the `limit` argument (say 100), and make multiple requests
with different offsets, indicated with the `start` argument, for the
initial record to return.

This is done automatically with the `batchsize` argument to
`arxiv_search()`.  A search is split into multiple calls, with no more
than `batchsize` records to be returned by each, and then the results
are combined.


## License and bugs

- License:
  [MIT](https://github.com/ropensci/aRxiv/blob/master/LICENSE)
- Report bugs or suggestions improvements by [submitting an issue](https://github.com/ropensci/aRxiv/issues) to
  [our GitHub repository for aRxiv](https://github.com/ropensci/aRxiv).



<!-- the following to make it look nicer -->
<link href="http://kbroman.org/qtlcharts/assets/vignettes/vignette.css" rel="stylesheet"></link>