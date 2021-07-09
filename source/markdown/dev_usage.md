# Developing with priv-libs

## priv-libs development shell 

One would normaly just run `build.sh` in the root of `priv-libs` directory to build the `priv-libs` image. Once can alternatively do the following to get a test shell that will allow you to test the libraries and run tests:   

1. cd into `priv-libs` root folder   
2. build `priv-libs` docker image using   
   - `./build-run-shell.sh`   
3. docker should now build the image and spawn a shell
   - the working directory is `/priv-libs`
4. after exiting (with command `exit`) the `build-run-shell.sh` script will also remove the container   
   
In the shell one can now use any of the libraries, tests, and anything else that is placed under `<priv-libs-root>/priv-libs`, these are copied to `/priv-libs` inside the container.

## Using libraries

One can use the libraries with `python3` by using the following snippet of code to import them:   
```python

import sys
sys.path.append("/priv-libs/libs")

# import using library name
# for example 
from de import RSADOAEP,AESSIV, AESCBC, AESCMC

```

For real examples look at `<priv-libs-root>/priv-libs/tests`.   

```{admonition} Warning
:class: warning

Use `python3` when running code. NOT `python2` or `python`. For example to run a test one must run `python3 ore.py`.
```

## Running development tests

To run a test one can do the following:   
1. use `./build-run-shell.sh` to get a shell
2. `cd tests`
3. `python3 <test-name.py>`
   
   
## Updating the code without rebuilding (development)   

To update the live development conainter's (only when using `build-run-shell.sh`) code without having to rebuild do the following:    
   
1. run `build-run-shell.sh` to get a development container with shell   
2. when its time to update code    
   - open new separate shell    
   - cd into the `priv-libs` root   
   - execute `update-code.sh`   
3. code should now be copied to live container   



## Using docker priv-libs module

After building privlibs module one can include it in other docker modules, essentially how this whole project is built. To do so base your docker image on the `prib-libs` image. Done as follows:  

1. create `Dockerfile` file
2. place `FROM cybexp-priv-libs` on the top of the file to base your project on the priv libs image 

## Backend API

Backend API is only available via `priv-libs` module, the submodule name is `web_client`. Unless you are developing there is no need to interface to it directly as it only recieves queries that are bundled using python pickle modules with encrypted fields using more of  privlibs. To use this interface, either use them directly from a privlibs container or base your project on the priblibs container.   


  
## Getting shell

Most modules support getting a shell for their docker container. Use the `--shell` flag when using their respective wrapper script.
