Development
===========

In development mode, your script path is passed into the container, so the
local scripts are used, and not the ones inside the container.

```
PATH=~/devel/icinga/icinga-build-scripts:"$PATH"
ICINGA_DOCKER_PULL=0 ICINGA_SCRIPT_DEVEL=1 icinga-build-docker centos/7:x86_64
```
