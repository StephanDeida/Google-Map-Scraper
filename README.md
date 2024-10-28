# Google maps scraper

> A command line and web UI google maps scraper

## Features

- Extracts many data points from google maps
- Exports the data to CSV, JSON or PostgreSQL
- Perfomance about 120 urls per minute (-depth 1 -c 8)
- Extendable to write your own exporter
- Dockerized for easy run in multiple platforms
- Scalable in multiple machines
- Optionally extracts emails from the website of the business
- SOCKS5 proxy support

## Notes on email extraction

By defaul email extraction is disabled.

If you enable email extraction (see quickstart) then the scraper will visit the
website of the business (if exists) and it will try to extract the emails from the
page.

Keep in mind that enabling email extraction results to larger processing time, since more
pages are scraped.

## Extracted Data Points

```
input_id
link
title
category
address
open_hours
popular_times
website
phone
plus_code
review_count
review_rating
reviews_per_rating
latitude
longitude
cid
status
descriptions
reviews_link
thumbnail
timezone
price_range
data_id
images
reservations
order_online
menu
owner
complete_address
about
user_reviews
emails
```

**Note**: email is empty by default (see Usage)

## Quickstart

file `results.csv` will contain the parsed results.

**If you want emails use additionally the `-email` parameter**

### On your host

(tested only on Ubuntu 22.04)

```
cd google-maps-scraper
go mod download
go build
./google-maps-scraper -input example-queries.txt -results restaurants-in-cyprus.csv -exit-on-inactivity 3m
```

Be a little bit patient. In the first run it downloads required libraries.

The results are written when they arrive in the `results` file you specified

**If you want emails use additionally the `-email` parameter**

### Command line options

try `./google-maps-scraper -h` to see the command line options available:

```
  -c int
        sets the concurrency [default: half of CPU cores] (default 8)
  -cache string
        sets the cache directory [no effect at the moment] (default "cache")
  -data-folder string
        data folder for web runner (default "webdata")
  -debug
        enable headful crawl (opens browser window) [default: false]
  -depth int
        maximum scroll depth in search results [default: 10] (default 10)
  -dsn string
        database connection string [only valid with database provider]
  -email
        extract emails from websites
  -exit-on-inactivity duration
        exit after inactivity duration (e.g., '5m')
  -geo string
        set geo coordinates for search (e.g., '37.7749,-122.4194')
  -input string
        path to the input file with queries (one per line) [default: empty]
  -json
        produce JSON output instead of CSV
  -lang string
        language code for Google (e.g., 'de' for German) [default: en] (default "en")
  -produce
        produce seed jobs only (requires dsn)
  -proxies string
        comma separated list of proxies to use
  -proxy-password string
        password for proxy authentication
  -proxy-username string
        username for proxy authentication
  -results string
        path to the results file [default: stdout] (default "stdout")
  -web
        run web server instead of crawling
  -writer string
        use custom writer plugin (format: 'dir:pluginName')
  -zoom int
        set zoom level (0-21) for search
```

## Using a custom writer

In cases the results need to be written in a custom format or in another system like a db a message queue or basically anything the Go plugin system can be utilized.

Write a Go plugin (see an example in examples/plugins/example_writeR.go)

Compile it using (for Linux):

```
go build -buildmode=plugin -tags=plugin -o ~/mytest/plugins/example_writer.so examples/plugins/example_writer.go
```

Please replace the values or the command args accordingly

Note: Keep in mind that because the application starts a headless browser it requires CPU and memory.
Use an appropriate kubernetes cluster

## Telemetry

Anonymous usage statistics are collected for debug and improvement reasons.
You can opt out by setting the env variable `DISABLE_TELEMETRY=1`

## Perfomance

Expected speed with concurrency of 8 and depth 1 is 120 jobs/per minute.
Each search is 1 job + the number or results it contains.

Based on the above:
if we have 1000 keywords to search with each contains 16 results => 1000 \* 16 = 16000 jobs.

We expect this to take about 16000/120 ~ 133 minutes ~ 2.5 hours

If you want to scrape many keywords then it's better to use the Database Provider in
combination with Kubernetes for convenience and start multipe scrapers in more than 1 machines.
