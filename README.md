tftp-hpa (tftpd) in a container
===============================

[![Download size](https://images.microbadger.com/badges/image/jumanjiman/tftp-hpa.svg)](http://microbadger.com/images/jumanjiman/tftp-hpa "View on microbadger.com")&nbsp;
[![Version](https://images.microbadger.com/badges/version/jumanjiman/tftp-hpa.svg)](http://microbadger.com/images/jumanjiman/tftp-hpa "View on microbadger.com")&nbsp;
[![Source code](https://images.microbadger.com/badges/commit/jumanjiman/tftp-hpa.svg)](http://microbadger.com/images/jumanjiman/tftp-hpa "View on microbadger.com")&nbsp;
[![Docker Registry](https://img.shields.io/docker/pulls/jumanjiman/tftp-hpa.svg)](https://registry.hub.docker.com/u/jumanjiman/tftp-hpa)&nbsp;
[![Circle CI](https://circleci.com/gh/jumanjihouse/docker-tftp-hpa.png?circle-token=a96c1956a20bb93a08f94b755d845b2ba0e324b2)](https://circleci.com/gh/jumanjihouse/docker-tftp-hpa/tree/master 'View CI builds')

Project URL: [https://github.com/jumanjihouse/docker-tftp-hpa](https://github.com/jumanjihouse/docker-tftp-hpa)

Registry: [https://registry.hub.docker.com/u/jumanjiman/tftp-hpa/](https://registry.hub.docker.com/u/jumanjiman/tftp-hpa/)


Overview
--------

This source is used to build an image for
[tftp-hpa](https://git.kernel.org/cgit/network/tftp/tftp-hpa.git/).
The image contains:

* H. Peter Anvin's [tftp server](https://git.kernel.org/cgit/network/tftp/tftp-hpa.git/)
* default, minimal configuration that you can easily override
* syslinux files suitable for a PXE server
* [map file](src/mapfile)
  to rewrite certain request paths

The runtime image is quite small (roughly 9 MB) since it is based on
[Alpine Linux](https://www.alpinelinux.org/).

The goal is to provide a compromise between a single, monolithic
tftpd image that contains *all the things* and a flexible tftpd
image that contains *just enough* to combine with custom-built
data containers or volumes an organization needs to bootstrap
their infrastructure.

:warning: The version of the image is tied to the version of tftp-hpa
as of 2020-January. Previously the version was tied to syslinux.


Build integrity and docker tags
-------------------------------

An unattended test harness runs the build script and acceptance tests.
If all tests pass on master branch in the unattended test harness,
circleci pushes the built image to the Docker hub.

The CI scripts on circleci apply two tags before pushing to docker hub:

* `jumanjiman/tftp-hpa:latest`: latest successful build on master branch
* `jumanjiman/tftp-hpa:<date>T<time>-git-<git-hash>`: a particular build on master branch

Therefore you can `docker pull` a specific tag if you don't want *latest*.


How-to
------

### Fetch an already-built image

The runtime image is published as `jumanjiman/tftp-hpa`.

    docker pull jumanjiman/tftp-hpa


### List files in the image

The image contains the typical syslinux, efi, and pxelinux files
from **syslinux 6.0.3** at `/tftpboot/`.
List them with:

    docker run --rm -t \
      --entrypoint=/bin/sh \
      jumanjiman/tftp-hpa -c "find /tftpboot -type f"


### Load NetFilter modules

Add helpers to track connections:

    sudo modprobe nf_conntrack_tftp
    sudo modprobe nf_nat_tftp


### Configure and run

The published image contains *just enough* files to provide
a base tftpd to PXE-boot your hosts to a simple menu.
The [simple menu](src/pxelinux.cfg/f1.msg) and
[`pxelinux.cfg/default`](src/pxelinux.cfg/default)
only allow to skip PXE.
Therefore you probably want to override the built-in menu.

Run a container with your own `pxelinux.cfg` files:

    docker run -d -p 69:69/udp \
      -v /path/to/your/pxelinux.cfg:/tftpboot/pxelinux.cfg:ro \
      jumanjiman/tftp-hpa

Run a container with your own `pxelinux.cfg` and map files:

    docker run -d -p 69:69/udp \
      -v /path/to/your/pxelinux.cfg:/tftpboot/pxelinux.cfg:ro \
      -v /path/to/your/mapfile:/tftpboot/rules/default:ro \
      jumanjiman/tftp-hpa

Use multiple volumes to add site-local boot files and menus
in addition to the built-in syslinux files:

    docker run -d -p 69:69/udp \
      -v /path/to/your/pxelinux.cfg:/tftpboot/pxelinux.cfg:ro \
      -v /path/to/your/bootfiles:/tftpboot/site:ro \
      jumanjiman/tftp-hpa


### Use systemd for automatic startup

Review and potentially modify the sample systemd unit file at
[`systemd/tftp-hpa.service`](systemd/tftp-hpa.service), then run:

    sudo cp systemd/tftp-hpa.service /etc/systemd/system/
    sudo systemctl daemon-reload
    sudo systemctl start tftp-hpa
    sudo systemctl enable tftp-hpa


### Build

On a docker host, run:

    ci/build
    ci/test


### Publish to a private registry

You can push the built image to a private docker registry:

    docker tag tftp-hpa registry_id/your_id/tftp-hpa
    docker push registry_id/your_id/tftp-hpa


Contribute
----------

See [`CONTRIBUTING.md`](CONTRIBUTING.md) in this repo.


License
-------

See [`LICENSE`](LICENSE) in this repo.
