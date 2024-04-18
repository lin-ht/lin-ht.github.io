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
# requirement.txt
numpy
```

5. Pip install depencies
```bash
python -m pip install -r requirement.txt
```
#### References
<ul>
	<li><a href="https://www.fuzzylabs.ai/blog-post/managing-python-dependencies">Managing Python Dependencies</a></li>
    <li><a href="https://packaging.python.org/en/latest/guides/installing-using-pip-and-virtual-environments/">Install packages in a virtual environment using pip and venv</a></li>
</ul>