Scripts
=======

## Upload

### Examples

```
export APTLY_SERVER=http://172.17.0.1:8080/api
export APTLY_USERNAME=icingaadmin
export APTLY_PASSWORD=icinga

# automatic repo name in aptly
icinga-build-upload-aptly --target debian --release icinga-stretch

# staying conform with old repo names
icinga-build-upload-aptly --target debian --release icinga-stretch-snapshots --repo icinga-raspbian-stretch-snapshot
icinga-build-upload-aptly --target debian --release icinga-stretch --repo icinga-raspbian-stretch-release

# architectures, e.g. for raspbian
icinga-build-upload-aptly --target raspbian \
  --release icinga-stretch --repo icinga-raspbian-stretch-release --architectures armhf

# for rpms
icinga-build-upload-aptly --target epel --release 7/release
icinga-build-upload-aptly --target epel --release 7/snapshot
```
