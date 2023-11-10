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

Check the list of the installed packages:
```bash
conda list

# or show only pip packages
pip list
```

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


Another option to <a href="https://datacomy.com/python/anaconda/add_folder_to_path/">add a Python packages folder to conda path</a>:
(Not verified yet)

To permanently include packages or folder into the `PYTHONPATH` of an conda environment, use [conda develop](https://docs.conda.io/projects/conda-build/en/latest/resources/commands/conda-develop.html):

```bash
conda activate <your-env>

conda develop /PATH/TO/YOUR/FOLDER/OF/MODULES
```

This command will create file named `conda.pth` file in the site-packages folder of your environment, e.g. `~/YOUR_ENV/lib/pythonX.X/site-packages`.

As an alternative, you can just add a `conda.pth` file in `~/YOUR_ENV/lib/pythonX.X/site-packages` with the folder that you want to include in your `PYTHONPATH`. The filename can be different as long as it has the `.pth` extension. Every line of the file can contain a folder.


<a href="https://conda.io/projects/conda/en/latest/user-guide/configuration/use-condarc.html">Using the .condarc conda configuration file</a>

The `.condarc` file can change many parameters, including:
<ul>
    <li>Where conda looks for packages.</li>
    <li>If and how conda uses a proxy server.</li>
    <li>Where conda lists known environments.</li>
    <li>Whether to update the Bash prompt with the currently activated environment name.</li>
    <li>Whether user-built packages should be uploaded to Anaconda.org.</li>
    <li>What default packages or features to include in new environments.</li>
</ul>

Creating and editing the `.condarc` with `conda config` <a href="https://conda.io/projects/conda/en/latest/commands/config.html">commads</a>. Example:
```base
conda config --add channels conda-forge
conda config --set auto_update_conda False
```

Conda looks in the following locations for a `.condarc` file:
```python
if on_win:
    SEARCH_PATH = (
        "C:/ProgramData/conda/.condarc",
        "C:/ProgramData/conda/condarc",
        "C:/ProgramData/conda/condarc.d",
    )
else:
    SEARCH_PATH = (
        "/etc/conda/.condarc",
        "/etc/conda/condarc",
        "/etc/conda/condarc.d/",
        "/var/lib/conda/.condarc",
        "/var/lib/conda/condarc",
        "/var/lib/conda/condarc.d/",
    )

SEARCH_PATH += (
    "$CONDA_ROOT/.condarc",
    "$CONDA_ROOT/condarc",
    "$CONDA_ROOT/condarc.d/",
    "$XDG_CONFIG_HOME/conda/.condarc",
    "$XDG_CONFIG_HOME/conda/condarc",
    "$XDG_CONFIG_HOME/conda/condarc.d/",
    "~/.config/conda/.condarc",
    "~/.config/conda/condarc",
    "~/.config/conda/condarc.d/",
    "~/.conda/.condarc",
    "~/.conda/condarc",
    "~/.conda/condarc.d/",
    "~/.condarc",
    "$CONDA_PREFIX/.condarc",
    "$CONDA_PREFIX/condarc",
    "$CONDA_PREFIX/condarc.d/",
    "$CONDARC",
)
```


##### Conda env creation using a yml file

TODO

Update conda environment:
```bash
conda env export > environment.yml
# Tip: backup your env before update
# --prune option removes any packages from the environment that are not listed in the .yml.
conda env update --file environment.yml --prune
```

#### References
<ul>
    <li><a href="https://sachinjose31.medium.com/creating-an-environment-in-anaconda-through-a-yml-file-7e5deeb7676d">Creating an environment in anaconda through a yml file</a>.</li>
    <li><a href="https://conda.github.io/conda-pack/">Conda pack</a>.</li>
    <li><a href="https://saturncloud.io/blog/updating-an-existing-conda-environment-with-a-yml-file-a-guide/">Updating an Existing Conda Environment with a .yml File: A Guide</a></li>
</ul>
