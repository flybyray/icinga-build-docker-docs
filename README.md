Icinga Docker Build
===================

This project slowly replace all embedded scripts in
[Puppet module icinga_build](https://github.com/Icinga/puppet-icinga_build).

Please note:
* This is for an internal build environment
* Image names are not best practice

Home for all projects is
[build-docker on Icinga Gitlab](https://git.icinga.com/build-docker).

## Usage

Most scripts here are meant to be executed in the target Docker environment.

A helper script called `icinga-build-docker` will help you run the images and
scripts with a local Docker.

```
cp icinga-build-docker ~/bin/

cd ~/devel/icinga/rpm-icinga2

# build the package
icinga-build-docker centos/7:x86_64

# result of the build
ls -al build/

# test the package, e.g. package installation
icinga-build-docker centos/7:x86_64 icinga-build-test
```

The full image name would be: `registry.icinga.com/build-docker/centos/7:x86_64`

Several settings and defaults are controlled via environment variables, all
variables beginning with `ICINGA` or `APTLY` are passed into the container.

By default you will only need `icinga-build-docker` locally, every other script
is included in the containers, and auto-updated via GIT.

## Docker Images

Several images are provided via Gitlab, and can be pulled from the registry.

Please check the `gitlab-ci.yml` and `Makefile` inside the individual
repositories.

* [CentOS](https://git.icinga.com/build-docker/centos)
* [Debian](https://git.icinga.com/build-docker/debian)
* [Ubuntu](https://git.icinga.com/build-docker/ubuntu)
* [SLES](https://git.icinga.com/build-docker/sles)

Some helper images are available for special tasks:

Image                | Task
---------------------|-----
[sles-base]          | Base OS images, without subscription and any additional package
[sles-image-builder] | Scripts to build a SLES chroot installation and a Docker image
[upload]             | Helper container to upload a build package to the repository

## Other documentation

* [Development](development.md)
* [Scripts](scripts.md) (TODO)
* [Packages](packages.md) (TODO)

## License

Icinga, all tools and documentation are licensed under the terms of the GNU
General Public License Version 2, you will find a copy of this license in the
COPYING file included in the source package.

[sles-base]: https://git.icinga.com/build-docker/sles-base
[sles-image-builder]: https://git.icinga.com/build-docker/sles-image-builder
[upload]: https://git.icinga.com/build-docker/upload
