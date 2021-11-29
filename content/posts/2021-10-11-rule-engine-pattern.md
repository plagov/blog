+++
title = "How I learned about the rule engine pattern with Java"
date = "2021-10-11"
author = "Vitali Plagov"
draft = true
+++

In this blog post I would like to describe how I learned about the rule engine pattern while contributing to the
open source project.
<!--more-->

Software developers work with 3rd party open source libraries on a daily basis. The more you use a library in your 
day-to-day work, the more familiar you become with it. In my job as a test automation engineer, I work with 
[Selenide](https://github.com/selenide/selenide) all the time. So, when I had to accomplish a certain task, I found 
out that Selenide doesn't have a solution to help me with. So, I thought it could be a good opportunity to contribute 
a new functionality into the open source library.

## Problem

Here's the problem I had. The Selenide library has a function that finds an ancestor in the HTML DOM of the current 
element. The ancestor is the same HTML element, so it has different attributes the one can use to locate it on the 
page - a tag name, a class name, an attribute, an attribute with a value. So, the task is to build an XPath 
expression for each of options.

A very straightforward solution would be to make an `if - else if - else` approach. But it's clear that with four 
options mentioned above that construct would look too sloppy. There must be a better and more clean approach of 
accomplishing this.
