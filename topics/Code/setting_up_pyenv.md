---
layout: default
title: setting up pyenv 
nav_order: 2 
# permalink: /topics/utils/target_execute
parent: Utilities
---

## setting up PYENV and MINICONDA

[Pyenv](https://github.com/pyenv/pyenv) is one of the most useful tools to get a robust development environment up and running. 

Even when installing [Miniconda](https://docs.conda.io/en/latest/miniconda.html) or Anaconda(check miniconda out before anaconda if you haven't) it is worth going through pyenv. (here is a useful [article](https://towardsdatascience.com/managing-virtual-environment-with-pyenv-ae6f3fb835f8))

<span style="background-color:LightYellow">
Extra Tip: before installing a new python version please make sure you have all necessary CONFIGURE_OPTS enabled i.e shared_libraries like [here](https://github.com/pyenv/pyenv/blob/master/plugins/python-build/README.md#building-with---enable-shariied) </span>

### Building with `--enable-shared`

You can build CPython with `--enable-shared` to install a version with
shared object.

If `--enable-shared` was found in `PYTHON_CONFIGURE_OPTS` or `CONFIGURE_OPTS`,
`python-build` will automatically set `RPATH` to the pyenv's prefix directory.
This means you don't have to set `LD_LIBRARY_PATH` or `DYLD_LIBRARY_PATH` for
the version(s) installed with `--enable-shared`.

```
$ env PYTHON_CONFIGURE_OPTS="--enable-shared" pyenv install 3.x.y
```


### Virtual Environments 

Virtual environments are useful when you need to avoid touching the system python, (which is the recommended way for most development)
pyenv plugin [pyenv-virtualenv](https://github.com/pyenv/pyenv-virtualenv) provides this functionality. 


once pyenv-virtualenv is set up `pyenv virtualenv  3.x.y name_of_vitualenv`



### Installing Latest Miniconda (QUITE OPTIONAL)

Install the latest miniconda 

```bash
$ pyenv install miniconda3-latest 

```

Create a virtual environment 

```bash
$  pyenv virtualenv miniconda3-latest YOUR-PREFERRED-NAME

```

Conda caveats: 

Conda may sometimes not support your target platform. check before committing 


### Fixing broken python wheel 

I encountered a problem where I couldn't use pip or wheel. I had to use [this](https://github.com/ansible-community/molecule/issues/1966#issuecomment-749982697)

```
curl https://bootstrap.pypa.io/get-pip.py | python -

```

