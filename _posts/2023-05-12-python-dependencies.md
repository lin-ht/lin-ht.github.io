---
layout: post
title:  Python dependencies
date:   2023-05-12 13:40:00
description: Manager
tags: notes code
categories: note-posts
---
#### Python project dependencies management
Various combinations of tools:

| Feature | venv + pip | conda + conda-lock | Poetry + Pyenv |
| ----------- | ----------- | ----------- | ----------- |
| Usability | ✅ | ✅ | ✅ |
| Virtual environments | ✅ | ✅ | ✅ |
| Python versions | ❌ | ✅ | ✅ |
| Dependency management | ✅ | ✅ | ✅ |
| Reproducability | ✅ | ✅ | ✅ |
| Collaboration | ❌ | ❌✅* | ✅ |


##### Example of using venv+pip
0. You need to have global python3 installed first.

1. Create a new virtual environment in a local folder:
```bash
cd repo-dir
python3 -m venv .venv
```

2. Activate the env
```bash
source .venv/bin/activate
# For deactivation:
# deactivate
```

3. Prepare pip
```bash
python3 -m pip install --upgrade pip
python3 -m pip --version
```

4. Using requirements.txt for package specification
Check <a href="https://pip.pypa.io/en/latest/reference/requirements-file-format/#requirements-file-format">requirements file format</a>.
```txt
# requirements.txt
numpy
```

5. Pip install depencies
```bash
python -m pip install -r requirements.txt
```

##### Install and Uninstall
Packages installed by `python setup.py install`.
[<a href="https://stackoverflow.com/questions/1550226/python-setup-py-uninstall">stackoverflow reference</a>] Note: Avoid using `python setup.py install` use `pip install .`
To uninstall, you need to remove all installed files manually, and also undo any other stuff that installation did manually.
To record a list of installed files, you can use:
```bash
python setup.py install --record files.txt
```
Once you want to uninstall you can use xargs to do the removal:
```bash
xargs rm -rf < files.txt
```

##### Trouble Shooting
1. ModuleNotFound Error
If you run "pip -V" in the cli it will display where pip will install.

If you run 'import sysconfig; print(sysconfig.get_paths()["purelib"])' it will show where python looks for packages.

If you know which interpreter you want to use, you can ensure an install will be going to the right place by running "python3 -m pip install mymodule".


#### References
<ul>
	<li><a href="https://www.fuzzylabs.ai/blog-post/managing-python-dependencies">Managing Python Dependencies</a></li>
    <li><a href="https://packaging.python.org/en/latest/guides/installing-using-pip-and-virtual-environments/">Install packages in a virtual environment using pip and venv</a></li>
</ul>