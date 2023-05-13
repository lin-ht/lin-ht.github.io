---
layout: page
title: Photoshop Camera
description: Adobe Photoshop Camera is a free photo editor camera app that lets you add the best filters and effects for your photos.
img: assets/img/projects/psc/psc-icon.png
importance: 1
category: work
---

Adobe Photoshop Camera <a href="https://www.adobe.com/products/photoshop-camera.html">(Psc)</a> is powered by the rendering pipeline used in Photoshop camera. I reimplemented the entire pipeline in GPU and optimized its performance. This GPU implementation was further used as the back end rendering module for machine learning base auto-tone. With this, I was able to bring the auto-tone to Psc too.

Besides the pipeline, I worked on variant lens effects. The most prominent one is lens blur effect in potrait lens. We have developed our own lens blur rendering techniques that handle both foreground and background blur correctly and efficiently.


<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/projects/psc/psc-autotone.jpg" title="auto tone" class="img-fluid rounded z-depth-1" %}
    </div>
</div>