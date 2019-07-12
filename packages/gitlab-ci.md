GitLab CI config for packages
=============================

These are examplary configs for a `.gitlab-ci.yml`. Please also see the examples and

## Debian

```
stages:
  - build
  - test
  - upload

variables:
  DOCKER_IMAGE_BASE: registry.icinga.com/build-docker

  # for snapshot builds
  ICINGA_BUILD_TYPE: snapshot

  # for pre-releases builds
  ICINGA_BUILD_RELEASE_TYPE: testing

  # usually set per build dist to use a specific flavor
  #ICINGA_BUILD_DEB_FLAVOR: stretch

.build: &build
  stage: build
  tags:
    - docker
  image: ${DOCKER_IMAGE_BASE}/${DOCKER_IMAGE}
  script:
    - icinga-build-package
  cache:
    key: "${CI_JOB_NAME}"
    paths:
      - ccache/
  artifacts:
    paths:
      - build/*
    expire_in: 1 week

.test: &test
  stage: test
  tags:
    - docker
  image: ${DOCKER_IMAGE_BASE}/${DOCKER_IMAGE}
  script:
    - find build/
    - icinga-build-test

.upload: &upload
  stage: upload
  tags:
    - docker
  image: ${DOCKER_IMAGE_BASE}/upload
  script:
    - find build/
    - icinga-build-upload-aptly
  only:
    - tags
    - master

build/debian/buster:
  <<: *build
  variables:
    DOCKER_IMAGE: debian/buster

test/debian/buster:
  <<: *test
  variables:
    DOCKER_IMAGE: debian/buster
  dependencies:
    - build/debian/buster

upload/debian/buster:
  <<: *upload
  dependencies:
    - build/debian/buster

build/ubuntu/bionic:
  <<: *build
  variables:
    DOCKER_IMAGE: ubuntu/bionic

test/ubuntu/bionic:
  <<: *test
  variables:
    DOCKER_IMAGE: ubuntu/bionic
  dependencies:
    - build/ubuntu/bionic

upload/ubuntu/bionic:
  <<: *upload
  dependencies:
    - build/ubuntu/bionic
```

## RPM

```
stages:
  - build
  - test
  - upload

variables:
  DOCKER_IMAGE_BASE: registry.icinga.com/build-docker

  # for snapshot builds
  ICINGA_BUILD_TYPE: snapshot

  # for pre-releases builds
  ICINGA_BUILD_RELEASE_TYPE: testing

.build: &build
  stage: build
  tags:
    - docker
  image: ${DOCKER_IMAGE_BASE}/${DOCKER_IMAGE}
  script:
    - icinga-build-package
  cache:
    key: "${CI_JOB_NAME}"
    paths:
      - ccache/
  artifacts:
    paths:
      - build/*
    expire_in: 1 week

.test: &test
  stage: test
  tags:
    - docker
  image: ${DOCKER_IMAGE_BASE}/${DOCKER_IMAGE}
  script:
    - find build/
    - icinga-build-test

.upload: &upload
  stage: upload
  tags:
    - docker
  image: ${DOCKER_IMAGE_BASE}/upload
  script:
    - find build/
    - icinga-build-upload-aptly
  only:
    - tags
    - master

build/centos/7:
  <<: *build
  variables:
    DOCKER_IMAGE: centos/7

test/centos/7:
  <<: *test
  variables:
    DOCKER_IMAGE: centos/7
  dependencies:
    - build/centos/7

upload/epel/7:
  <<: *upload
  dependencies:
    - build/centos/7

build/sles/12.4:
  <<: *build
  variables:
    DOCKER_IMAGE: sles/12.4

test/sles/12.4:
  <<: *test
  variables:
    DOCKER_IMAGE: sles/12.4
  dependencies:
    - build/sles/12.4

upload/SUSE/12.4:
  <<: *upload
  dependencies:
    - build/sles/12.4
```

## Examples

You can find examples in several testing repositories:

* [deb-icingabuildtest](https://git.icinga.com/packaging/deb-icingabuildtest)
* [rpm-icingabuildtest](https://git.icinga.com/packaging/rpm-icingabuildtest)
