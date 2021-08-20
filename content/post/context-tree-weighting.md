---
title: 'Context Tree Weighting and Compression'
date: 2021-04-23 
tags: ["algorithms", "compression", "context tree weighting"]
categories: ["algorithms", "compression"]
draft: false
katex: true
---

In this post I go over the basics of data compression with arithmetic coding and describe the Context Tree Weighting algorithm. At the end I discuss implementation and some experimental results.

## Information Theory Review
First a quick review of information theory.
Suppose that we receive some data $x$ drawn from a discrete random variable $X$, whose probability distribution is $P$. Then the Shannon information content of $x$ is defined as 

$$I(x) = -\log(P(x)).$$

Why is this a good measure of information? Basically, it formalizes the idea that less probable events yield more information, and conversely predictable events yield less information.  

Given an incomplete message like ``Hello Worl`` you'd assign a high probability (say 95%) that the next character is "_d_". Finding out that the next character is indeed "_d_" would only give you about 0.07 units of information (also called shannons), since you in a sense already knew that.  

The Shannon entropy is defined as the expected value of information $$H(X) = \sum_x -P(x)\log(P(x)).$$
It can be thought of as a measure of how predictable the data is on average.
Now suppose we want to compress a stream of data.  
Formally, we want a code $C: \mathcal{A} \to \text{ \\{0, 1\\} } ^* $ where $\mathcal{A}$ is the alphabet the data comes from (e.g. ASCII characters) and $\text{\\{0, 1\\} }^*$ is the set of all binary strings. A good code should have two properties: 

* $C$ has an inverse (compression is lossless)
* The codes $C(x)$ have minimal average length (good compression ratio).

