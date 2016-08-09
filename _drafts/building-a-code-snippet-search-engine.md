---
layout: post
title:  "Building a Code Snippet Search Engine"
author: "Martin Velez"
date:   2016-08-03 19:00:00 -0700
categories: research 
tags: ['research', 'code reuse', 'code search', 'search engine', 'stack overflow']
---

## Introduction
There is vast amount of code snippets online.
Code snippets are short pieces of code.
Programmers often search online for snippets that they can reuse or they can
learn from.


google or bing
stackoverflow, tutorials point, or thousands of websites with code snippets

## Related Work 
Bing Developer Assistant

CSnippex

## Methodology

1. Get an initial set of snippets.
2. Index snippets by word 
3. Define a ranking algorithm  
4. Build frontend
5. Publish frontend online

### Corpus of Snippets
Stack Overflow ([http://stackoverflow.com/](http://stackoverflow.com)) is a
website where users posts programming-related questions, and, usually, other
users posts answers. The user who posted the question usually selects one of the
answers.  This answer is called the *accepted answer*.  Users can vote posts up
or down.

Stack Exchange ([http://stackexchange.com/](http://stackexchange.com/)), the
company that owns Stack Overflow and other Q&A websites, published a data dump
of all of their websites including Stack Overflow on the Internet Archive
([https://archive.org/details/stackexchange](https://archive.org/details/stackexchange))
on June 13, 2016.  We downloaded it on August 3, 2016.

We downloaded the stackoverflow.com-Posts.7z, a 8.8G file.  This compressed file
contains a 44G file caled Posts.xml  There is no detailed description but it
appears to contain all of the Stack Overflow posts, which consists of questions
and answers.  For our purposes, this dataset appears to be large enough to build
a seed database of snippets.

#### Extracting Snippets
Users tag posts with words like 'android', 'c#', 'python', 'python-2.7', 'java'
to identify the main topics of a posts.  Tags are created by users but users
tend to reuse tags, like 'python'.  Stack Overflow uses these tags to sort
posts.  

We used the post tags to sort posts by programming language since our queries
will be restricted to a particular programming language.  We will focus on
Python, C, C++, Java, the main programming languages taught at UC Davis, and C#,
the programming language Bing Developer Assistant supports.  

In Stack Overflow, most code snippets are found inside '\<pre\>' tags.  For each
post, we extracted the text inside '\<pre\>' tags.  Afterward, we identified
duplicates and removed them.


Tag | Post Count | Snippet Count | Unique Snippet Count
:------------- | -------------: | -------------: | -------------: 
C | 220,085 | 306,716 | 303,986
C# | 962,755 | 1,291,526 | 1,281,136 
C++ | 452,908 | 660,441 | 655,008
Java | 1,086,506 | 1,539,388 | 1,522,709 
Python | 587,679 | 947,241 | 932,529 


### Indexing Snippets
We tokenized each post to extract its set of words.  Since the posts contains
both English text and code, the words we extract belong to the union of English
and the respective programming language of the post.  

Tag | Word Count | Unique Words 
:------------- | -------------: | -------------:
C | 0 | 0 
C# | 0 | 0 
C++ | 0 | 0 
Java | 0 | 0 
Python | 0 | 0 




## Evaluation

BDA

Elastic Search

## Discussion


## Conclusion


