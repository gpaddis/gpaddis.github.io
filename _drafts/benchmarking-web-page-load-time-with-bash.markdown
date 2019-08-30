---
layout: post
title:  "Benchmarking web page load time with cURL"
categories: bash performance
---

A couple of days ago, I needed to measure the performance of a category page in a Magento shop, with different combinations of sorting and filtering parameters. I needed the test to be objective and reproducible, in order to check for performance before and after some changes.

Well, this sounds like a perfect job for a cURL + a bash script! I have written a short one[^1], which just takes an input url and appends the parameter `no_cache=1` afterwards, in order to measure the load time with and without cache. Here's the output:

```
$ ./benchmark-url.sh http://www.example.com/
Checking response time for http://www.example.com/ (cached) - 0,189899
Checking response time for http://www.example.com/?no_cache=1 (not cached) - 0,207218
```

The script leverages a great feature of cURL[^2], which can also return additional information about the transaction: for instance, the duration (with the parameter `-w` or `--write-out` and a formatting string):

```bash
curl -s -w %{time_total}\\n -o /dev/null "www.example.com"
```

In this example, I was just looking at the total time of the operation, but the report can become very granular if you also check for other values, e.g. lookup time, connect time, and many more metrics[^3].

### Notes

[^1]: Check out the script on [GitHub](https://github.com/gpaddis/scripts/blob/master/benchmark-url.sh)
[^2]: cURL Basic Tutorial for HTTP Timing Measurement - [Medium](https://medium.com/@pvouzis/curl-basic-tutorial-for-http-timing-measurement-dd8dafaa9400)
[^3]: A more detailed description of cURL and its capabilities - [HTTP Transaction Timing Breakdown with cURL](https://netbeez.net/blog/http-transaction-timing-breakdown-with-curl/)