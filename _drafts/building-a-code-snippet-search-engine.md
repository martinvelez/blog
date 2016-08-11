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
Different from searching actual projects.

We estimate that there are thousands of websitew with code snippets, like Stack
Overflow and Tutorial Point.

Currently, programmers use a general-purpose search engine, like Google or Bing,
to find the snippet they are searching for.

Example:
* Open Google.
* Type query.
* Pick a URL.
* Find snippet on page.

Problem: 
* There is a lot of information users have to sift through to get to the actual code snippet.
* Not high quality snippets
* keyword matching is over the code and not the context

Can we fulfill this information need better?

## Related Work 
Query: keyword
Set: Open Source Projects
* https://searchcode.com
* https://www.openhub.net/
* Google code search (defunct)
* Huawei internal code search


Query: natural language keyword query
Set: Code Snippets
* Bing Developer Assistant

Automatically fixing code snippets
* CSnippex

## Design and Implementation 
Given a natural language keyword query, find a set of snippets. 

0. Keyword queries
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

In Stack Overflow, users usually wrap code snippets inside '\<pre\>' tags.  A
posts can have more than one snippet of code.  For each post, we extracted the
text inside '\<pre\>' tags.  We call the t  Note that a posts can have more than
one snippet of code.  A posts can have a snippet of code found in other posts.
We identified duplicates and removed all duplicate snippets.  

The table below shows our results of processing the Posts.xml file to extract
posts and snippets.

Tag | All Posts | Posts with Snippets | Snippets | Snippets/Post
:--- | ---: | ---: | ---: | ---:
c | 220,085 | 165,935 | 303,986 | 1.85
c# | 962,755 | 686,621 | 1,281,136 | 1.88
c++ | 452,908 | 334,821 | 655,008 | 1.97
java | 1,086,506 | 780,777 | 1,522,709 | 1.97
python | 587,679 | 467,566 | 932,529 | 2.03


### Indexing Snippets

We tokenized each post to extract its set of words using the tokenize function
in the Appendix.  Since queries are usually case-insensitive, we also store only
the lowercase version of words, for example, "I" is stored as "i".  Since the
posts contains both English text and code, the words we extract belong to the
union of English and the respective programming language of the post.  This
should allow users to use programming language terms in their queries. 

The table below shows how many unique words we found in each set of posts.  It
also shows how many word-to-snippet edges we found, and to how many snippets each
word points to, on average. 


Tag | Words | Words/Post | Word-to-Snippet Edges | Mean Snippets/Word
:------------- | -------------: | -------------: | -------------: | -------------:
c | 800,754 | 636.05 | 33,341,781 | 41.63 
c# | 2,768,494 | 593.38 | 138,166,285 | 204.14
c++ | 1,491,612 | 661.63 | 70,897,739 | 196.09
java | 3,306,456 | 795.53 | 193,544,817 | 228.73
python | 1,952,072 | 637.92 | 97,945,358 | 210.30

For each word, we counted how many snippets it points to.  The following table
lists the top-10 words for each tag.

Word | C 
:--- | ---:
i|294016
the|283017
to|274416
is|259572
a|258010
and|237314
in|232866
this|220304
of|216761

Word | C | % | Word | C# | % | Word | C++ | % | Word | Java | % | Word | Python | %
:------------- | -------------: | -------------: | :------------- | -------------: | :------------- | -------------: | :------------- | -------------: | :------------- | -------------: 
I | 284,767 | 0 | I | 1,218,920 | 0 | I  | 620,710 | 0 | I  | 0 | 0 | I | 904,851 | 0
the | 282,278 | 0 | to | 1,202,769 | 0 | the  | 607,361 | 0 | to  | 0 | 0 | to | 878,562 | 0 
to | 276,315 | 0 | the | 1,176,373 | 0 | to  | 605,672 | 0 | the  | 0 | 0 | the | 869,901 | 0 
a | 257,874 | 0 | a | 1,066,947 | 0 | a  | 563,295 | 0 | is | 0 | 0 | a | 800,764 | 0
is | 255,140 | 0 | is | 1,054,660 | 0 | is  | 549,704 | 0 | a  | 0 | 0 | in | 787,593 | 0
and | 232,321 | 0 | in | 1,009,249 | 0 | and  | 506,082 | 0 | in | 0 | 0 | is | 746,705 | 0
in | 226,092 | 0 | and | 964,759 | 0 | in | 494,377 | 0 | and | 0 | 0 | and | 711,191 | 0
of | 217,928 | 0 | this | 957,859 | 0 | this | 481,016 | 0 | this | 0 | 0 | this | 651,727 | 0
int | 212,197 | 0 | of | 847,140 | 0 | of  | 478,500 | 0 | it | 0 | 0 | of | 624,748 | 0
0 | 211,774 | 0 | it | 839,999 | 0 | it  | 452,990 | 0 | of | 0 | 0 | for | 611,496 | 0


### Ranking Snippets

Each post has a 'score' attribute, which is the sum of upvotes and downvotes
and can be negative, and a 'favorite_count' attribute, which is the number of
users who have marked the post as a favorite and is greater than or equal to
zero.  We assume that higher scores indicate posts of higher quality and,
transitively, snippets of higher quality.

For our initial ranking scheme, we will rank the query results by the sum of
'score' and 'favorite_count' attributes.

## Evaluation

BDA

Elastic Search

How often do users seek context as well as code? 
Provide link to page.
Track how often users click.


## Discussion


## Conclusion


## Appendix

### Tokenizer 

We used the following Ruby function to tokenize each post.

{% highlight ruby linenos %}
def tokenize(html_fragment)
	text = ''
	doc = Nokogiri::HTML.fragment(html_fragment)
	doc.traverse do |node|
		text += node.text
	end			
	words = text.split(/[^[[:word:]]]+/)

	return words
end

{% endhighlight %}



