---
layout: post
title:  Conda Tutorial
date:   2023-07-31 12:00:00
description: references
tags: notes
categories: note-posts
---
#### Python version management tool -- conda

##### Adding a python package
The default installation path should be `site-packages` of `python` folder. However, if you'd like to add a python package with customized installation path to the conda env: use command <a href="https://docs.conda.io/projects/conda-build/en/latest/resources/commands/conda-develop.html">conda-develop</a>.

##### Adding shared libraries
Check <a href="https://docs.conda.io/projects/conda-build/en/latest/resources/use-shared-libraries.html">here</a> for conda shared libraries. 

Conda depends primarily on searching directories for matching filenames. It does not currently use side-by-side assemblies.

For now, most DLLs are installed into `(install prefix)\\Library\\bin` --Q:what is this path?. This path is added to `os.environ["PATH"]` for all Python processes, so that DLLs can be located, regardless of the value of the system's PATH environment variable.

##### Execution environment var setting in conda env

Reference <a href="https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#macos-and-linux">here</a>.

```bash
cd $CONDA_PREFIX

# edit the var 
vim etc/conda/activate.d/env_vars.sh
# example content: export MY_KEY='secret-key-value'
vim etc/conda/deactivate.d/env_vars.sh
# example content: unset MY_KEY

# Check by the following
conda activate analytics
```



#### References
<ul>
    <li><a href="https://sachinjose31.medium.com/creating-an-environment-in-anaconda-through-a-yml-file-7e5deeb7676d">Creating an environment in anaconda through a yml file</a>.</li>
    <li><a href="https://conda.github.io/conda-pack/">Conda pack</a>.</li>
</ul>
