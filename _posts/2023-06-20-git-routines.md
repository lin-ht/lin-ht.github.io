---
layout: post
title:  Git Routine
date:   2023-06-20 10:00:00
description: Routine git commands
tags: notes code
categories: note-posts
---
#### Useful git commands
<a href="https://git-scm.com/book/en/v2/Git-Tools-Submodules">Git Tools - Submodules</a>
{% highlight terminal %}
git clone --recurse-submodules -j8  target_git@github.com
{% endhighlight %}

If we cloned the repo without recursively pull submodules, try the following: 
{% highlight terminal %}
cd <repo-dir>
git submodule update --init --recursive
{% endhighlight %}

To individually update a specific submodule:
{% highlight terminal %}
cd <submodule-dir>
git pull --recurse-submodules
{% endhighlight %}

<a href="https://linuxhint.com/pull-git-submodules-after-cloning-project-from-github/">Submodule can be added</a> to your current repo by:
{% highlight terminal %}
cd <repo-dir>
git submodule add <git_url_to_the_submodule> <submodule_folder_name>
{% endhighlight %}

<a href="https://stackoverflow.com/questions/11168141/find-which-commit-is-currently-checked-out-in-git">Git Tools - commit information</a> variations:
{% highlight terminal %}
git show [--outline -s]
git log -1 [--outline] # show the last commit info
git status
{% endhighlight %}
Launch gitk graphic display:
{% highlight terminal %}
git bisect visualize
git bisect view  # shorter, means same thing
{% endhighlight %}

<a href="https://www.codeblocq.com/2016/02/Stash-your-changes-before-switching-branch/">Git Tools - Stash</a>:
{% highlight terminal %}
git stash save "changes on the current-branch"
git stash pop # unstash the changes of the top stash
git stash list # list the stash stack
git stash pop "stash@{1}" # unstash a specific stash
git stash apply # apply the top stash
git stash drop # drop the top stash
{% endhighlight %}

