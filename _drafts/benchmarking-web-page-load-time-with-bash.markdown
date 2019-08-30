---
layout: post
title:  "Benchmarking web page load time with bash"
categories: bash performance
---

A couple of days ago, I needed to measure the performance of a category page on Magento, with different sorting and filtering parameters. I needed the test to be objective and reproducible, in order to check for performance loss after some changes.

Well, this sounds like a perfect job for a bash script! I have written a very short one, which just takes an input url and appends the parameter `no_cache=1` afterwards, in order to check with and without cache:

```bash
url="$1"
bold=$(tput bold)
normal=$(tput sgr0)

withoutCache() {
    local url="$1"
    if [[ $url == *"?"* ]]; then
        echo "${url}&no_cache=1"
    else
        echo "${url}?no_cache=1"
    fi
}

checkResponseTime() {
    local url="$1"
    responseTime=$(curl -s -w %{time_total}\\n -o /dev/null "$url")
    echo -e "Checking response time for $url (cached) - ${bold}$responseTime${normal}"
    url=$(withoutCache $url)
    responseTime=$(curl -s -w %{time_total}\\n -o /dev/null "${url}")
    echo -e "Checking response time for $url (not cached) - ${bold}$responseTime${normal}"
}

checkResponseTime $url
```