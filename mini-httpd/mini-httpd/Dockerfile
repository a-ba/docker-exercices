# ---------------------------------------------------------------------
# Stage 0  build an intermediary docker image that builds a customised
#          version of mini-httpd
# ---------------------------------------------------------------------

# base image: last debian release
FROM debian:buster AS builder

# add the source repositories (for each 'deb' line in sources.list, we add the
# corresponding 'deb-src' line)
RUN sed -i '/^deb /p ; s/^deb /deb-src /' /etc/apt/sources.list

# update the package lists
RUN apt-get -qqy update

# install all packages required to build mini-httpd
RUN apt-get -qqy build-dep mini-httpd

# download the source package and extract it in /tmp/mini-httpd
RUN cd /tmp && apt-get -qqy source mini-httpd && mv mini-httpd-* mini-httpd

# run all subsequent command inside /tmp/mini-httpd (instead of /)
WORKDIR /tmp/mini-httpd

# apply our patches
COPY *.patch /tmp/
RUN cat /tmp/*.patch | patch -p2

# build the package and rename it as mini-httpd.deb
RUN dpkg-buildpackage -b && cd .. && mv mini-httpd_*.deb mini-httpd.deb


# ---------------------------------------------------------------------
#  Stage 1  build the final image
# ---------------------------------------------------------------------

# base image: last debian release
FROM debian:buster

# install libssl1.1 (requisite for mini-httpd)
RUN apt-get -qqy update && apt-get -qqy install libssl1.1

# get the mini-httpd.deb package we built in stage 0
COPY --from=builder /tmp/mini-httpd.deb /tmp/

# install it (and remove the .deb)
RUN dpkg -i /tmp/mini-httpd.deb && rm /tmp/*.deb

#
# default runtime config
#

# run mini_httpd
#  - in the foreground (-D)
#  - as user nobody (-u nobody)
#  - write logs to stdout (-l /dev/fd/1)
#  - do a chroot() into /html (-r -d /html)
#  - run as pid 1 so that we can stop cleanly (exec)
CMD exec mini_httpd -D -u nobody -l /dev/fd/1 -r -d /html
# the container will have a TCP server running on port 80, and it is meant to
# be published
EXPOSE 80
# the container expects an external volume to be mounted at /html (to provide
# the hosted files)
VOLUME /html
