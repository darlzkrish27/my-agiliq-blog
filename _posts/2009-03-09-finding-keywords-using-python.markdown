---
layout: post
title:  "Finding keywords using Python"
date:   2009-03-09 23:49:56+05:30
tags: algorithms
author: lakshman
---
Update: keywords2.txt is Pride and Prejudice from Project Gutenberg. <a href='http://uswaretech.com/blog/wp-content/uploads/2009/03/keywords2.txt'>Attached for convenience.</a>

-------------

Finding keywords in a given snippet of text has many uses. From classifying web pages to fighting spam mails, keyword recognition is the first step
in many algorithms.

We here show the naive Bayesian filter to find keywords, which was popularised by [Paul Graham](http://www.paulgraham.com/spam.html) to discover spam mails.

###Steps to find keywords.

1. Have a large corpus of text against which we will compare.
2. Find the relative frequency of words in corpus. Eg if your corpus is "the green way is very green way green".
Relative frequency is..
[
(the, 1/8),
(green, 3/8),
(way, 2/8),
(is, 1/8),
(very, 1/8),
]
3. Get the relative frequency of words in search string.
4. Divide frequency of search string by corpus, taking care of cases like when a word is not in corpus.
5. Sort by answer in 4, the top n words are keyword.

###Why does this work?

The relative frequency in corpus string is a measure of how frequently the word occurs generally. Dividing search frequency word by
relative corpus frequency gets a measure of how frequently the word occurs relative to normal. Sorting on this gives us the keywords.

###The code

The code is longer than it needs to be make every step as clear as possible

	def find_keyword(test_string = 'Hacker news is a good site while Techcrunch not so much'):
		key_file = open('keywords2.txt')
		data = key_file.read()
		words = data.split()
		word_freq = {}
		for word in words:
		    if word in word_freq:
			word_freq[word]+=1
		    else:
			word_freq[word] = 1
		word_prob_dict = {}
		size_corpus = len(words)
		for word in word_freq:
		    word_prob_dict[word] = float(word_freq[word])/size_corpus

		prob_list = []
		for word, prob in word_prob_dict.items():
		     prob_list.append(prob)
		non_exist_prob = min(prob_list)/2

		words = test_string.split()
		test_word_freq = {}
		for word in words:
		    if word in test_word_freq:
			test_word_freq[word]+=1
		    else:
			test_word_freq[word] = 1

		test_words_ba = {}
		for word, freq in test_word_freq.items():
		    if word in word_prob_dict:
			test_words_ba[word] = freq/word_prob_dict[word]
		    else:
			test_words_ba[word] = freq/non_exist_prob

		test_word_ba_list = []
		for word, ba in test_words_ba.items():
		     test_word_ba_list.append((word, ba))

		def sort_func(a, b):
		    if a[1] > b[1]:
		       return -1
		    elif a[1] < b[1]:
			return 1
		    return 0

		test_word_ba_list.sort(sort_func)
		return test_word_ba_list[:2]



###Output

Here is some sample output.

	In [59]: ff.find_keyword
	Out[59]: <function find_keyword at 0x92b7e9c>

	In [60]: ff.find_keyword()
	Out[60]: [('Hacker', 249160.0), ('Techcrunch', 249160.0)]

	In [61]: ff.find_keyword('')
	Out[61]: []

	In [62]: ff.find_keyword('Java is an island and a programming language')
	Out[62]: [('Java', 249160.0), ('island', 249160.0)]

	In [63]: ff.find_keyword('Java is an island and a programming language. Python is a snake and a programming')
	Out[63]: [('programming', 498320.0), ('Java', 249160.0)]

	In [64]: ff.find_keyword('Java is an island and a programming language')
	Out[64]: [('Java', 249160.0), ('island', 249160.0)]

	In [65]: ff.find_keyword('Java is an island and a programming  programming language')
	Out[65]: [('programming', 498320.0), ('Java', 249160.0)]


--------------------------------

Need to build a web app which does more than read from a database? Contact us at sales@uswaretech.com, to discuss further.
			  
		 

