---
layout: post
title:  Transformer
date:   2023-05-15 14:28:00
description: Transformer explained
tags: notes math
categories: note-posts
---
#### Transformer explained
Check out the great illustration of  <a href="https://jalammar.github.io/illustrated-transformer/">transformer</a>.

#### Cost formulation background
<ul>
    <li><a href="https://colah.github.io/posts/2015-09-Visual-Information/">Entropy</a>: coding length as a system intrinsic. $$ H(p)=\sum_{x}p(x)\log_2(\frac{1}{p(x)}) $$</li>
    <li><p><a href="https://colah.github.io/posts/2015-09-Visual-Information/">Cross entropy</a>: coding length for a message (sample) p as if it is drawn from distribution q. $$H_q(p)=\sum_{x}p(x)\log_2(\frac{1}{q(x)})$$</p></li>
    <li><a href="https://www.countbayesie.com/blog/2017/5/9/kullback-leibler-divergence-explained">Lullback-Lerbler divergence</a>: coding efficiency difference based on true message (sample) distribution p (Zero-rebased cross entropy). $$D_q(p)=H_q(p) - H(p) = \sum_{x}p(x)\log_2(\frac{p(x)}{q(x)})$$</li>
</ul>

<hr>

#### Transformer practice
Pytorch version <a href="http://nlp.seas.harvard.edu/annotated-transformer/">The Annotated Transformer</a>.

Open <a href="https://colab.research.google.com/github/lin-ht/annotated-transformer/blob/master/AnnotatedTransformer.ipynb"> my forked playground in colab.</a>

Github of <a href="https://github.com/lucidrains/x-transformers"> x-transformers </a> by <a href="https://github.com/lucidrains?tab=repositories">lucidrains</a>.
