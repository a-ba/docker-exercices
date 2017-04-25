# base image: last debian release
FROM debian:buster


###############################################################################
# system configuration (packages, user account) + unpack etherpad-lite
###############################################################################

# get the latest package lists and apply the needed upgrades
RUN apt-get update && apt-get -y dist-upgrade


# install all required packages
RUN apt-get -y install --no-install-recommends nodejs unzip curl python libssl-dev pkg-config build-essential


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
RUN apt-get -y install --no-install-recommends npm

# set the default user to 'etherpad'
# (will affect all containers running this image, including subsequent RUN
# commands)
USER etherpad

# set the default command for the container
CMD ["bin/run.sh"]

