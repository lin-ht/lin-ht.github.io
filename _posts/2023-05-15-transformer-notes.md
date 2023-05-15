---
layout: post
title:  Transformer notes
date:   2023-05-15 14:28:00
description: transformer explained
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
Pytorch version <a href="http://nlp.seas.harvard.edu/2018/04/03/attention.html">The Annotated Transformer</a>.