# base image: buster debian release
# NB: End-of-Life release used on purpose for this exercise: see README file.
#     Do NOT use it for real applications!
FROM debian:buster


###############################################################################
# system configuration (packages, user account) + unpack etherpad-lite
###############################################################################

# get the necessary archived package lists and apply the needed upgrades
RUN printf '%s\n' \
      "deb http://archive.debian.org/debian buster main" \
      "deb http://archive.debian.org/debian-security buster/updates main" \
      "deb http://archive.debian.org/debian buster-updates main" \
    > /etc/apt/sources.list
RUN apt-get -qq update && apt-get -qqy dist-upgrade


# TODO: install required packages
# RUN apt-get -qqy install --no-install-recommends  ...

# create a dedicated user
RUN useradd --create-home etherpad


# get the sources of etherpad-lite
# https://github.com/ether/etherpad-lite/releases/latest
#ADD https://codeload.github.com/ether/etherpad-lite/tar.gz/1.6.1   /opt/source.zip
COPY 1.6.1.zip   /tmp/src.zip

# TODO
# ...

