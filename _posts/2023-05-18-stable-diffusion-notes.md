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
        {% include figure.html path="assets/img/posts/unet_architecture.jpeg" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>

<div class="caption">
    Figure. UNet architecutre with residual connections.
</div>

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