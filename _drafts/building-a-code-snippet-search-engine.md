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
* Type query.  'open a file in python'.
* Pick a URL. 'pick first'.
* Find snippet on page. '11th snippet on 1st page'.

Problem: 

* There is a lot of information users have to sift through to get to the actual code snippet.
* Not high quality snippets
* keyword matching is over the code and not the context

Can we fulfill this information need better?

## Related Work 
Vocabulary: code
Query: keyword
Set: Open Source Projects

* https://searchcode.com
* https://www.openhub.net/
* Google code search (defunct)

Vocabulary: code and natural language
Query: keyword
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
website where users posts programming-related *questions*, and, usually, other
users posts *answers*. The user who posted the question usually selects one of the
answers.  This answer is called the *accepted answer*.  Users can vote posts up
or down.

Stack Exchange ([http://stackexchange.com/](http://stackexchange.com/)), the
company that owns Stack Overflow and other Q&A websites, published a data dump
of all of their websites including Stack Overflow on the Internet Archive
([https://archive.org/details/stackexchange](https://archive.org/details/stackexchange))
on June 13, 2016.  We downloaded it on August 3, 2016.

We downloaded the stackoverflow.com-Posts.7z, a 8.8G file.  This compressed file
contains a 44G file caled Posts.xml.  There is no detailed description but it
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

Symbol | Statistic | C | C# | C++ | Java | Python
:--- | :--- | ---: | ---: | ---: | ---: | ---:
- | Posts with Snippets | 165,935 | 686,621 | 334,821 | 780,777 | 467,566
- | all posts | 220,085 | 962,755 | 452,908 | 1,086,506 | 587,679
N | Code Snippets (documents) | 303,986 | 1,281,136 | 655,008 | 1,522,709 | 932,529
- | avg. # of snippets per post | 1.85 | 1.88 | 1.97 | 1.97 | 2.03
- | avg. # of snippets per term | 41.64 | 52.19 | 50.09 | 61.05 | 52.64
M | Terms (unique, case folding) | 800,754 | 2,768,494 | 1,491,612 | 3,306,456 | 1,952,072
- | avg. # of tokens (words) per post | 636.05 | 593.38 | 661.63 | 795.53 | 637.92

For each word, we counted how many snippets it points to.  The following table
lists the top-10 words for each tag.


Word (c) | Snippets | % |Word (c#) | Snippets | % |Word (c++) | Snippets | % |Word (java) | Snippets | % |Word (python) | Snippets | % 
:--- | ---: | ---: |:--- | ---: | ---: |:--- | ---: | ---: |:--- | ---: | ---: |:--- | ---: | ---: 
i | 294016 | 0.88 |i | 1252663 | 0.87 |i | 634703 | 0.85 |i | 1484667 | 0.74 |i | 912531 | 0.89 
the | 283017 | 0.85 |to | 1195481 | 0.83 |the | 609632 | 0.82 |to | 1405202 | 0.70 |to | 866589 | 0.84 
to | 274416 | 0.82 |the | 1180694 | 0.82 |to | 601768 | 0.81 |the | 1399591 | 0.69 |the | 866318 | 0.84 
is | 259572 | 0.78 |is | 1081368 | 0.75 |a | 565735 | 0.76 |is | 1295443 | 0.64 |a | 797027 | 0.78 
a | 258010 | 0.77 |a | 1068197 | 0.74 |is | 563222 | 0.75 |a | 1228642 | 0.61 |in | 791091 | 0.77 
and | 237314 | 0.71 |in | 1028865 | 0.71 |and | 518141 | 0.69 |and | 1191358 | 0.59 |is | 764424 | 0.74 
in | 232866 | 0.70 |this | 1006546 | 0.70 |in | 512532 | 0.69 |in | 1191134 | 0.59 |and | 721980 | 0.70 
this | 220304 | 0.66 |and | 995178 | 0.69 |this | 506810 | 0.68 |this | 1188850 | 0.59 |this | 691557 | 0.67 
of | 216761 | 0.65 |it | 871196 | 0.60 |of | 476137 | 0.64 |it | 1028621 | 0.51 |for | 629272 | 0.61 
it | 212731 | 0.64 |of | 844138 | 0.58 |it | 469052 | 0.63 |of | 972197 | 0.48 |of | 618666 | 0.60


### Ranking Snippets

We define a static quality score for documents.  Each post has a 'score'
attribute, which is the sum of upvotes and downvotes and can be negative, and a
'favorite_count' attribute, which is the number of users who have marked the
post as a favorite and is greater than or equal to zero.  We assume that higher
scores indicate posts of higher quality and, transitively, snippets of higher
quality. Let d be a document, then 

* score(d) = score + favorite_count

We define a scoring function for retrieved documents given a query.
Let *q* be a query and *d* be a document, then

* score(q,d) = score(d) = score + favorite_count

**We rank documents, in decreasing order, by their *score(q,d)*.**
We will use this as our initial ranking scheme and later evaulate alternative
scoring functions.

* score(q,d) = for all t in q, sum of tf-id(t,d) 
* score(q,d) = cosine_sim(q,d)
* lnc.ltc (pg.118 of IR by Manning)


### Related Snippets

Find similar documents: 

* Calculate term frequencies for posts.
* Calculate euclidean normalized tf values. 
* Precomute document -> ranked list of similar documents



## Evaluation

BDA

Elastic Search

How often do users seek context as well as code? 
Provide link to page.
Track how often users click.


## Discussion

### User Interface

1. Look at krugle
2. Add filter by token
3. Add upvote counts.
4. Add compilation result info.
5. Allow search over multiple languages.

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



