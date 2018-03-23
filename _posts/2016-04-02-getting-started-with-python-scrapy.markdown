---
layout: post
title:  "Getting started with python scrapy"
date:   2016-04-02 21:04:43+05:30
tags: scrapy
author: manjunath
---
Sometimes you may want to extract data from website. What if the website don't provide an API? don't worry scrapy comes to your rescue.

`Scrapy` is a python library which can crawl websites and extract data.

We will try to extract small piece of data from [Times of India](http://timesofindia.indiatimes.com/) and store the extracted data in our django model for our example.

### To get started install scrapy
    pip install scrapy

For more information on installing scrapy on different platforms [check this](http://doc.scrapy.org/en/latest/intro/install.html)

Alright, we have installed scrapy and lets create a new scrapy project

    scrapy startproject toi

`toi` is the name of our project(Times of India)


