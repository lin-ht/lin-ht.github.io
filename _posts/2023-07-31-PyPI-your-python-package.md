---
layout: post
title:  PyPI your python package
date:   2023-07-31 12:00:00
description: tutorial
tags: notes
categories: note-posts
---
#### How to install, download and build Python wheels

Python Packaging Index (PyPI) is a repository containing several hundred thousand packages. 

##### Install package

{% highlight terminal %}
# Install a PyPl indexed package optionally with version v.v
pip install <packagename>[==v.v]

# test install with dry-run
pip install --dry-run <packagename>

# Upgrade an installed package
pip install --upgrade <packagename>

# Uninstall a package
pip uninstall <packagename>

# Install a package from a repository other than PyPI, such as Github
pip install -e git+<https://github.com/myrepo.git#egg=packagename>

# Install a package from specific index url
pip install <packagename> --index-url https://some.index.url

# Install a package with extra index url
pip install <packagename> --extra-index-url https://some.extra.index.url

# Install package from local .whl file
pip install /path/to/some/package.whl
{% endhighlight %}


##### Build your own .whl file

###### Pure python package

When it comes to Python packaging, if your package consists purely of Python code, you can do the following:

<b>1.</b> Make sure Wheel and the latest version of setuptools is installed on your system by running:

`python -m pip install -U wheel setuptools`

<b>2.</b> Then run:

`python setup.py sdist bdist_wheel`

This will create both a source distribution (sdist) and a wheel file (bdist_wheel) , along with all of its dependencies. You can now upload your built distributions to PyPI. For more information, see <a href="https://hynek.me/articles/sharing-your-labor-of-love-pypi-quick-and-dirty/">Sharing Your Labor of Love: PyPI Quick and Dirty</a>.

###### Python package with C libraries

If your package has linked C libraries, you’ll need to create specific build environments, and then compile your package separately for each target operating system you want to support. 

<b>1.</b> Wheel naming
```
{dist}-{version}(-{build})?-{python.version}-{os_platform}.whl
# deployment with Python 2.7 on 32 bit Windows example:
# PyYAML-5.3.1-cp27-cp27m-win32.whl
```

{% highlight terminal %}
# Install a PyPl indexed package optionally with version v.v
pip install <packagename>[==v.v]

{% endhighlight %}

#### References
<ul>
    <li><a href="https://www.activestate.com/resources/quick-reads/python-install-wheel/">How to install, download and build Python wheels</a>.</li>
    <li><a href="https://realpython.com/pypi-publish-python-package/#configuring-your-package">How to Publish an Open-Source Python Package to PyPI</a>.</li>
    <li><a href="https://realpython.com/python-wheels/">What Are Python Wheels and Why Should You Care?</a></li>
</ul>
