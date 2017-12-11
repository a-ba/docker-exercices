# base image: latest debian release
FROM debian:latest

# install the latest upgrades (reduce verbosity)
RUN apt-get -qq update && apt-get -qqy dist-upgrade

# install nginx
RUN apt-get -y --no-install-recommends  install nginx-light

# set the default container command
# -> run nginx in the foreground
CMD ["nginx", "-g", "daemon off;"]

# Tell the docker engine that there will be something listening on the tcp port 80
EXPOSE 80
