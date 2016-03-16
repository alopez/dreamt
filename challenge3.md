---
layout: default
img: exam
img_link: http://www.flickr.com/photos/gianellbendijo/4034021658/
caption: Image by gianellbendijo (used with permission)
title: Challenge 3 | Evaluation
---

Evaluation <span class="text-muted">Challenge Problem 3</span>
==============================================================

Automatic evaluation is a key problem in machine translation. 
Suppose that we have two machine translation systems. On one 
sentence, system A outputs:

_This type of zápisníku was very ceněn writers and cestovateli._

And system B outputs:

_This type of notebook was very prized by writers and travellers._

We suspect that system B is better, though we don't necessarily
know that its translations of the words _zápisníku_, _ceněn_,
and _cestovateli_ are correct. But suppose that we also have
access to the following reference translation.

_This type of notebook is said to be highly prized by writers and travellers._

We can easily judge that system B is better. __Your challenge is to 
write a program that makes this judgement automatically__.

Getting started
---------------

If you don't already have a copy of the code:

    git clone https://github.com/alopez/dreamt.468.git

Under the `evaluate` directory, you now have simple program
that decides which of two machine translation outputs is better.
Test it out!

    python evaluate > eval.out

This challenge uses a very simple evaluation method. Given
machine translations $$h_1$$ and $$h_2$$ and reference translation
$$e$$, it computes $$f(h_1, h_2, e)$$ as follows, where $$\ell(h,e)$$ 
is the count of words in $$h$$ that are also in $$e$$.

<center>
$$f(h_1,h_2,e) = \left\{\begin{array}{lcc}1 & \textrm{if} & \ell(h_1,e) > \ell(h_2,e)\\ 0  & \textrm{if} & \ell(h_1,e) = \ell(h_2,e)\\-1  & \textrm{if} & \ell(h_1,e) < \ell(h_2,e) \end{array}\right.$$
</center>

We can compare the results of this function with those of human 
annotator who rated the same translations.

    python compare-with-human-evaluation < eval.out
    
The Challenge
-------------

Your challenge is to __improve the accuracy of automatic evaluation as 
much as possible__. Improving the metric to use the simple METEOR metric
in place of $$\ell(h, e)$$ is sufficient to pass. Simple METEOR computes
the harmonic mean of precision and recall. That is:

<center>
$$\ell(h,e) = \frac{P(h,e)\cdot R(h,e)}{(1-\alpha)R(h,e)+\alpha P(h,e)}$$
</center>

where $$P$$ and $$R$$ are precision and recall, defined as:

<center>
$$\begin{array}{c}
R(h,e) = \frac{|h\cap e|}{|e|}\\
P(h,e) = \frac{|h\cap e|}{|h|}
\end{array}$$
</center>

Be sure to tune the parameter $$\alpha$$ that balances precision and
recall. This is a very simple 
baseline to implement. However, evaluation is not solved,
and the goal of this challenge is for you to experiment with methods
that yield improved predictions of relative translation accuracy. Some
things that you might try:

* Learn [a classifier](http://aclweb.org/anthology//W/W11/W11-2113.pdf) from the training data.
* Use [WordNet](http://wordnet.princeton.edu/) to match synonyms.
* Compute string similarity using [string subsequence kernels](http://jmlr.org/papers/volume2/lodhi02a/lodhi02a.pdf).
* Use an n-gram language model to better assess fluency.
* Develop a single-sentence variant of [BLEU](http://aclweb.org/anthology//P/P02/P02-1040.pdf).
* Use a dependency parser to [assess syntactic well-formedness](http://ssli.ee.washington.edu/people/jgk/dist/metaweb/mtjournal.pdf).
* Develop a method to automatically assess semantic similarity.
* See what evaluation measures [other people have implemented](http://www.statmt.org/wmt10/pdf/wmt10-overview.pdf).

But the sky's the limit! Automatic evaluation is far from solved, and there
are many different solutions you might invent. 

Acknowledgements
----------------

This challenge was designed by [Chris Dyer](http://www.cs.cmu.edu/~cdyer)
based on one we gave in [2012](http://mt-class.org/past/jhu/2012/hw3.html), which also inspired a 
[whole](http://aclweb.org/anthology//W/W12/W12-3101.pdf)
[series](http://aclweb.org/anthology//W/W12/W12-3102.pdf) 
[of](http://hltc.cs.ust.hk/iwslt/proceedings/paper_34.pdf) 
[papers](http://aclweb.org/anthology//P/P13/P13-1139.pdf).*
