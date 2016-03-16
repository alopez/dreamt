---
layout: default
img: rosetta
img_link: http://en.wikipedia.org/wiki/Rosetta_Stone
caption: Rosetta stone (credit&#58; flickr user calotype46)
title: Challenge 1 | Alignment
---

Alignment <span class="text-muted">Challenge Problem 1</span>
=============================================================

Word alignment is a fundamental problem in machine translation. The input is 
a large _parallel text_ of translated sentences. For example, our
parallel text might contain the following translation:

    Das Parlament erhebt sich zu einer Schweigeminute .  
    The House rose and observed a minute 's silence .

Your task is to align the words of the sentences. Given the sentence pair
above, you would ideally produce an alignment containing these pairs:

| German            | | English                |
|-------------------|-|------------------------|
| 0. Das            | | 0. The                 |
| 1. Parlament      | | 1. House               |
| 2. erhebt         | | 2. rose                |
| 3. sich           | | 2. rose                |
| 5. einer          | | 5. a                   |
| 6. Schweigeminute | | 6. minute              |
| 6. Schweigeminute | | 7. 's                  |
| 6. Schweigeminute | | 8. silence             |
| 7. .              | | 9. .                   |

\\
Notice that some words, like _observed_, are not aligned, while other words,
like <em>rose</em>, are multiply aligned. In some cases, the aligned words will not
even be in the same order. In other cases, bilingual speakers won't even 
agree on the correct alignment of a translation---for example, some 
might align _zu_ to _rose_. These phenomena make word alignment challenging.

Getting started
---------------

You must have `git` and `python2.7`, both available on dice. To get the code
and data, run:

    git clone https://github.com/alopez/dreamt.git

In the new `dreamt/aligner` directory you will find a python program 
called `align.py`, which contains a complete but very simple alignment 
algorithm based on the intuition that English and German words which 
frequently appear together are likely to be translations of each other. 
So for every word, 
our aligner computes the set of sentences that the word appears in. Then 
for every pair of English and German words, it computes the similarity of 
their corresponding sets, and it aligns those pairs with similarity higher 
than a threshold. The aligner computes sentence similarity with 
[Dice's coefficient](http://en.wikipedia.org/wiki/Dice's_coefficient/). 
More formally, for an English word $$e$$ and French word $$f$$, let $$C(e)$$
and $$C(f)$$ count the number of sentences in which they (respectively) 
appear, and $$C(e,f)$$ count the number of sentences in which
they appear together. Dice's coefficient $$\delta(e,f)$$ is then:

<p>$$ \delta(e,f) = \frac{2 \times C(e,f)}{C(e) + C(f)} $$</p>

By default, the aligner guesses that $$e$$ and $$f$$ are aligned if
$$\delta(e,f) > 0.5$$. Run it on 200 sentences:

    python2.7 align.py -n 200 > dice200.out

For each sentence, it generates pairs of numbers on a line, one per 
alignment link. For our example alignment above, this would be:

    0-0 1-1 2-2 3-2 5-5 6-6 6-7 6-8 7-9

How accurate are the alignments we just produced? To find out, let's compare 
them with human alignments of the first 150 sentences. To account for 
the fact that humans don't always agree on word alignment, we
obtained alignments from two different annotators and distinguished two types
of alignments: _sure_ alignments are those that both annotators agreed on, 
while _possible_ alignments are those that were produced by one annotator,
but not the other. 

Our measures will include <em>precision</em>, the percentage of guessed alignments 
that are correct; _recall_, the percentage of human alignments that are 
guessed; and [_alignment error rate_](http://aclweb.org/anthology/P/P00/P00-1056.pdf)
(AER), which incorporates both precision and recall. For precision and recall, 
higher is better, while for AER, lower is better.
Possible alignments are counted for precision, but not
recall.

To compute precision, recall, and AER, run:


    python2.7 score-alignments.py < dice200.out

In addition to these numbers, you'll also see the guessed, sure, and probable
alignments. The guessed alignments are terrible! They're so noisy that it's
difficult to see what the human alignment look like. You can look at the 
human alignments without this noise by feeding empty alignments to
`score-alignments.py`:

    yes "" | head -150 | python2.7 score-alignments.py

__Question 1.__ What regularities do you observe in the human alignments?

Do you think the automatic alignments will improve if you use more data?
Try it:

    python2.7 align.py -n 2000 | python score-alignments.py -n 0

__Question 2.__ What happens when you use different amounts of data? Why do
you think this happens?

__Question 3.__ Vary the threshold using the <code>-t</code> option to <code>score-alignments.py</code>. 
How are precision, recall, and AER affected?

__Question 4.__ Do the automatic alignments reflect the regularities you
observed in the human alignments? If they don't, how do they differ?


Baseline: IBM Model 1
---------------------

Your challenge is to improve the alignment error rate as much as possible. 
It shouldn't be hard: while our intuition about the coocurrence of words 
makes our aligner better than chance, it is still very crude. Among other 
problems, you may have noticed that some words tend to spuriously align to 
many other words in the translation. Perhaps you could add some heuristics 
to the Dice aligner to solve this. We will use probability, which gives 
us powerful tools to reason about data. Specifically, we will use IBM 
Model 1, which you must implement, examine, and extend.
IBM Model 1 is a *word-to-word* or *lexical* translation model. 
It has only one set of parameters: a
conditional probability $$t(f|e)$$ for every foreign word 
$$f$$ and English word $$e$$ that cooccur in at least one sentence. 

Pick a translation pair from the data $${\cal D}$$. Let the German
sentence be $$\mathbf{f} = f_1 \ldots f_I$$, where $$f_i$$ is the
$$i$$th word of $$\mathbf{f}$$. The English
sentence is $$\mathbf{e} = e_1 \ldots e_J$$. Now we make a big 
assumption: that this pair of sentences exists
because someone translated the English into German following a simple
procedure. First, the translator chose the length of the German sentence based
on the length of the English sentence. Second, for each position in
the German sentence, the translator chose a word $$e$$ at random from
the English sentence, and then translated it to foreign word $$f$$
according to the translation probability $$t(f|e)$$. 

This means that we can represent an alignment $$\textbf{a}$$
of the foreign words by: $$\textbf{a} = a_1 \ldots a_I$$. There
is one random variable $$a_i$$ for each foreign word $$f_i$$, and
each $$a_i$$ must take value in the range $$1,...,J$$, 
indicating the chosen English word.[^1] Hence we obtain the following
model:

[^1]: In some versions of this story, $$a_i$$ is allowed to be $$0$$, which means the translator plucked the German word from thin air rather than translating any English word. We call this *null* alignment.

<p>$$ \Pr(\mathbf{f}, \textbf{a}|\mathbf{e}) = \ell(I|J) \prod_{i=1}^I
c(a_i|J) \cdot t(f_i|e_{a_i})$$</p>

Here, parameter $$\ell(I|J)$$ is the probability of producing
a German sentence of length $$I$$ from an English sentence of length
$$J$$, and $$c(a_i=j|J)$$ is the probability of choosing the $$j$$th
English word. We have assumed that English words are chosen at random 
from a uniform distribution, so $$c(a_i=j|J) = \frac{1}{J}$$.
Take for example a three-word German sentence ($$I = 3$$) aligned
to a three-word English sentence ($$J = 3$$).  If the alignment
$$\textbf{a} = (1,3,2)$$, that is, $$a_1 = 1$$, $$a_2 = 3$$, and
$$a_3 = 2$$ we can write the probability of this sentence pair 
with this alignment as:

<p>$$\Pr(\mathbf{f}, \textbf{a}|\mathbf{e}) = \ell(3|3) 
\cdot \left(\frac{1}{3}\right)^3 t(f_1|e_1)
\cdot t(f_2|e_3) \cdot t(f_3|e_2)$$</p>

If you are familiar with sequence tagging problems---like part-of-speech
tagging---you might recognize this as a tagging problem in disguise:
the goal is simply to tag each German word with an English word! The model
we've just described is a zero-order Markov model in which the tag of each
word depends only on the prior probability of the tag ($$c$$) and
the probability of choosing the word given the tag ($$t$$). But there are
some important differences. A major one is that, unlike in part-of-speech tagging, 
the set of tags changes in each training example!

A more important difference is that we aren't actually given any tagged
training examples, so we can't use supervised learning. So how do we set
parameters $$t(f_3|e_2)$$? Let's think about this for a second. We 
have some data that we assume was generated by our model, and the
probability of this data under the model is a function of the parameters
(called the _likelihood_). So a reasonable solution is to choose 
parameters that maximize this function---the _maximum likelihood_
estimate. If we have $$S$$ sentence pairs, where $$\mathbf{f}^{(s)}$$,
$$\mathbf{e}^{(s)}$$, and $$\mathbf{a}^{(s)}$$ are the German, English,
and alignment variables of the $$s$$th pair (respectively), we want
to choose parameters $$\hat{t}$$ according to this _objective function_:

<p>$$ \hat{t} = \arg\max_{t} \sum_{s=1}^S \log \Pr(\mathbf{f}^{(s)},\mathbf{a}^{(s)}|\mathbf{e}^{(s)}, t) $$</p>

You will notice a small problem with this scheme: we weren't given the
alignments, which are needed to compute the maximum likelihood estimate.
The alignment $$\mathbf{a}$$ is a _latent variable_ (or, if you're feeling
less charitable, a _nuisance variable_). But we can still
maximize the data likelihood---we just maximize the probability of the
_observed_ data by summing over all possible values of the latent variables.

<p>$$
\begin{eqnarray*}
\Pr(\mathbf{f}|\mathbf{e}, t) &amp; = &amp; \sum_{\textbf{a}} \Pr(\mathbf{f}, \textbf{a}|\mathbf{e}, t) \\
&amp; = &amp; \sum_{a_1=1}^J \cdots \sum_{a_I=1}^J  \ell(I|J) \prod_{i=1}^I c(a_i|J) \cdot t(f_i|e_{a_i}) \\
&& \textrm{(this computes all possible alignments)} \\
& = & C \prod_{i=1}^I \sum_{j=1}^J t(f_i|e_j) \\
&& \textrm{(since the } \ell \textrm{ and } c \textrm{ terms don't depend on the alignment, we lump them into a constant } C)
\end{eqnarray*}
$$</p>

To perform this optimization we will use the _Expectation-Maximization_
(EM) algorithm. A full explanation of this algorithm is lenghy, 
but I expect you to understand what it does and how it works at least at
the level described in your textbook and in 
[this note](http://edin.ac/1DH1C78). In particular,
you should be able to derive EM update equations for simple models, though
I won't ask you to prove anything about it.

In order to estimate the parameters $$t(\cdot|\cdot)$$
we start with an initial estimate $$t_0$$ and modify it iteratively
to get $$t_1, t_2, \ldots$$. The parameter updates are derived
for each German word $$f_i$$ and English word $$e_j$$ as follows:

<p>$$t_k(f_i|e_j) = \sum_{s=1}^N \sum_{(f_i, e_j) \in (\textbf{f}^{(s)}, \textbf{e}^{(s)})} \frac{ \mathbb{E}_{t_{k-1}}(f_i, e_j, \textbf{f}^{(s)}, \textbf{e}^{(s)}) }{\mathbb{E}_{t_{k-1}} (e_j, \textbf{f}^{(s)}, \textbf{e}^{(s)}) }$$</p>

These counts are *expected counts* over all possible alignments,
and each alignment has a probability computed using $$t_{k-1}$$.

Once we have trained an alignment model, we want to find
the most probable  alignment for a given translation pair.

<p>$$ 
\hat{\textbf{a}} = \arg\max_{\textbf{a}} \Pr(\textbf{a}|\textbf{e}, \textbf{f})
$$</p>

The best alignment to a target sentence in our simple baseline model
is obtained by finding the best alignment for each word in
the source sentence independently of the other words. For each
foreign word $$f_i$$ in the source sentence the best alignment is
given by:

<p>$$ 
\hat{a_i} = \arg\max_{a_i} t(f_i|e_{a_i}) 
$$</p>

If your Model 1 implementation is correct, it will give you
much better alignments than the Dice aligner. They are still far
from perfect, though. 

I strongly recommend that you work through the derivation of the full
algorithm from first principles, described in much more detail in 
[this note](http://edin.ac/1DH1C78). This is to practice writing
down a model, deriving simple estimators from the model, and implementing
those estimators as code. On the exam I am very likely to ask you to
derive updates for a new model. I will not ask you to recite the specific
form of Model 1.

You are required to implement IBM Model 1. This requires
a very modest amount of code: my implementation is one line shorter than the 
Dice aligner! You are not required to use the python baseline (though that
will be easiest). Use whatever language you prefer.

How will you know if your implementation is correct? One way is to sanity
check it by comparing with the results of a known reference implementation. I
implemented Model 1 according to [this note](http://edin.ac/1DH1C78). My
implementation:

* Learns a model of German sentences conditioned on English sentences.
* Does not model null alignments---every German word must align to exactly one English word.
* Initializes word translation probabilities uniformly.
* Is about the same length as the Dice aligner (38 LOC).

When I run this implementation for 5 iterations on the full data, I get 
these results:

    Precision = 0.545814
    Recall = 0.588443
    AER = 0.434428

Don't worry about reproducing these numbers exactly. If your results differ
after three or four decimal places, you probably have a correct implementation.
Welcome to the joy of programming with stochastic models. (For some real joy,
try debugging stochastic algorithms, like Monte Carlo algorithms).

__Question 5.__ The theory behind expectation maximization guarantees that
the data likelihood increases after each iteration. Plot the data likelihood
for some amount of data over ten iterations. You will need to do this 
in logspace to avoid numerical underflow. How does likelihood change? Does 
it appear to converge? Now compare this with the AER of the model after
each iteration. How do data likelihood and AER relate?

__Question 6.__: Choose two English words from the data: a very frequent word,
and an infrequent word. Plot their translation distributions. What do you 
observe? How do the shapes of these distributions affect the resulting 
alignments?

__Question 7.__: Choose two English words from the data that are morphological
variants of each other (e.g. _house_ and _houses_) and plot their translation
distributions. Why are they similar or different? Does this seem sensible?

__Question 8.__: Do your Model 1 alignments reflect the regularities you
observed in the human alignments? If they don't, how do they differ?


The Challenge
-------------

__Question 9.__ Developing a Model 1 aligner and answering questions 1 through 8 
will help you understand alignment. But if you answered the questions in full
you will have noticed some serious problems with Model 1. By thinking
carefully about these problems, you should be able to develop better 
probabilistic models of alignment. How well can you do?
To really understand alignment, you must experiment with at least one 
well-motivated extension of Model 1 and report the results. Your report
should concisely state the problem you tried to solve, how you modeled it, and
the results you obtained on the data. You are **not required** to do better than 
Model 1, only to hypothesize and experiment in good faith. Failed experiments
are a necessary component of good problem-solving.

Here are some ideas:

* Implement [a model that prefers to align words close to the diagonal](http://aclweb.org/anthology/N/N13/N13-1073.pdf).
* Implement an [HMM alignment model](http://aclweb.org/anthology-new/C/C96/C96-2141.pdf).
* Implement [a morphologically-aware alignment model](http://aclweb.org/anthology/N/N13/N13-1140.pdf).
* [Use *maximum a posteriori* inference under a Bayesian prior](http://aclweb.org/anthology/P/P11/P11-2032.pdf).
* Train a German-English model and an English-German model and [combine their predictions](http://aclweb.org/anthology-new/N/N06/N06-1014.pdf).
* Train a [supervised discriminative alignment model](http://aclweb.org/anthology-new/P/P06/P06-1009.pdf) on the annotated development set.
* Train an [unsupervised discriminative alignment model](http://aclweb.org/anthology-new/P/P11/P11-1042.pdf).
* Seek out additional [inspiration](http://scholar.google.com/scholar?q=word+alignment).

But the sky's the limit! Alignment is not a solved problem, and you are 
welcome to invent your own solution.

Resources
---------

In completing this challenge, you may want to consult other descriptions of
the material if they are clearer to you. Here are a few:

* [My notes on Model 1](http://edin.ac/1DH1C78)
* [Statistical Machine Translation: IBM Models 1 and 2](http://www.cs.columbia.edu/~mcollins/courses/nlp2011/notes/ibm12.pdf), Collins.
* [Statistical MT Tutorial Workbook](http://www.isi.edu/natural-language/mt/wkbk.rtf), Knight.
* [Model 1 EM example](http://people.cs.umass.edu/~brenocon/inlp2014/notes/model1_em.html), O'Connor.

### Acknowledgements

This challenge was developed in collaboration with
[Chris Callison-Burch](http://www.cis.upenn.edu/~ccb/),
[John DeNero](http://www.denero.org/),
[Chris Dyer](http://www.cs.cmu.edu/~cdyer),
[Philipp Koehn](http://homepages.inf.ed.ac.uk/pkoehn/),
[Matt Post](http://cs.jhu.edu/~post/), and
[Anoop Sarkar](http://www.cs.sfu.ca/~anoop/).


### Footnotes 
