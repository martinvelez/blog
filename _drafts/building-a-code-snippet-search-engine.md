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

Given a keyword query, find a set of snippets. 

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

In Stack Overflow, most code snippets are found inside '\<pre\>' tags.  A posts
can have more than one snippet of code.  For each post, we extracted the text
inside '\<pre\>' tags.  Note that a posts can have more than one snippet of
code.  A posts can have a snippet of code found in other posts.   We identified
duplicates and removed all duplicate snippets.  The table below shows how many
posts and snippets we found and extracted for each programming language tag.

Tag | Posts | Snippets | Unique Snippets 
:------------- | -------------: | -------------: | -------------: 
C | 220,085 | 306,716 | 303,986
C# | 962,755 | 1,291,526 | 1,281,136 
C++ | 452,908 | 660,441 | 655,008
Java | 1,086,506 | 1,539,388 | 1,522,709 
Python | 587,679 | 947,241 | 932,529 


### Indexing Snippets

We tokenized each post to extract its set of words using the tokenize function
in the Appendix.  Since the posts contains both English text and code, the words
we extract belong to the union of English and the respective programming
language of the post. 

The table below shows how many unique words we found in each set of posts.  It
also shows how many word-to-snippet edges we found, and to how many snippets each
word points to, on average. 

Tag | Words | Words/Post | Word-to-Snippet Edges | Mean Snippets/Word
:------------- | -------------: | -------------: | -------------: | -------------:
C | 954,873 | 0.00 | 34,735,409 | 42.79 
C# | 3,441,419 | 0.00 | 153,264,533 | 52.27
C++ | 1,807,594 | 0.00 | 77,956,382 | 51.25
Java | 4,047,130 | 0.00 | 0  | 0
Python | 2,289,742 | 0.00 | 107,438,703 | 54.61

For each word, we counted how many snippets it points to.  The following table
lists the top-10 words for each tag.

{% assign c = 284767 %}
{% assign cp = (c * 1.0 / 34735409.0) * 100 %}
{{ c }}
{{ cp }}

Word | C | % | Word | C# | % | Word | C++ | % | Word | Java | % | Word | Python | %
:------------- | -------------: | -------------: | :------------- | -------------: | :------------- | -------------: | :------------- | -------------: | :------------- | -------------: 
I | {{c}} | {{ cp | round: 4 }} | I | 1,218,920 | I  | 620,710 | x  | 0 | I | 904,851
the | 282,278 | to | 1,202,769 | the  | 607,361 | x  | 0 | to | 878,562  
to | 276,315 | the | 1,176,373 | to  | 605,672 | x  | 0 | the | 869,901 
a | 257,874 | a | 1,066,947 | a  | 563,295 | x  | 0 | a | 800,764 
is | 255,140 | is | 1,054,660 | is  | 549,704 | x  | 0 | in | 787,593 
and | 232,321 | in | 1,009,249 | and  | 506,082 | x | 0 | is | 746,705 
in | 226,092 | and | 964,759 | in | 494,377 | x  | 0 | and | 711,191 
of | 217,928 | this | 957,859 | this | 481,016 | x | 0 | this | 651,727 
int | 212,197 | of | 847,140 | of  | 478,500 | x  | 0 | of | 624,748 
0 | 211,774 | it | 839,999 | it  | 452,990 | x  | 0 | for | 611,496 


### Ranking Snippets

Each post has a 'score' attribute, which is the sum of upvotes and downvotes
and can be negative, and a 'favorite_count' attribute, which is the number of
users who have marked the post as a favorite and is greater than or equal to
zero.  We assume that higher scores indicate posts of higher quality and,
transitively, snippets of higher quality.

For our initial ranking scheme, we will rank the query results by the sum of
'score' and 'favorite_count' attributes.

An alternative ranking scheme, i





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



