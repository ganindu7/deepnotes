---
layout: default
title: Setting up CVAT
nav_order: 2 
# permalink: /topics/utils/target_execute
parent: Data Prep
grand_parent: Utilities
---


## Setting up [CVAT][CVAT]
<span style="background-color:LightGreen">
Created : 08/11/2022 | on Linux: 5.15.0-52-generic <br />
Updated: 08/11/2022 | on Linux: 5.15.0-52-generic <br />
Status: Draft
</span>


### Installing CVAT locally:

*Note: I personally like to install [pyenv](https://github.com/pyenv/pyenv) to keep my system python clean.* <br/> 

please follow these [installation steps](https://opencv.github.io/cvat/docs/administration/basics/installation/) listed in the official [CVAT][CVAT] repository. 

I checked out the `master` branch instead of the development branch. 


After installing for the first time I suggest that you run docker-compose and keep the shell running to see the diagnostics. (this can be done by omitting the `-d` flag in the compose command) <br/>

Then with a new shell you can log into the django environment with `docker exec -it cvat_server /bin/bash --login`  

once you log in you can create a user with 

```
python manage.py createsuperuser
```
Now you can login to the server depending `CVAT_HOST` variable you set at the initialisation using google chrome

e.g. type `http://cvat.gsrv.lan:8080/` in the address bar. (*note: this will only work for me in my lab environment*)

After you conform everything works fine you may restart the server with the shell detached. 

### Shutting down

You can shut the server down by `cd` ing to the cvat directory and then typing `docker-compose down`

<br />


### Tricks 

#### Log into PostgreSQL 

you can log into `postgres` with <br/> 

`docker exec -it cvat_db /bin/bash --login`  <br/>

#### Change where the data is stored or retrieved from. 

if you want to create a database in a separate place follow along the instructions [here](https://opencv.github.io/cvat/docs/administration/basics/installation/#share-path) but instead of the `cvat_share` use the `cvat_data` option shown below (or use both appropriately). The extra storage is now mounted to somewhere in `/mounted/files/location/cvat_extended_storage/cvat_database`

```
volumes:
  cvat_data:
    driver_opts:
      type: none
      device: /mounted/files/location/cvat_extended_storage/cvat_database
      o: bind

``` 

<!-- <span style="background-color:LightYellow"> Check the [**next topic**](../pytorch_walkthrough#Starting-Development-with-PyTorch)  </span> -->

---
*Click [here][ERRORS-SUGGESTIONS] to report Errors, make Suggestions or Comments!*

[CVAT]: https://www.cvat.ai/
[ERRORS-SUGGESTIONS]: https://github.com/ganindu7/deepnotes/issues

<!-- Latex in markdown -->
<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>


