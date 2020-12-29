


[![Linux Build Status](https://travis-ci.org/r-lib/prettyunits.svg?branch=master)](https://travis-ci.org/r-lib/prettyunits)
[![Windows Build status](https://ci.appveyor.com/api/projects/status/github/r-lib/prettyunits?svg=true)](https://ci.appveyor.com/project/gaborcsardi/prettyunits)
[![CRAN RStudio mirror downloads](http://cranlogs.r-pkg.org/badges/prettyunits)](https://CRAN.R-project.org/package=prettyunits)

# prettyunits

The `prettyunits` package formats quantities in human readable form. Currently
- time units 
- information (i.e. bytes) 
- linear quantities (i.e. like quantities representing distance, but not surface or volume)
are supported.

## Installation

You can install the package from CRAN:


```r
install.packages("prettyunits")
```


```r
library(prettyunits)
library(magrittr)
```

## Bytes

`pretty_bytes` formats number of bytes in a human readable way:


```r
pretty_bytes(1337)
```

```
##> [1] "1.34 kB"
```

```r
pretty_bytes(133337)
```

```
##> [1] "133.34 kB"
```

```r
pretty_bytes(13333337)
```

```
##> [1] "13.33 MB"
```

```r
pretty_bytes(1333333337)
```

```
##> [1] "1.33 GB"
```

```r
pretty_bytes(133333333337)
```

```
##> [1] "133.33 GB"
```

Here is a simple function that emulates the Unix `ls` command, with
nicely formatted file sizes:


```r
uls <- function(path = ".") {
  files <- dir(path)
  info <- files %>%
    lapply(file.info) %>%
    do.call(what = rbind)
  info$size <- pretty_bytes(info$size)
  df <- data.frame(d = ifelse(info$isdir, "d", " "),
	mode = as.character(info$mode), user = info$uname, group = info$grname,
    size = ifelse(info$isdir, "", info$size), modified = info$mtime, name = files)
  print(df, row.names = FALSE)
}
uls()
```

```
##>  d mode        user group     size            modified       name
##>     644 gaborcsardi staff 440.00 B 2019-03-25 10:25:08    NEWS.md
##>     644 gaborcsardi staff  4.65 kB 2017-12-15 11:00:16  README.md
##>     644 gaborcsardi staff  2.95 kB 2019-03-27 09:58:43 README.Rmd
```

## Quantities

`pretty_num` formats number related to linear quantities in a human readable way:

```r
pretty_num(1337)
```

```
##> [1] "1.34 k"
```

```r
pretty_num(-133337)
```

```
##> [1] "-133.34 k"
```

```r
pretty_num(1333.37e-9)
```

```
##> [1] "1.33 µ"
```
Be aware that the result is wrong in case of surface or volumes, and for any non-linear quantity.

Here is a simple example of how to prettify a entire tibble

```r
library(tidyverse)
tdf <- tribble( ~name, ~`size in m`, ~`speed in m/s`,
                "land snail", 0.075, 0.001,
                "photon", NA,  299792458,
                "African plate", 10546330, 0.000000000681)
tdf %>% mutate(across(where(is.numeric), pretty_num))
```

```
##> # A tibble: 3 x 3
##>   name          `size in m` `speed in m/s`
##>   <chr>         <chr>       <chr>         
##> 1 land snail    "   75 m"   "     1 m"    
##> 2 photon        "    NA "   "299.79 M"    
##> 3 African plate "10.55 M"   "   681 p"
```
You may want to use non-breakable space so that the unit prefix is never separated from the number in space constrained situation like text hover or text labels:

```r
pretty_num(1333.37e-9 , sep = "\xC2\xA0")
```

```
##> [1] "1.33 µ"
```


### Quantitiies of class `units`

`pretty_num` loosely preserves units associated with a quantity: 

```r
library(units)
```

```
##> Warning: package 'units' was built under R version 3.6.2
```

```
##> udunits system database from /Library/Frameworks/R.framework/Versions/3.6/Resources/library/units/share/udunits
```

```r
l_cm <- set_units(1337129, cm)
pretty_num(l_cm)
```

```
##> [1] "1.34 M [cm]"
```
So it is up to you to turn the unit into the right [base-unit](https://en.wikipedia.org/wiki/SI_base_unit)

If you do so, then the best prefix is potentially moved to the units : 

```r
pretty_num(l_cm %>% set_units(m))
```

```
##> [1] "13.37 [km]"
```

If you try non-linear units, you should get an error:
```
surface <- set_units(1337129, "m2")
pretty_num(surface)
```
```
##> Error in compute_num(number) : pretty_num() doesn't handle non-linear units
```

This can be used for an entire data-frame as well

```r
names(tdf) <- c( "name", "size", "speed" )
units(tdf$size) <- "m"
units(tdf$speed) <- "m/s"
tdf %>% mutate(across(where(is.numeric), pretty_num))
```

```
##> # A tibble: 3 x 3
##>   name          size          speed           
##>   <chr>         <chr>         <chr>           
##> 1 land snail    "   75 m [m]" "     1 m [m/s]"
##> 2 photon        "    NA  [m]" "299.79 M [m/s]"
##> 3 African plate "10.55 M [m]" "   681 p [m/s]"
```

## Time intervals

`pretty_ms` formats a time interval given in milliseconds. `pretty_sec` does
the same for seconds, and `pretty_dt` for `difftime` objects. The optional
`compact` argument turns on a compact, approximate format.


```r
pretty_ms(c(1337, 13370, 133700, 1337000, 1337000000))
```

```
##> [1] "1.3s"            "13.4s"           "2m 13.7s"        "22m 17s"         "15d 11h 23m 20s"
```

```r
pretty_ms(c(1337, 13370, 133700, 1337000, 1337000000),
  compact = TRUE)
```

```
##> [1] "~1.3s"  "~13.4s" "~2m"    "~22m"   "~15d"
```

```r
pretty_sec(c(1337, 13370, 133700, 1337000, 13370000))
```

```
##> [1] "22m 17s"          "3h 42m 50s"       "1d 13h 8m 20s"    "15d 11h 23m 20s"  "154d 17h 53m 20s"
```

```r
pretty_sec(c(1337, 13370, 133700, 1337000, 13370000),
  compact = TRUE)
```

```
##> [1] "~22m"  "~3h"   "~1d"   "~15d"  "~154d"
```

## Vague time intervals

`vague_dt` and `time_ago` formats time intervals using a vague format,
omitting smaller units. They both have three formats: `default`, `short` and `terse`.
`vague_dt` takes a `difftime` object, and `time_ago` works relatively to the
specified date.


```r
vague_dt(format = "short", as.difftime(30, units = "secs"))
```

```
##> [1] "<1 min"
```

```r
vague_dt(format = "short", as.difftime(14, units = "mins"))
```

```
##> [1] "14 min"
```

```r
vague_dt(format = "short", as.difftime(5, units = "hours"))
```

```
##> [1] "5 hours"
```

```r
vague_dt(format = "short", as.difftime(25, units = "hours"))
```

```
##> [1] "1 day"
```

```r
vague_dt(format = "short", as.difftime(5, units = "days"))
```

```
##> [1] "5 day"
```


```r
now <- Sys.time()
time_ago(now)
```

```
##> [1] "moments ago"
```

```r
time_ago(now - as.difftime(30, units = "secs"))
```

```
##> [1] "less than a minute ago"
```

```r
time_ago(now - as.difftime(14, units = "mins"))
```

```
##> [1] "14 minutes ago"
```

```r
time_ago(now - as.difftime(5, units = "hours"))
```

```
##> [1] "5 hours ago"
```

```r
time_ago(now - as.difftime(25, units = "hours"))
```

```
##> [1] "a day ago"
```


