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


# install all required packages
RUN apt-get -qqy install --no-install-recommends nodejs unzip curl python libssl-dev pkg-config build-essential

# create a dedicated user
RUN useradd --create-home etherpad


# get the sources of etherpad-lite
# https://github.com/ether/etherpad-lite/releases/latest
#ADD https://codeload.github.com/ether/etherpad-lite/tar.gz/1.6.1   /opt/source.zip
COPY 1.6.1.zip   /tmp/src.zip

# extract it into /opt/etherpad and make it owned by the etherpad user
RUN cd /opt				\
	&&  unzip /tmp/src.zip		\
	&&  rm /tmp/src.zip		\
	&&  mv ether-etherpad-lite-* etherpad	\
	&&  chown -R etherpad: etherpad

# set the default working directory
# (will affect all containers running this image, including subsequent RUN
# commands)
WORKDIR /opt/etherpad

# make nodejs available as the command 'node'
RUN ln -s /usr/bin/nodejs /usr/local/bin/node

# install dependencies
RUN apt-get -qqy install --no-install-recommends npm

# set the default user to 'etherpad'
# (will affect all containers running this image, including subsequent RUN
# commands)
USER etherpad

# set the default command for the container
CMD ["bin/run.sh"]

# Expose tcp port 9001 and mark /opt/etherpad/var as an external volume
#
# NOTE:	these lines are not strictly required, since 'docker run' allows
#	publishing arbitrary ports and binding arbitrary mount points.
#
#	Including them in the Dockerfile (thus in the image metadata) has the
#	following effects:
#	  EXPOSE port
#	  - it documents the port listened by the container (shown by
#	    'docker ps' and 'docker inspect'), it does NOT publish it nor
#	    restrict access to other ports
#	  - all exposed ports are automatically published (on random host
#	    ports) when 'docker run' is called with option -P (publish all)
#	  VOLUME path
#	  - if the container is started without binding this path, then an
#	    anonymous volume is created on-the-fly for storing this path
#	    (see 'docker volume --help')
EXPOSE 9001
VOLUME /opt/etherpad/var

