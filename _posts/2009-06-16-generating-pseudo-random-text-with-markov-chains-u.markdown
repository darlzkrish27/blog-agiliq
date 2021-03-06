---
layout: post
title:  "Generating pseudo random text with Markov chains using Python"
date:   2009-06-16 11:54:11+05:30
categories: algorithms
author: shabda
---
First the definition from [Wolfram](http://mathworld.wolfram.com/MarkovChain.html)

<blockquote>
A Markov chain is collection of random variables {X_t} (where the index t runs through 0, 1, ...) having the property that, given the present, the future is conditionally independent of the past.
</blockquote>

[Wikipedia](http://en.wikipedia.org/wiki/Transition_probabilities) is a little clearer

<blockquote>
 ...Markov chain is a stochastic process with markov property ... [Which means] state changes are probabilistic, and future state depend on current state only.
</blockquote>

Markov chains have various uses, but now let's see how it can be used to generate
gibberish, which might look legit.

The algorithm is,

0. Have a text which will serve as the corpus from which we choose the next
transitions.
1. Start with two consecutive words from the text. The last two words constitute
the present state.
2. Generating next word is the markov transition. To generate the next word, look
in the corpus, and find which words are present after the given two words. Choose
one of them randomly.
3. Repeat 2, until text of required size is generated.

The code for this is

<script src="http://gist.github.com/131679.js"></script>

To see a sample output, we take the text of _My man jeeves_ by Wodehouse from
[Project Gutenberg](http://www.gutenberg.org/etext/8164), and see a sample output.

    In [1]: file_ = open('/home/shabda/jeeves.txt')
    
    In [2]: import markovgen
    
    In [3]: markov = markovgen.Markov(file_)
    
    In [4]: markov.generate_markov_text()
    Out[4]: 'Can you put a few years of your twin-brother Alfred,
    who was apt to rally round a bit. I should strongly advocate
    the blue with milk'


[The files you need to run this are <a href='http://uswaretech.com/blog/wp-content/uploads/2009/06/jeeves.txt'>jeeves.txt</a> and <a href='http://uswaretech.com/blog/wp-content/uploads/2009/06/markovgenpy.txt'>markovgen.py</a>]



### How is this a markov algorithm?

* The last two words are the current state.
* Next word depends on last two words only, or on _present state_ only.
* The next word is _randomly chosen_ from a statistical model of the corpus.


Here is some sample text.

"The quick brown fox jumps over the brown fox who is slow jumps over the brown
fox who is dead."

This gives us a corpus like,

    {('The', 'quick'): ['brown'],
     ('brown', 'fox'): ['jumps', 'who', 'who'],
     ('fox', 'jumps'): ['over'],
     ('fox', 'who'): ['is', 'is'],
     ('is', 'slow'): ['jumps'],
     ('jumps', 'over'): ['the', 'the'],
     ('over', 'the'): ['brown', 'brown'],
     ('quick', 'brown'): ['fox'],
     ('slow', 'jumps'): ['over'],
     ('the', 'brown'): ['fox', 'fox'],
     ('who', 'is'): ['slow', 'dead.']}
 
 Now if we start with "brown fox", the next word can be "jumps" or "who". If we
 choose "jumps", then the current state is "fox jumps" and next word is over,
 and so on.
 
### Hints

* The larger text we choose, the more choices are there at each transition, and
a better looking text is generated.
* The state can be set to depend on one word, two words, or any number of words.
As the number of words in each state increases, the generated text becomes less
random.
* Don't strip the punctuations etc. They lead to a more representative corpus,
and a better random text.

### Resources

* <a href='http://uswaretech.com/blog/wp-content/uploads/2009/06/jeeves.txt'>jeeves.txt</a>
* <a href='http://uswaretech.com/blog/wp-content/uploads/2009/06/markovgenpy.txt'>markovgen.py</a>
* [Online markov text generator](http://www.yisongyue.com/shaney/)
* [Twitter markov script](http://www.yaymukund.com/twittov/)

-------------

Did you know that markov chains have many other uses in finance, statistics and mathematics? In fact Google's page rank algorithm is essentially a markov chain of the web! We publish articles on Algorithms, Python, Django and related technologies every week. Why not [Subscribe to us](http://agiliq.com/newsletter/subscribe/) or [follow us on twitter](http://twitter.com/agiliqdotcom)?

