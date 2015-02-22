FROM gliderlabs/alpine:latest

# Publish the typical syslinux and pxelinux files at tftp root.
# We only need one subtree of files from syslinux, and
# we don't need its dependencies, so...
#
# Install the package and its dependencies as a virtual set "sl_plus_deps",
# copy the desired files, then purge the virtual set.
#
# This trick keeps the image as small as possible.
#
# NOTE: If you bump the syslinux version here,
#       please also update the README.md.
RUN apk-install --virtual sl_plus_deps syslinux=6.03-r0 && \
    cp -r /usr/share/syslinux /tftpboot && \
    find /tftpboot -type f -exec chmod 0444 {} + && \
    apk del sl_plus_deps

# Add safe defaults that can be overriden easily.
COPY pxelinux.cfg /tftpboot/pxelinux.cfg/

# Support clients that use backslash instead of forward slash.
ADD https://git.kernel.org/cgit/network/tftp/tftp-hpa.git/plain/tftpd/sample.rules /tftpboot/rules/default

# Do not track further change to /tftpboot.
VOLUME /tftpboot

# http://forum.alpinelinux.org/apk/main/x86_64/tftp-hpa
RUN apk-install tftp-hpa

EXPOSE 69/udp

RUN adduser -D tftp

ENTRYPOINT ["in.tftpd"]
CMD ["-L", "--verbose", "-m", "/tftpboot/rules/default", "-u", "tftp", "--secure", "/tftpboot"]
