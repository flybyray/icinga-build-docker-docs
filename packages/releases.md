Building Releases
=================

Releases need to be tagged on the upstream side, so we can download a tarball automatically.

Valid upstream tags can differ based on config, but are usually:
* `v1.1.0`
* `v1.1.0-rc1` (see notes on pre-releases below)
* `1.1.0`

* Released via standard git tags in format `v1.1.0-rc1`
* Published to a separate repository, e.g. `testing`

## Normal Releases

When starting a new major version for releases, it will be branched from master.

```
$ git checkout -b 1.1 master
```

Any specific release preparation should be done to this branch, also GitLab CI configuration. Relevant changed should be cherry-picked or merged back to master.

At this point a few steps should be ensured in `.gitlab-ci.yml`:

* Build matrix for targets is up to date
* `ICINGA_BUILD_TYPE` is unset or set to `release`
* Other vars like `ICINGA_BUILD_RELEASE_TYPE`

### Debian

Release relevant are the following files, `changelog` defines the built version, which `watch` instructs the tool `uscan` on how to download the source tarball.

> **Note:** In old repository layouts packaging is split into multiple versions, so make sure to update all of them.

**debian/changelog**

```
icingabuildtest (1.0.0-1) icinga-stretch; urgency=medium

  * Test Package

 -- Markus Frosch <markus.frosch@icinga.com>  Wed, 21 Nov 2018 10:02:44 +0000
```

**debian/watch**

```
version=3
opts=filenamemangle=s/.+\/(?:icingabuildtest-|v)([\d.]+(?:-.+)?).tar.gz/icingabuildtest-$1.tar.gz/,versionmangle=s/^([\d.]+)-(.+)?/$1~$2/ \
https://github.com/Icinga/icingabuildtest/releases .*/(?:icingabuildtest-|v)([\d.]+(?:-.+)?).tar.gz
```

Notes on `debian/watch`:

* **filenamemangle** prefixes the resulting tarball with the project name
* **versionmangle** rewrites a pre-release version to the tilde notation
* uscan is actually able to list existing versions and download a specific one we then specify

```
$ export DEBFULLNAME="$(git config user.name)"
$ export DEBEMAIL="$(git config user.email)"

$ dch -v 1.1.0-1 "Release 1.1.0-1"
$ git add .
$ git commit -m "Release 1.1.0-1"
$ git push

$ git tag -m "Release 1.1.0-1" 1.1.0-1
$ git push --tags
```

### RPM

The release relevant parts of the SPEC file should look similar to this:

```rpmspec
Version:        1.0.0
Release:        1%{?dist}

Source:         https://github.com/Icinga/%{name}/archive/v%{version}.tar.gz
# ...

%changelog
* Thu Feb 07 2019 Markus Frosch <markus.frosch@icinga.com> 1.0.0-1
- Initial package
```

In `ICINGA_BUILD_TYPE` mode `release` the source is downloaded via spectool, so the version controls the built version.

```
$ vim *.spec
$ git add .
$ git commit -m "Release 1.1.0-1"
$ git push

$ git tag -m "Release 1.1.0-1" 1.1.0-1
$ git push --tags
```

## Pre-Releases

* Pre-Releases should be suffixed tags like `v1.1.0-rc1`
* Package files need to be prepared with proper versioning
* Source and watch config needs to be set correctly

Fully built versions for pre-releases should look like this:

**Debian:** 1.1.0~rc1-1 \
**RPM:** 1.1.0-0.rc1.1

In addition a few settings differentiate pre-releases from normal releases:

* `ICINGA_BUILD_RELEASE_TYPE` will be set to `testing` (or similar) - to upload to a different repository

### Pre-Releases on Debian

Important:

* In **debian/watch** the **versionmangle** settings should be similar as above

```
$ export DEBFULLNAME="$(git config user.name)"
$ export DEBEMAIL="$(git config user.email)"

$ dch -v 1.1.0~rc1-1 "Prepared pre-release"
$ git add .
$ git commit -m "Prepared pre-release 1.1.0~rc1-1"
$ git push

$ git tag -m "Pre-release 1.1.0~rc1-1" 1.1.0-rc1-1
$ git push --tags
```

### Pre-Releases on RPM

Since the version is slightly different as for normal releases a few changes to the SPEC file are required:

```rpmspec
%global src_version_suffix -rc1

Version:        1.1.0
Release:        0.rc1.1%{?dist}

Source:         https://github.com/Icinga/%{name}/archive/v%{version}%{?src_version_suffix}.tar.gz
# ...
%prep
%setup -q -n %{name}-%{version}%{?src_version_suffix}

%changelog
* Thu Feb 07 2019 Markus Frosch <markus.frosch@icinga.com> 1.0.0-1
- Initial package
```

```
$ vim *.spec
$ git add .
$ git commit -m "Prepared pre-release 1.1.0-0.rc1.1"
$ git push

$ git tag -m "Pre-release 1.1.0-0.rc1.1" 1.1.0-0.rc1.1
$ git push --tags
```

## Examples

You can find examples in several testing repositories:

* [deb-icingabuildtest](https://git.icinga.com/packaging/deb-icingabuildtest)
* [rpm-icingabuildtest](https://git.icinga.com/packaging/rpm-icingabuildtest)
* [icingabuildtest](https://git.icinga.com/packaging/icingabuildtest)
