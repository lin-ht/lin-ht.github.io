---
layout: post
title:  Stable diffusion
date:   2023-05-18 09:30:00
description: Stable diffusion explained
tags: notes math
categories: note-posts
---
#### Stable diffusion explained

<a href="https://lilianweng.github.io/posts/2021-07-11-diffusion-models/">What are diffusion models</a>.

<a href="https://yang-song.net/blog/2021/score/">
Generative Modeling by Estimating Gradients of the Data Distribution</a>.


<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/posts/unet-architecture-fig.webp" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

<div class="caption">
    Figure 1. [<a href="https://betterprogramming.pub/diffusion-models-ddpms-ddims-and-classifier-free-guidance-e07b297b2869">Source</a>] UNet architecutre with residual connections and attention layers.
</div>


<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/posts/ddpm-training-sampling.webp" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

<div class="caption">
    Figure 2. [<a href="https://betterprogramming.pub/diffusion-models-ddpms-ddims-and-classifier-free-guidance-e07b297b2869">Source</a>] DDPM traing and sampling.
</div>

Diffusion process system information analysis:
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/posts/system-entropy.webp" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Figure 3. [<a href="https://betterprogramming.pub/diffusion-models-ddpms-ddims-and-classifier-free-guidance-e07b297b2869">Source</a>] System entropy indicates how much information the system contains, or the randamness of the system.
</div>

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/posts/diffusion-system-entropy-analysis.webp" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Figure 4. [<a href="https://betterprogramming.pub/diffusion-models-ddpms-ddims-and-classifier-free-guidance-e07b297b2869">Source</a>] Diffusion system entropy analysis on delta information change during one-step process.
</div>

Since $$ X^{(t)} $$ 
is from adding Gaussian noise to $$ X^{(t-1)} $$ and Gaussian distribution alteration maximizes the entropy increase, therefore $$ H_q(X^{(t)}|X^{(t-1)}) >= H_q(X^{(t-1)}|X^{(t)}) $$. In order words, $$ H_q(X^{(t-1)}|X^{(t)}) $$ has less randamness than the corresponding Gaussian noise.

“A lower bound on the entropy difference can be established by observing that additional steps in a Markov chain do not increase the information available about the initial state in the chain, and thus do not decrease the conditional entropy of the initial state.” (page 12 original paper)

For the lower bound, any information gain against the initial state, i.e. 
$$ H_q(X^{(t)}|X^{(0)}) - H_q(X^{(t-1)}|X^{(0)})$$, could possiblily be introduced during the last process from $$ t-1 $$ to $$ t $$, so when we remove that gained information entirely, we get the lower bound of the amount of the information.

Based on which, the improved DDPM paper decides to learn the variances to help improve the model's log-likelihood by representing the variances as an log domain interpolation between the upper and lower bounds.
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/posts/variances-interpolation.webp" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Figure 5. [<a href="https://betterprogramming.pub/diffusion-models-ddpms-ddims-and-classifier-free-guidance-e07b297b2869">Source</a>] Variances represented as a log domain interpolation.
</div>

The difference between distributions is a great way to model the variance because the variance at any step should model how much the distribution changes between timesteps.

These bounds are very useful to have the model estimate the variance at any timestep in the diffusion process. The improved DDPM paper notes that βₜ and βₜ~ represent two extremes on the variance (when the equal sign holds). One when the original image, x₀, is pure Gaussian, and the other when the original image, x₀, is a single value. Any input image x₀ will fall either between a pure Gaussian or a single-valued image, making it intuitive to interpolate between these two extremes.

#### Stable diffusion practice
The <a href="https://huggingface.co/blog/annotated-diffusion"> Annotated Diffusion Model </a>.

Github of <a href="https://github.com/lucidrains/denoising-diffusion-pytorch"> pytorch denoising diffusion </a> by <a href="https://github.com/lucidrains?tab=repositories">lucidrains</a>.

OpenAI <a href="https://github.com/openai/guided-diffusion">guided-diffusion</a>.


#### References

<ul>
    <li>Concept of variational lower bound (also called ELBO) in <a href="https://arxiv.org/abs/1312.6114">variational auto-encoder (VAE)</a> introduced by Kingma et al., 2013.</li>
    <li><a href="https://arxiv.org/abs/1505.04597">U-Net</a> introduced by Ronneberger et al., 2015.</li>
    <li><a href="http://karpathy.github.io/2019/04/25/recipe/">A Recipe for Training Neural Networks</a></li>
</ul>