---
layout: post
title:  Deploy jekyll github homepage
date:   2023-05-11 10:47:00
description: Command cheatsheet
tags: notes code
categories: note-posts
---
#### Installation cheatsheet
Install chruby:
```bash
which ruby # Old ruby

brew install chruby
brew info chruby # Check its information

# https://github.com/postmodern/chruby/wiki/Ruby
brew install automake bison openssl readline libyaml gdbm libffi

brew tap raggi/ale
# macOS users must update their OpenSSL CA cert bundle
# https://github.com/raggi/openssl-osx-ca#readme
brew install openssl-osx-ca

brew services start openssl-osx-ca

# Caveat: restart raggi/ale/openssl-osx-ca after an upgrade
brew services restart raggi/ale/openssl-osx-ca
```

Install Ruby:
```bash
curl --remote-name https://cache.ruby-lang.org/pub/ruby/3.2/ruby-3.2.0.tar.xz
tar -xJvf ruby-3.2.0.tar.xz
cd ruby-3.2.0
./configure --prefix="$HOME/.rubies/ruby-3.2.0" --with-opt-dir="$(brew --prefix openssl):$(brew --prefix readline):$(brew --prefix libyaml):$(brew --prefix gdbm):$(brew --prefix libffi)"
make -j4
make install
```

Switch Ruby:
```bash
chruby # List ruby
chruby 3.2.0 # Select ruby
```

Update gem:
```bash
gem update --system 3.4.13
```

Install Imagemagick:
```bash
brew install imagemagick
```

Configure:
```bash
# To solve openssl ssl error:
export SSL_CERT_DIR=/Users/halin/mambaforge/ssl
export SSL_CERT_FILE=/Users/halin/mambaforge/ssl/cacert.pem

# Set repository
export PAGES_REPO_NWO="lin-ht/lin-ht.github.io"
```

Personal github page set up:
```bash
cd <path-to-the-repo-lin-ht.github.io>

# deploy the page
bin/deploy --user

# start local server
bundle install
bundle exec jekyll serve
```

To resolve <a href="">baseurl error while deployed on github</a>, use empty string `baseurl: ''` in `_config.yml`.


#### New terminal configuration cheatsheet

When open a new terminal, remember to chruby:
```bash
chruby # List ruby
chruby 3.2.0 # Switch ruby
ruby -v # check the current ruby version

# configuration
export SSL_CERT_DIR=/Users/halin/mambaforge/ssl
export SSL_CERT_FILE=/Users/halin/mambaforge/ssl/cacert.pem
export PAGES_REPO_NWO="lin-ht/lin-ht.github.io"

bundle exec jekyll serve
```

#### Command cheatsheet

```bash
git diff
git add --all 
git commit -m "Post update" 
git push -u origin customized 
bin/deploy --user
bundle exec jekyll serve --lsi
```

#### git commands

```bash
git branch # List local branches 
git branch -r # List remote branches 
git branch -a # List local and remote branches

git checkout my-branch-name
```


#### References
<ul>
    <li><a href="https://docs.github.com/en/get-started/quickstart/fork-a-repo">Fork a repo.</a></li>
</ul>
