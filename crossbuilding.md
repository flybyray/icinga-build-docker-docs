Cross Building
==============

## Raspbian (ARM)

Since we want to support other platforms like Raspbian, I investigated an easy solution to run builds in our Docker
build system.

**Why not build on target platform?**

Most ARM systems are pretty small sized since they are meant for IOT use, and don't need much computing power.

On most Raspberry Pi a build can take multiple hours.

**Why Docker?**

It is well integrated in our build system and an easy solution for throw-away environments.

### QEMU user

The QEMU virtualization software can actually run builds in the same user space as the host, wrapping all CPU
instructions during execution.

Mainly this avoids running a complete VM for the foreign architecture, so any chroot or Docker build could benefit
from a nearly native environment.

```
sudo apt install qemu-user-static
qemu-user-static <any-arm-binary>
```

When inspecting the process list, you will see the foreign binary run with the interpreter, just
as Python or Perl is visible.

### Binfmt misc

The Linux kernel has support for recognizing foreign binaries, and running them with a custom interpreter. A meta
package configures the various interpreters installed.

```
sudo apt install binfmt-support
sudo update-binfmts --display

ls -al /proc/sys/fs/binfmt_misc/
```

### Using in Docker

The environment inside Docker sees the same binfmt configuration as the host, and can simply use it.

Only requirement is that the "interpreter" binary is also available inside the Docker container, this can be done via
volume mounting:

```
docker run -it ... -v /usr/bin/qemu-arm-static:/usr/bin/qemu-arm-static:ro raspbian
```

For environments like GitLab this can be configured in the runner itself, so the volume is mounted every time:

```toml
[[runners]]
  name = "docker-arm-static"
  url = "https://git.icinga.com"
  token = "xxx"
  executor = "docker"
  [runners.docker]
    tls_verify = false
    image = "alpine"
    privileged = true
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/cache", "/usr/bin/qemu-arm-static:/usr/bin/qemu-arm-static:ro"]
    shm_size = 0
```

### Kernel Support

I noticed newer Kernels, at least 4.18 (I use Debian testing), supports this a bit better.

This newer implementation does not seem to require the interpreter inside the container, but it seems to load it
the interpeter binary from the host's filesystem namespace.

Since it is a static binary, it doesn't need any additional resource.

TODO: This is still under investigation.
