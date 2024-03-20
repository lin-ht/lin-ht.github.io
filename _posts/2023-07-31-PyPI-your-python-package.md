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

```bash
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

pip install --find-links /path/to/the/wheel/file/ <package-name>

pip wheel [--no-deps] -w /path/to/the/wheel/files/
```


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

An example of wheel building:
```bash
# file structure:
setup.py
src/
    mypkg/
        __init__.py
        module.py
        data/
            tables.dat
            spoons.dat
            forks.dat 
```
Content of `setup.py`:

```python
from setuptools import setup

setup(name='foo',
      version='1.0',
      description='Python Distribution Example',
      author='lin',
      packages=['mypkg'],
      package_dir={'mypkg': 'src/mypkg'},
      package_data={'mypkg': ['data/*.dat']},
      )
```

Another `setup.py` for importing prebuilt external lib dependencies through a dummy package:
```bash
# file structure:
setup.py
cvcuda_lib/
    libcvcuda.so.0.3.1
    libcvcuda.so.0
    libcvcuda.so
    libnvcv_types.so.0.3.1
    libnvcv_types.so.0
    libnvcv_types.so
    cvcuda.cpython-38-x86_64-linux-gnu.so
    nvcv.cpython-38-x86_64-linux-gnu.so
```

```python
from setuptools import setup

setup(
    name='cvcuda_import',
    version='0.3.1',
    description='proxy wheel to bring in cvcuda Library',
    data_files=[('lib', ['cvcuda_lib/libcvcuda.so.0.3.1',
                         'cvcuda_lib/libcvcuda.so.0',
                         'cvcuda_lib/libcvcuda.so',
                         'cvcuda_lib/libnvcv_types.so.0.3.1',
                         'cvcuda_lib/libnvcv_types.so.0',
                         'cvcuda_lib/libnvcv_types.so']),
                ('lib/python3.8/site-packages',['cvcuda_lib/cvcuda.cpython-38-x86_64-linux-gnu.so',
                                                'cvcuda_lib/nvcv.cpython-38-x86_64-linux-gnu.so'])],

    install_requires=["numpy"],
 )
```


###### Upload your package to PyPI

<b>1.</b> Wheel naming is automatically generated from the wheel building
```
{dist}-{version}(-{build})?-{python.version}-{os_platform}.whl
# deployment with Python 2.7 on 32 bit Windows example:
# PyYAML-5.3.1-cp27-cp27m-win32.whl
```

<b>2.</b> Test local installation
```bash
pip install --find-links /path/to/the/wheel/file/ <package-name>
```

<b>3.</b> Upload to the index server

Get the publishing tool <a href="https://pypi.org/project/twine/">twine</a>  for this:
```bash
pip install -U twine
# Since both sdist and bdist_wheel output to dist/ by default, you can safely tell twine to upload everything under dist/ using a shell wildcard (dist/*).
twine upload dist/* [--repository-url https://$USER:$API_TOKEN@url.to.the.artifactory]
# if no url is provided, the default index server will be PyPI.
```


#### References
<ul>
    <li><a href="https://www.activestate.com/resources/quick-reads/python-install-wheel/">How to install, download and build Python wheels</a>.</li>
    <li><a href="https://realpython.com/pypi-publish-python-package/#configuring-your-package">How to Publish an Open-Source Python Package to PyPI</a>.</li>
    <li><a href="https://realpython.com/python-wheels/">What Are Python Wheels and Why Should You Care?</a></li>
    <li><a href="https://docs.python.org/3/distutils/setupscript.html#other-options">Writing the Setup Script</a> with <a href="https://docs.python.org/3/distutils/apiref.html">API</a></li>
    <li><a href="https://packaging.python.org/en/latest/tutorials/packaging-projects/">Packaging Python Projects</a></li>
</ul>
