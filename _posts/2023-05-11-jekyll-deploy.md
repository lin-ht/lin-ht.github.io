---
layout: post
title:  Deploy jekyll github homepage
date:   2023-05-11 10:47:00
description: Command cheatsheet
tags: notes code
categories: note-posts
---
#### Command cheatsheet

{% highlight terminal %}
git diff
git add --all 
git commit -m "Post update" 
git push -u origin customized 
bin/deploy --user
bundle exec jekyll serve --lsi
{% endhighlight %}

#### git commands

{% highlight terminal %}
git branch # List local branches 
git branch -r # List remote branches 
git branch -a # List local and remote branches

git checkout my-branch-name
{% endhighlight %}


#### References
<ul>
    <li><a href="https://docs.github.com/en/get-started/quickstart/fork-a-repo">Fork a repo.</a></li>
</ul>
