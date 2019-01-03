
<!-- README.md is generated from README.Rmd. Please edit that file -->
lgr
===

[![Lifecycle: maturing](https://img.shields.io/badge/lifecycle-maturing-blue.svg)](https://www.tidyverse.org/lifecycle/#maturing) [![Travis build status](https://travis-ci.org/s-fleck/lgr.svg?branch=master)](https://travis-ci.org/s-fleck/lgr) [![Codecov test coverage](https://codecov.io/gh/s-fleck/lgr/branch/master/graph/badge.svg)](https://codecov.io/gh/s-fleck/lgr?branch=master)

lgr is a logging package for R built on the back of [R6](https://github.com/r-lib/R6) classes. It is designed to be flexible, performant and extensible.

Users that have not worked with R6 classes before, will find the syntax for configuring Loggers a bit strange, but I did my best to compose a hopefully helpful [vignette](https://s-fleck.github.io/lgr/articles/lgr.html). Users that come from python or Java, will feel at home as lgr borrows heavily from [Apache Log4j](https://logging.apache.org/log4j/2.x/) and [Python logging](https://docs.python.org/3/library/logging.html).

Features
--------

-   *Hierarchical loggers* like in log4j and python logging. This is useful if you want to be able to configure logging on a per-package basis.
-   An *arbitrary number of appenders* for each logger. A single logger can write values to the console, a logfile, a database, etc... .
-   Allow for *custom fields* in log events. As opposed to many other logging packages for R a log event is not just a message with a timestamp, but can contain arbitrary data fields. This is very helpful if you want to produce logs that are machine readable and easy to analyze.
-   *Vectorized* logging (so `lgr$fatal(capture.output(iris))` works)
-   Lightning fast in-memory log based in `data.table` included for interactive use.
-   Support for advanced appenders such as database appenders, cached appenders, etc... .
-   Optional *color* support via [crayon](https://github.com/r-lib/crayon)

Usage
-----

To log an *event* with with lgr we call `lgr$<logging function>()`. Unnamed arguments to the logging function are interpreted by `sprintf()`.

``` r
lgr$fatal("A critical error")
#> FATAL [16:08:51.839] A critical error
lgr$error("A less severe error")
#> ERROR [16:08:51.868] A less severe error
lgr$warn("A potentially bad situation")
#> WARN  [16:08:51.897] A potentially bad situation
lgr$info("iris has %s rows", nrow(iris))
#> INFO  [16:08:51.899] iris has 150 rows

# the following log levels are hidden by default
lgr$debug("A debug message")
lgr$trace("A finer grained debug message")
```

A Logger can have several Appenders. For example, we can add a JSON appender to log to a file with little effort.

``` r
tf <- tempfile()
lgr$add_appender(AppenderJson$new(tf))
lgr$info("cars has %s rows", nrow(cars))
#> INFO  [16:08:51.927] cars has 50 rows

cat(readLines(tf))
#> {"level":400,"timestamp":"2019-01-03 16:08:51","caller":"eval","msg":"cars has 50 rows"}
```

JSON naturally supports custom fields. Named arguments passed to `info()`, `warn()`, etc... are intepreted as custom fields.

``` r
lgr$info("loading cars", "cars", rows = nrow(cars), cols = ncol(cars))
#> INFO  [16:08:51.937] loading cars {cols: 2, rows: 50}

cat(readLines(tf), sep = "\n")
#> {"level":400,"timestamp":"2019-01-03 16:08:51","caller":"eval","msg":"cars has 50 rows"}
#> {"level":400,"timestamp":"2019-01-03 16:08:51","caller":"eval","msg":"loading cars","cols":2,"rows":50}
```

For more examples please see the package [vignette](https://s-fleck.github.io/lgr/articles/lgr.html) and [documentation](https://s-fleck.github.io/lgr/)

Development Status
------------------

lgr is stable, but I want to conduct a some field-tests to iron out the last few bugs before I announce it. A CRAN release is planned for February 2019.

Dependencies
------------

[R6](https://github.com/r-lib/R6): The R6 class system prevents the framework on which lgr is built and the **only Package lgr will ever depend on**.

### Optional Dependencies

These optional dependencies that are not necessary to use lgr, but that are required for some extra appenders. Care was taken to choose packages that are slim, stable, have minimal dependencies, and are well mentained:

-   [crayon](https://github.com/r-lib/crayon) for colored console output.
-   [data.table](https://github.com/Rdatatable/) for fast in-memroy logging with `AppenderDt`.
-   [jsonlite](https://github.com/jeroen/jsonlite) for JSON logging via `LayoutJson`. JSON is a populat plaintext based file format that is easy to read for humans and machines alike.
-   [DBI](https://github.com/r-dbi/DBI) for logging to databases. Logging with lgr has been tested with the following backends:
    -   [RSQLite](https://github.com/r-dbi/RSQLite),
    -   [RMariaDB](https://cran.r-project.org/web/packages/RMySQL/index.html) for MariaDB and MySQL,
    -   [RPostgreSQL](https://cran.r-project.org/web/packages/RPostgreSQL/index.html),
    -   [RJDBC](https://github.com/s-u/RJDBC) for DB2.

    In theory all DBI compliant database packages should work. If you are using lgr with a database backend, please report your (positive and negative) experiences to me.
-   [whoami](https://github.com/r-lib/whoami/blob/master/DESCRIPTION) for guessing the user name from various sources. You can also set the user name manually if you want to use it for logging.

Installation
------------

``` r
devtools::install.github("s-fleck/lgr")
```

Outlook
-------

The long term goal is to support (nearly) all features of the python logging module. If you have experience with python logging or Log4j and are missing features/appenders that you'd like to see, please feel free to post a feature request on the issue tracker.

Acknowledgements
----------------

-   [Inkscape](https://inkscape.org/) for the hex sticker
-   [draw.io](https://draw.io/) for the flow chart in the vignette
