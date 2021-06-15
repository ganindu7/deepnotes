---
layout: default
title: setting up pyenv 
nav_order: 2 
# permalink: /topics/utils/target_execute
parent: Utilities
---

## setting up PYENV

[Pyenv](https://github.com/pyenv/pyenv)is one of the most useful tools 



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