Shannon proved that such an optimal code would have an average code length essentially _equal to the entropy_ of the source.   
Moreover, he proved that for an optimal code, the code length $\mid C(x) \mid$ would be _equal to the information content_ $I(x)$.  In other words, if we define the _redundancy_ of our code $C(x)$ as the difference $\rho(x) = \mid C(x)\mid - I(x)$, then finding a good code is equivalent to minimizing the redundancy.  
Of course this is a bit sloppy; for more precise statements see e.g. [[2]](#references)

## Arithmetic Coding
Unsurprisingly, Shannon's proof is non-constructive, so how do we make an optimal code in practice? Several algorithms have been developed. You make have heard of the Huffman code or Lempel-Ziv algorithm, which one can prove are roughly optimal. Less commonly known is _arithmetic coding_. 

The nice thing about arithmetic coding is you can plug in any probabilistic model $\mathcal{M}$ of the source (i.e. a way to generate predictions for the next symbol) and it guarantees you a code length approximately equal to the entropy according to your model $H(X \mid \mathcal{M})$. The idea is to associate each sequence to a subinterval of $[0,1)$, whose length is equal to its probability.


To illustrate the algorithm, suppose we are recieving a stream of binary data $x_1,x_2,\dots,x_N$ (abbreviated $x_1^N$). Let $L$ be the lower endpoint of the interval and $U$ be the upper endpoint after receiving the $n$th symbol. Initially we have received no bits and set $L=0$, $U=1$.   

```bash
L, U ⟵ 0, 1
For n = 1...N do  
    generate prediction P(xn = 0)
        if xn = 1
            then L ⟵ L +  P(xn = 0)*(U-L)
        else xn = 0 and U ⟵ U-P(xn = 1)*(U-L)
        endif
endfor
```

At each step, we divide the current interval according to the probabilities of the next symbol. At the end, the compressed sequence is the sequence of binary digits of a number chosen from the interval $[L,U)$, (e.g. $L$ rounded up). 

After $n$ steps there are $2^n$ subintervals, corresponding to the possible sequences $x_1...x_n$. For example after $n=3$ the intervals might look like this: 

{{< figure src="/images/coding.png#center" size="900x400" >}}

Note that the length of the intervals are $P(x_1\dots x_n) = P(x_1^n)$. A trivial model that predicts $P(x_1^n) = 2^{-n}$ would essentially give us back the original sequence (a terrible compressor indeed) -- ideally our model assigns a larger probability to the sequence.

That interval is guaranteed to contain a rational number $c$ with approximately $-\log( P(x_1^n))$ digits in its binary expansion, reducing the number of bits needed to describe it. The encoder can end the message either with a special `EOT` symbol or by transmitting the length.

Given the code $c$ and access to the same model $\mathcal{M}$, the decoder can sequentially deduce whether the $n$th bit was a $0$ or $1$ by comparing $c$ to the divider $L + P(x_n = 0)\cdot(U-L)$ and hence uniquely decode the compressed data. 

Arithmetic coding works the same way for non-binary alphabets, at each step dividing the interval into $\mid\mathcal{A}\mid$ sections according to their probabilites. 

## Models
Assuming arithmetic coding can be implemented efficiently (see [below](#implementation-and-experiments)), we have reduced compression to finding a good model of the data. Of course, the model will depend a lot on what type of data you're compressing and your speed/memory goals.

The simplest model could just use frequency counts of the different symbols to make predictions. If you wanted to go all out you could train a neural net like an RNN or [transformer](https://en.wikipedia.org/wiki/Transformer_(machine_learning_model)) on your data to make the predictions. However you'd have to account for the size of the weights, which have to be sent the decoder. It would also be too slow for most purposes (although lightweight NN's [achieve reasonable speed](http://mattmahoney.net/dc/mmahoney00.pdf)). Clearly there is a tradeoff between the complexity of the model and the compression ratio.

A simpler option is a _tree model_. We assume that $P(x_n)$ depends on at most $D$ of the previous symbols (in probability lingo, a $D$-Markov model). 

The main idea is to build a suffix tree (or trie) from the source. Given past symbols $x_{n-D},...,x_{n-1}$ (also known as the _context_) we take the last $k \le D$ as the suffix. We are free to choose $k$, and it can vary between contexts.   We then record the frequencies of each symbol we observe after seeing a suffix. 

For example if the alphabet is $\mathcal{A} = \text{ \\{a ,b ,n\\}}$ and the sequence "$\text{bananan}$" here is one possible depth $D=2$ tree (suffixes are read from bottom up)

{{< figure src="/images/path12.png#center" size="1024x900" caption="A 2-Markov Model">}}

Each leaf has three counters (# $\text{a}$'s, # $\text{b}$'s , #$\text{n}$'s). E.g. "$\text{n}$" appears two times after the suffix "$\text{na}$". A tree like this is called a _prediction suffix tree_ (PST). We can make rough estimates of the probability using the statistics stored in the leaves with e.g. a [Beta distribution](https://en.wikipedia.org/wiki/Beta_distribution). Another common choice is the [Krichevsky-Trofimov estimator](https://en.wikipedia.org/wiki/Krichevsky–Trofimov_estimator).

A similar idea combined with the Lempel-Ziv algorithm forms the basis for the "Prediction by Partial Matching" algorithm, which is considered one of the top-performing tree models [[3]](#references).


## Context Tree Weighting
Context Tree Weighting (CTW) is a beautiful algorithm invented by Willems, Shtarkov, and Tjalkens [[1]](#references) that efficiently computes a weighted sum over all prediction suffix trees of depth $D$ for a binary alphabet. Naively, this would take $O(2^D)$, while CTW computes it in $O(D)$. 

We first build a complete binary tree of depth $D$, called the context tree. Like a prediction suffix tree, each node is labeled by its corresponding suffix $s$. Each node stores the frequencies of 0's and 1's that occur after suffix $s$, and computes a probability $P^s$ recursively from its children as follows: Let $x_1\dots x_n$ be the past symbols, then the probability stored at the node $s$ is defined as

$$
P^s = \begin{cases}P_e(x_{1}^n) & \text{ if } s \text{ is a leaf }\cr
	\frac{1}{2}\left(P_e(x_{1}^n) + P^{s0}P^{s1}\right)& \text{ otherwise.} \end{cases} 
$$

Here $P_e$ is a probability estimate based on the frequencies stored at $s$ (in the original paper it is the [KT estimator](https://en.wikipedia.org/wiki/Krichevsky–Trofimov_estimator)) and $s0$, $s1$ denote the children of $s$. 

So we just average the frequency estimate with the predictions of the children nodes. Simple, right? But what is the probability we get at the root node, corresponding to the empty suffix $\epsilon$? Brace for notation... it is

$$
P^{\epsilon} = \sum_{T \in \mathcal{M}_D}2^{-\Gamma_D(T)}P(x_1^n \mid T) 
$$

where 
* $\mathcal{M}_D$ is the set of all PST's of depth at most $D$ 
* $P(x_1^n \mid T)$ is the probability of $x_1\dots x_n$ according to the PST $T$
* $\Gamma_D(T)$ is the number of nodes of $T$ minus the number of leaves at depth $D$

Well we certainly have a weighted sum of PST predictions, but what is this $\Gamma_D$? It is actually the length of the optimal [prefix code](https://en.wikipedia.org/wiki/Prefix_code) for the tree $T$, i.e. the number of bits needed to describe $T$. Thus each each prediction suffix tree is weighted by an Occam's razor-like penalty. 

To make predictions with CTW we use the root probability, then update the path in the tree corresponding to the context when we receive the next symbol. The original paper proves a sharp bound on the redundancy of the code computed by CTW (Theorem 2).

### Sidenote: Compression = AGI?
Anyone into data compression probably knows about the [Hutter prize](http://prize.hutter1.net/), a cash prize for compressing the first GB of Wikipedia to smaller than the current record (116 MB). Hutter's slogan is "Compression = AGI," a (rather exaggerated) summary of the principle behind [AIXI](https://en.wikipedia.org/wiki/AIXI). 

Roughly speaking, AIXI solves the general reinforcement learning problem by weighing models of the environment by both the expected reward _and_ a measure of the model's complexity. In particular, the complexity is measured by the [Kolmogorov complexity](https://en.wikipedia.org/wiki/Kolmogorov_complexity) of the observation-reward sequence. (Hutter gives a fairly good non-technical explanation in [this interview](https://www.youtube.com/watch?v=E1AxVXt2Gv4)).

Although theoretically optimal, the AIXI action function is infeasible to compute directly.  
What is so interesting about CTW is that the probability it computes is a mixture of models weighted by complexity (for a restricted set of models) _and_ it is efficient to compute. For this reason Veness et. al [[5](#references)] used CTW in their AIXI approximation algorithm. Other mixture models have also become popular in data compression algorithms (You can for example run several probability models in parallel).

### CTW Extensions
The binary CTW can be extended to non-binary alphabets by replacing $P^{s0}P^{s1}$ by a product over all children of $s$ in the recursive formula. However the straightforward method of making direct predictions hasn't had much empirical success, especially with large alphabets. 

In practice, CTW is good at making binary predictions, although it can still have non-binary contexts. How can we convert binary predictions to general symbol predictions?

The obvious way to do this is to first choose a binary code for the alphabet $\mathcal{A}$, then predict each bit. This is the idea behind a "decomposition tree," which is basically a binary search tree. The leaves of the tree are the elements of $\mathcal{A}$, and each internal node has a context tree (a tree of trees) whose job is to predict whether a symbol is in the left or right subtree of the node. 

The probability of a symbol $a \in \mathcal{A}$ is then calculated as the product of the probabilities on the path from the root to the leaf $a$. 
For example, here is a possible decomposition tree for $\mathcal{A} = \text{ \\{b, a, n\\}}$.

{{< figure src="/images/ctw.png#center" width="450" caption="A decomposition tree with two context trees">}}

Context Tree 1 predicts whether the next symbol will be $\text{b}$ or not, and context tree 2 decides between $\text{a}$ and $\text{n}$.

One drawback of this approach is that the performance is sensitive to the topology of the decomposition tree. One choice is [the Huffman tree](https://en.wikipedia.org/wiki/Huffman_coding), which minimizes the number of context trees updates that are required.    

In general we want to find groupings of the symbols which are "similar," in some way. We could for example collect statistics on co-occurence of symbols à la [GloVe](https://nlp.stanford.edu/projects/glove/). Once we decide on a decomposition tree, we have to describe it to the decompressor, but this is only a few more bytes overhead.


## Implementation and Experiments
To experiment I wrote a [simple implementation](https://github.com/meiji163/ctwz) of CTW using a Huffman decomposition tree. 

The main difficulty in implementation is the precision of the probabilities. The original form of the arithmetic encoder requires arbitrary precision. To make it practical we can use finite precision and output a bit once the leading bits of $U$ and $L$ are equal. Then we scale the whole interval by $2$. A more clever version of this scaling was invented by Witten, Neal, and Cleary [[6]](#references), which is the version I used.

The original CTW algorithm also needs modification. The main technique is to store a ratio of probabilities in the nodes to handle float-point errors. The exact method I used is the one described in [[4]](#references).

I tested my implementation with depth $D=12$ on some text data from the [Canterbury corpus](https://www.corpus.canterbury.ac.nz/). For comparison I also compressed the files with Unix's "compress" and gzip on default settings. Both use variants of the Lempel-Ziv algorithm. 

|   File   |  Size (Bytes)    |   Description
| :----------: | :----------: | :-----:
| grammar.lsp |  3721 | LISP code
|  fields.c |  11150 | C code
| plrabn12.txt | 481861 | Milton's Paradise Lost 
| bible.txt | 4047392 | King James Bible
| E.coli	| 4638690 | Genome of E.Coli


| Compression Ratio  | ctw  |  compress  | gzip 
| :---: | :----:  | :-----:| :----: | :---: |  :------: |
| fields.c | 4.21 | 2.24 | 3.55
| grammar.lsp | 3.65 | 2.05 | 2.99
| plrabn12.txt | 3.15 | 2.37 |2.48
| bible.txt |   4.50 | 3.39 | 3.39 |
| E.coli |  4.09 | 3.69  | 3.56  

It is interesting to look at the predictions CTW makes. It is particularly good at detecting basic structure like spaces between words, braces, parenthesis, etc. and repeated words and phrases. For example in fields.c consistently predicts ( $p \approx 0.8$) that function declarations like `realloc ()` are followed by `;` and that comments starting with `/*` end with `*/`. 

In `bible.txt`, it is particularly good at predicting common words like "God" (big surprise). The probabilities for E.coli are actually rarely greater than $0.3$. Since $\mathcal{A} = \text{\\{g, c, t, a\\}}$ , the compression in this case is mainly due to the fact that the alphabet is small. 

We can see CTW is basically finding the "low hanging fruits" of redundancy. This is about all we can expect, since we know Markov models aren't particularly good at learning grammatical structures.

Although CTW compares favorably on these text files, my implementation failed miserably when I tried it on binary data (possible due to my janky code). My implementation is also 5-10 times slower than gzip and compress on large files, and undoubtedly uses much more memory. 

The good news is there is a lot of room for improvement. Obvious optimizations would be parallelizing the context trees and using fixed-point arithmetic instead of doubles. I highly recommend Volf's thesis [[4]](#references), in which he documents his CTW compressor project in great detail. He uses the previously mentioned optimizations (and many more) to satisfy a memory limit of 32MB and a compression speed of around 10kB/s. 

## Conclusion
Context Tree Weighting is a beautiful example of a mixture model with "Occam's-Razor" weighting. Its solid theoretical foundations make it attractive compared to e.g. Prediction by Partial Matching. However it is trickier to implement because of greater memory and computational requirements.

## References
[[1]](https://www.cs.cmu.edu/~aarti/Class/10704_Spring15/CTW.pdf) Willems, Shtarkov, Tjalkens "_The Context-Tree Weighting Method: Basic Properties_" 1995 

[[2]](https://web.cs.iastate.edu/~honavar/infotheorybook.pdf) D. MacKay, "_Information Theory, Inference, and Learning Algorithms_" 2003  

[[3]](https://www.jair.org/index.php/jair/article/view/10394) Begleiter, El-Yaniv, Yona, "_On Prediction Using Variable Order Markov Models_" 2004  

[[4]](https://pure.tue.nl/ws/files/2043425/200213835.pdf) P. Volf, "_Weighting techniques in data compression: theory and algorithms_" 2002  

[[5]](https://www.aaai.org/Papers/JAIR/Vol40/JAIR-4004.pdf) Veness, Siong Ng, Hutter, Uther, Silver "_A Monte-Carlo AIXI Approximation_" 2011  

[[6]](https://dl.acm.org/doi/10.1145/214762.214771) Witten, Neal, Cleary "_Arithmetic Coding for Data Compression_" 1987

