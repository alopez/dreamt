---
layout: default
title: DREAMT | Home
base_url: "./"
img: artsrouni
img_link: http://www.hutchinsweb.me.uk/IJT-2004.pdf
caption: Georges Artsrouni's mechanical brain, a translation device patented in 1933 in France.
active_tab: main_page
---
<div class="page-header">
  <h1>Welcome to DREAMT!</h1>
  <p class="lead">Decoding, Reranking, Evaluation, and Alignment for Machine Translation</p>
</div>

DREAMT consists of code and data for students studying the
basic algorithms behind statistical machine translation, 
the technology that drives popular web services like
[Google Translate](http://translate.google.com),
[Bing Translator](http://microsofttranslator.com), and
[SDL FreeTranslation](http://freetranslation.com).

The best way to really understand how these algorithms work is to
implement them yourself and see how they behave on real data.
DREAMT provides you with data, simple baseline algorithms, and
code to check the accuracy of your solution. You can run the baseline straight out of the box
and try simple experiments within minutes. Then, to test your knowledge
of [textbook algorithms](http://www.statmt.org/book), you 
can implement them and see how they improve on our baselines. The 
textbook algorithms only require a few dozen lines of code at most, and
you'll understand them much better once you derive and implement them
yourself.

But the sky's the limit! Machine translation is hardly a solved problem.
Can you do better than the textbook algorithms? Test out your NLP 
hacking skills and see how accurately you can solve these four challenges:

* [The Alignment challenge](challenge1.html): given some translated sentences, identify the word-to-word translations within them.
* [The Decoding challenge](challenge2.html): given a statistical translation model and some foreign sentences, return the most probable English translations.
* [The Evaluation challenge](challenge3.html): given the output of several translation systems, rank them by their accuracy.
* [The Reranking challenge](challenge4.html): given a set of possible English translations of a foreign sentence, choose the best one.

If you're an instructor, or you're simply curious, you can read more 
about [why we created DREAMT](about.html).
