There are three Python programs here (`-h` for usage):

 - `./rerank` chooses the best candidate translations from a k-best list using a linear model.
 - `./oracle` computes a lower bound of BLEU on the development data.
 - `./compute-bleu` computes the BLEU score of a set of translations.

The commands are designed to work in a pipeline. For instance, these are valid invocations:

    python rerank | python compute-bleu

    python oracle | python compute-bleu

The `data/` directory contains training, devlopment, and test data.

 - `train.src`: Russian source sentences.
 - `train.ref`: English reference sentences.
 - `train.100best`: Candidate translations of `train.src` from a machine translation system.
 - `dev+test.src`: Russian source sentences.
 - `dev.ref`: English references sentences for the first half of `dev+test.src`.
 - `dev+test.100best`: Candidate translations of `dev+test.src`.


