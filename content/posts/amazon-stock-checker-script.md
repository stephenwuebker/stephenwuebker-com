---
title: "Amazon Stock Checker Script"
date: 2022-12-30T11:31:06-05:00
draft: false

thumbnail: images/python-logo.png

tags: 
  - "python"

socialshare: true
---

Recently, I was shopping for some upgrades for my home office on Amazon and a few of the things I really wanted were out of stock at the time. I thought it was really annoying to have to remember to go and check on the things on a periodic basis to see if they were back in stock, and I didn't see anything on Amazon's site that would send me an email or anything if and when the stock was replenished. **(Hint to Amazon: this would be a great idea!)**

So, let's fire up the old webscraper and code ourselves a solution to this problem. The basic idea here is to read a list of product URLs and use BeautifulSoup or something similar to scrape the page looking for text that is something like "Currently Unavailable". If it's found, it's not yet back in stock, so do nothing. Otherwise, send an email with the link to me so I can buy it.

Well, OpenAI and specifically the ChatGPT bot have been getting a lot of press about how it can just write programs for you. \<scarequotes\> **IT'S GOING TO PUT US ALL OUT OF A JOB!!** \</scarequotes\> Yeah. Well, we'll see about that. I thought this might be a good way to test the bot's dev chops, so I gave it the following prompt:

`"Python script to check a list of products on amazon and see if they are back in stock"`

This is what it gave me back:

python code from chat.openai.com:

```python
import requests from bs4 
import BeautifulSoup

#enter the Amazon product URLs here products = ["[https://www.amazon.com/product1](https://www.amazon.com/product1)", "[https://www.amazon.com/product2](https://www.amazon.com/product2)", "[https://www.amazon.com/product3](https://www.amazon.com/product3)"]

for product in products: 
response = requests.get(product) 
soup = BeautifulSoup(response.text, "html.parser") 
availability = soup.find("span", {"class": "a-size-medium a-color-success"})

if availability:
    print(product + " is back in stock!")
else:
    print(product + " is still out of stock.")

```

The Open AI bot kind of had the right idea, but ultimately, it didn't work at all. I don't think AI is quite there yet. I can envision it being a good tool to output some basic framework code or give examples or prompts, and then an actual person can complete or write the actual code. I can see it helping immensely with code output and speed. Relying on AI *right now* for anything approaching accurate and secure code is folly.

All that said, scrapehero.com actually had a good example.

https://www.scrapehero.com/tutorial-how-to-scrape-amazon-product-details-using-python-and-selectorlib/

They used selectorlib to get the right css elements. It's super easy to use. Their example was wildly out of date, of course. But, using selectorlib and the browser plugin to generate a selectors.yml file is very straightforward. I only grabbed the price field and the product name as that was really all I was interested in. Then, you just add the page urls for each product you want to check to a urls.txt file. The python script will go through that file and load each page, pulling the elements defined in the selectors file and grabbing the data from the Amazon page, and then outputting it as a json lines file. 

Code to pull data into json lines file:

```python
from selectorlib import Extractor
import requests 
import json 
from time import sleep

# Create an Extractor by reading from the YAML file
e = Extractor.from_yaml_file('selectors.yml')
def scrape(url):    
    headers = {
    'authority': 'www.amazon.com',
    'pragma': 'no-cache',
    'cache-control': 'no-cache',
    'dnt': '1',
    'upgrade-insecure-requests': '1',
    'user-agent': 'Mozilla/5.0 (X11; CrOS x86_64 8172.45.0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/51.0.2704.64 Safari/537.36',
    'accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9',
    'sec-fetch-site': 'none',
    'sec-fetch-mode': 'navigate',
    'sec-fetch-dest': 'document',
    'accept-language': 'en-GB,en-US;q=0.9,en;q=0.8',
    }
    # Download the page using requests
    print("Downloading %s"%url)
    r = requests.get(url, headers=headers)
    # Simple check to check if page was blocked (Usually 503)
    if r.status_code > 500:
        if "To discuss automated access to Amazon data please contact" in r.text:
            print("Page %s was blocked by Amazon. Please try using better proxies\n"%url)
        else:
            print("Page %s must have been blocked by Amazon as the status code was %d"%(url,r.status_code))
        return None
        # Pass the HTML of the page and create 
    return e.extract(r.text)
    # product_data = []

with open("urls.txt",'r') as urllist, open('output.jsonl','w') as outfile:
    for url in urllist.readlines():
        data = scrape(url) 
        if data:
            json.dump(data,outfile)
        outfile.write("\n")
```

From here, we could read that json file into a database to track the status of each item, and know if it came back in stock, and if we've already sent an email about the product.

I just checked the file manually for a couple of days, and things were back in stock before I ever got around to building out any kind of tracking and notification system. I guess I'll get around to that one of these days...

