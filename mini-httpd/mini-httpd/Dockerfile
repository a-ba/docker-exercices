# base image: last debian release
FROM debian:buster

# install mini-httpd
RUN apt-get -qqy update && apt-get -qqy install mini-httpd

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
