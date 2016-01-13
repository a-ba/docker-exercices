Docker Exercises
================

[Tutorial home](https://people.irisa.fr/Anthony.Baire/)

License: Creative Commons BY-NC 3.0


Part 2. Containers
------------------

1.  Check that docker is correctly running and that you have permission to use
    the engine

    ```
    docker info
    ```

2.  **(pull)** Pull an image from the official registry, eg: `debian:latest`
    (you can browse <https://store.docker.com> if you want to find other
    images).

    ```
    docker pull debian:latest
    ```

    `debian` is the repository name, and `:latest` is a tag that identifies an
    image in the repository (for the case of the debian repository it is the
    [latest stable
    release](https://store.docker.com/images/debian#about-this-image)). You
    could also write `:jessie` or `:stretch` to use a specific version of the
    distribution.
    

    You can check that your image is present in the docker engine:

    ```
    docker images
    ```

3.  **(run)** Run a container from this image.

    ```
    docker run debian:latest
    ```

    (you may also write `docker run debian` which is equivalent: `:latest` is
    the defaut tag if none are provided)

    Nothing happens? actually the container has already terminated, you can
    display it with `docker ps`, but add `-a/--all` because non-running
    container are not displayed by default.

    ```
    docker ps -a
    ```

    The default command of the debian image is `/bin/bash` and by default
    docker containers are run without stdin (it is redirected from
    `/dev/null`). Thus bash exits immediately.

4.  **(run a command)** You may override the default command by providing extra
    arguments after the image name. Then this command will be executed (instead
    of bash).

    ```
    docker run debian ls /bin
    docker run debian cat /etc/motd
    ```

5.  **(stdin)** Let's go back to bash, this time we want interact with the
    shell. To keep stdin open, we launch the container with `-i/--interactive`.

    ```
    docker run -i debian
    ```

    The container runs, but displays nothing. Actually bash is running in batch
    mode. You can try to execute commands (eg: `ls`, `id`, `hostname`...) and
    you will see the result.

    Bash is in batch mode because it is not running on a terminal (its stdout
    is a pipe, not a tty).

6.  **(tty)** To have a real interactive shell inside our container, we need to
    allocate a tty with `-t/--tty`

    ```
    docker run -t -i debian
    ```

7.  **(start)** You can exit your container and display the list of all
    containers:

    ```
    docker ps -a
    ```

    It is possible to start them again with `docker start`. Like the run
    command, you may use `-i` to have stdin open. Note that start expects you
    to tell which container you want to start. Containers may be identified
    either by their id (first column of `docker ps`) or by their name (last
    column). You may provide only the first digits of the id (as long as there is
    no ambiguity). Examples:
<code><pre>docker start -i 85bcdca6c38f
docker start -i 85bcd
docker start -i 85
docker start -i 85bcdca6c38f07e3f8140cbf8b4ad37fd80d731b87c6945012479439a450a443
docker start -i pensive_hodgkin
</pre></code>


8.  **(commit)** You can modify files inside a container. If you restart the
    same container you can note that these changes are still present. However
    they will not be present in the other container (even if they are running
    the same image) because docker uses a copy-on-write filesystem. Use the
    command `docker diff` to show the difference of a container from its image.

    Remember that all changes inside a container are thrown away when the
    container is removed. If we want save a container filesytem for later use,
    we have to *commit* the conainer (i.e take a snapshot).

    ```
    docker commit CONTAINER
    ```

    This operation creates a new image (visible in `docker images`). This image
    in turn can be used to start a new container.
    
    Note: `docker commit` does not affect the state of the container. If it is
    running, then it just keeps running. You may take as many snapshots as you
    like.


9.  **(rm)** You now have too many dead containers in your engine. You should
    use `docker rm` to remove them. Alternatively you can run `docker container
    prune` which removes *all* dead container.

10. **(extras)** If you still have extra time, you can experiment
    * the other `docker run` options we introduced so far:
        * `--rm` to remove the container automatically when it terminates
        * `-d/--detach` to run a container in the background
        * `-u/--user` to run the container as a different user
        * `-w/--workdir` to start the container in a different directory
        * `-e/--env` to set an environment variable
        * `-h/--hostname` to set a different hostname (the host name inside the container)
        * `--name` to set a different name (the name of the container in the docker engine)
        * also you may type `docker run --help` to display all configuration keys
    * other docker commands (note: some of these commands require the container
      to be running, just launch `docker run -d -t -i debian` to have one that
      keeps running in the background)
        * `docker inspect` to display the metadata of a container (json format)
        * `docker cp` to transfer files from/into the container
        * `docker exec` to have launch a separate command (**very useful** for providing a debugging shell -> `docker exec -t -i CONTAINER bash`)
        * `docker top` to display the processes running inside the container
        * `docker stats` to display usage statistics
        * `docker logs` to display the container output
        * `docker attach` to reattach to the console of a detached container


Part 3. Container inputs/outputs
--------------------------------

1.  **(external volume)** Run a container with `-v/--volume` to mount an external volume.


    Eg: mount the `/tmp/myvol` from the host machine at `/myvol` inside the container:

    ```
    docker run --rm -t -i -v /tmp/myvol:/myvol debian
    ```

    > **Note:** on Windows/MacOS over the [Docker
    > Toolbox](https://docs.docker.com/toolbox/overview/), your docker engine
    > is running inside a virtual machine. This means `/tmp/myvol` refers to
    > the `/tmp/myvol` path *inside* the VM. You can mount directories from
    > your host system only if they are shared with the VM. By default the
    > toolbox is configured to share the `Users` directory inside the VM:
    >   * on Windows `C:\\Users` is mounted as `/c/Users`
    >   * on MacOS `/Users` is mounted as `/Users`
    >
    >
    > Thus you can mount a directory from these places, eg:
    >
    > `docker run --rm -t -i -v '/c/Users/NAME/My documents/myvol:/myvol' debian`
    >
    > `docker run --rm -t -i -v '/Users/NAME/Documents/myvol:/myvol' debian`

    Once the container is started, you can note that the directory `/tmp/myvol`
    is mounted inside the container at `/myvol`. If you create files there they
    will be visible on both sides, and they will persist if the container is
    removed. You can remove your container and create a new one with the same
    parameters to check that.

    This way of using an external volume is a *direct* mount. The docker engine
    will not care about the management of this directory (apart from creating
    it).

2.  **(named volume)** Alternatively we can create a *named volume*. That is a volume managed by
    docker (and by stored by default in `/var/lib/docker/volumes`). A named
    volume is a volume that does not start with `/`. Example:

    ```
    docker run --rm -t -i -v my-named-volume:/myvol debian
    ```

    This named volume is persistent of course. It is managed separately from
    the containers with the `docker volume` command, eg:

    ```
    docker volume ls
    docker volume rm my-named-volume
    ```

3.  **(running a server)** We will now play with the network.

    Pull the `nginx:stable-alpine` image *([nginx](https://nginx.org/) is a web
    server, and [alpine](https://alpinelinux.org/) is a very lightweight linux
    distribution based on the musl C library+busybox and a popular solution for
    building small docker images)*.

    ```
    docker pull nginx:stable-alpine
    ```

    Our goal here is to run a http server to serve some content (ex: the
    package documentations on our machine in `/usr/share/doc`). Before
    starting the container we need to know how to configure it, especially we
    need to know where the served directory must be mounted.

    The [documentation of the nginx
    image](https://store.docker.com/images/nginx#how-to-use-this-image) says
    the content is to be located in `/usr/share/nginx/html`. So let's mount our
    doc directory at this place. Note that nginx does not require write access
    on these files, therefore it is a good idea to append `:ro` to make the
    mount read-only.

    ```
    docker run -d --name nginx -v /usr/share/doc:/usr/share/nginx/html:ro nginx:stable-alpine
    ```
    
    The server is now running somewhere in a container. Since this container
    has a separate network stack, it has a different IP address. There are
    multiple ways to obtain this IP address:
    *   inspect the container metadata:

        ```
        docker inspect nginx | grep IPAddress
        ```

    *   run a command inside the container:

        ```
        docker exec nginx ip addr show dev eth0
        ```
    
    Once you know this address you can open it in your web browser ->
    `http://172.17.x.x/`. Unfortunately the nginx image is compiled without the
    autoindex module so it will not display directories without a index.html file.

    Just find a html document with the following command and append it to your
    url to see if it works.

    ```
    (cd /usr/share/doc && find * -name index.html)
    ```

4.  **(publish)** We have confiremed that we are able to run a HTTP server inside a container
    and serve some content. However this container is in a private network
    (`172.17.0.0/16`), it is not reachable from the public.

    To make it reachable we have to publish the HTTP port (tcp/80) on host
    machine (which may have a public IP address). We add a `-p/--publish`
    option:

    ```
    docker run -d --name nginx -v /usr/share/doc:/usr/share/nginx/html:ro -p 80:80 nginx:stable-alpine
    ```

    Then the server should be reachable at <http://localhost/>. **Note**: the
    command will fail if you already have a server using the port 80 of your
    machine. If this happens, you may specify an alternate port, eg: `-p
    1234:80` the server will be reachable at <http://localhost:1234/>

5.  **(legacy links)** Our nginx server is reachable from outside. Another use
    case would be to make a server reachable from another container (for
    example, a web application in a server may want to use a database hosted in
    another container).

    To test this feature we run a busybox container *(it's lightweight and it
    provides the wget http client)*
    
    ```
    docker run --rm -t -i --link nginx:http-server busybox
    ```

    This makes the `nginx` container reachable from this new container under
    the alias `http-server`. Inside the busybox container we make a HTTP
    request with wget:

    ```
    wget http://http-server/
    ```

6.  **(user-defined network)** Legacy links are deprecated and that is
    unfortunate. The alternative is have the two containers connected to the
    same internal network.

    By default they are launched on the `bridge` network. Depending on the
    configuration of your daemon (in `/etc/docker/daemon.json`),
    inter-container communications may or may not be authorized on the default
    bridge (in case we have `icc: false`).

    If icc is enabled, we can already communicate between the containers (using
    the container name as TCP/IP destination).

    ```
    docker run --rm -t -i busybox wget http://nginx/
    ```
    
    In a production context, to improve the security, it would be preferable to
    put unrelated containers in separate networks.

    To test this we will create a dedicated network named `ngnet`, to let our
    two containers communicate privately.

    ```
    docker network create ngnet
    ```

    We can display its config (and especially observe that it uses a different
    IP prefix) with:

    ```
    docker network inspect ngnet
    ```

    At container creation time, we can use `-n/--net` to select a specific
    network (instead of the default `bridge`). But it is also possible to
    connect and disconnext dynamically the containers to the networks
    (especially to allow having a container connected to multiple network).

    We connect our `nginx` container to the `ngnet` network:
    
    ```
    docker network connect ngnet nginx
    ```
    
    The container is now connected to the two networks (`bridge` on eth0 and
    `ngnet` on eth1). We can verify this:

    ```
    docker exec nginx ip addr
    ```

    We can now run our busybox container with `--net ngnet` to be on this
    network:

    ```
    docker run --rm -t -i --net ngnet busybox wget http://nginx/
    ```


Part 5. Using the builder
-------------------------


The objective of this part is to build and run 4 services:

1.  **nginx:** a simple HTTP server that serves static content

2.  **etherpad:** a collaborative editing application that will require
    persisten storage in an external volume

3.  **etherpad-mysql:** the same etherpad application but configured to use a
    mysql db as storage backend (and running in a separate container)

4.  **mini-httpd:** a webserver that we want to patch, this will require a
    multi-stage build


These examples are provided with a docker-compose configuration
(`docker-compose.yml`) to ease their deployment. The documentation for
docker-compose is available at https://docs.docker.com/compose/

Here are the main commands:

*    `docker-compose up` to deploy all the containers, if not present in the
     engine, they will be be automatically pulled from the registry (for
     container using the "image:" option) or built from source (for those using
     the "build:" option)
*    `docker-compose build` to force rebuilding the images (useful when the
     source were modified)
*    `docker-compose down` to clean up everything
*    `docker-compose run CONTAINER [ARG0 ARG1 ...]` to run a container with the
     same config as the container *CONTAINER* defined in `docker-compose.yml`
     (useful for debugging when nothing works -> just run `docker-compose run
     CONTAINER bash` to have a shell)

By default `docker-compose up` works in the foreground. If the command is
interrupted (Control-C), all container are stopped and the command terminates.
You can use `docker compose up -d` to launch it in the background.

### nginx server

We will build an image with a nginx server from the debian distribution and run
it to serve some static content.

This example is located in the **nginx** directory:

*   `nginx/nginx/` -> contains sources of our docker image (Dockerfile)
*   `nginx/www/` -> contains the static files we want to serve
*   `nginx/log/` -> will store the server logs
*   `nginx/docker-compose.yml` -> docker compose configuration

Procedure:

1.  Go to the `nginx/` directory and have a look to the docker-compose
    configuration

2.  Run `docker-compose up` to build the image and run the container. Note: the
    build should work but the container will stop immediately (because nginx is
    not yet installed).

3.  Edit the Dockerfile to have the nginx package installed in the image and
    override the default container command so that nginx is run when we do
    `docker-compose up`. Note: by default `docker-compose up` will not rebuild
    the image, you should run `docker-compose up --build` (or run
    `docker-compose build` before).

4.  Once it is running, open http://localhost:8080/ in your browser to see it
    working and check the `log/` directory to ensure that the logs are there.

    > **Note:** on Windows/MacOS, if your docker engine is running inside a
    > virtual machine, then you need to configure a port redirection in
    > VirtualBox (i.e redirect host TCP connections to 127.0.0.1:8080 towards
    > the 8080 port of the guest)

5.  In your Dockerfile you may add a `EXPOSE` instruction to declare that the
    container listens to its port 80 and a `VOLUME` to tell that
    `/var/www/html` and `/var/log/nginx` are expected to be mounted from
    external volumes.


### etherpad (dirtydb backend)

In this part we will run a real web application (etherpad) and have its
persisent data stored in an external volume. We will use the etherpad's default
backend (dirty db).

This example is located in the **etherpad** directory:

*   `etherpad/etherpad/Dockerfile`  -> Dockerfile draft
*   `etherpad/etherpad/1.6.1.zip`   -> sources of the 1.6.1 release
*   `etherpad/var/`                 -> external volume
*   `etherpad/docker-compose.yml`   -> docker-compose configuration

The installation procedure is documented here:
<https://github.com/ether/etherpad-lite/tree/1.6.1#installation>. You should keep it open,
and have a look in it during the exercise because we do not give all the
details! We will use the default config and install the source from the
1.6.1.zip archive provided in this directory.

1.  Go to the `etherpad/` and have a look to the docker-compose configuration
    and to the draft Dockerfile

2.  Modify the Dockerfile to install the required debian packages. Then run
    `docker-compose build`.

3.  Continue to modify the Dockerfile to:
    *   extract the sources in `/opt/etherpad/` and make them owned by the
        *etherpad* user
    *   set the default runtime configuration
        * run as user `etherpad`
        * run the command `bin/run.sh`
        * run inside the directory `/opt/etherpad/`

4.  **(first run)** Once you are done, run:

    ```
    docker compose up
    ```
        
    Once etherpad is started, open <http://localhost:9001/> in your web
    browser and test the app.

    > **Note:** on Windows/MacOS, if your docker engine is running inside a
    > virtual machine, then you need to configure a port redirection in
    > VirtualBox (i.e redirect host TCP connections to 127.0.0.1:9001 towards
    > the 9001 port of the guest)

5.  **(persistence)** Now we will ensure that the data are effectively stored
    in the external volume and that we still have them when we restart the
    container. Once the app is working, create a pad and put some text in it.

    Then Stop the container (hit Ctrl-C) and remove it completely with:

    ```
    docker-compose rm etherpad 
    ```

    List the content of the `var/` subdirectory, normally the dirtydb files
    should be there.

    Relaunch the container and reopen the pad in your browser. It should still
    be there.

6.  **(startup time)** When running etherpad, you noticed that its startup
    script installs some npm dependencies before running the app. This normally
    happens only during the first run. Unfortunately since we are running in an
    immutable image (the container filesystem is copy-on-write) all then
    packages are drop when the container is removed and they have to be
    reinstalled at every subsequent run.

    Inside `bin/run.sh` there is a line that runs `bin/installDeps.sh` before
    running etherpad. This is the script that installs the dependencies.

    Modifiy your Dockerfile so that `bin/installDeps.sh` is executed at build
    time, then rebuild and relaunch the container twice.

    ```
    docker-compose build
    docker-compose up
    (Ctrl-C)
    docker-compose rm
    docker-compose up
    ```

    The second run should be much faster.

7.  **(graceful stop)** Stop your container and restart it in the background.

    ```
    docker-compose stop
    docker-compose up -d
    ```

    Now stop it:

    ```
    docker-compose stop
    ```

    The container should have stopped quicky. If it takes 10 seconds, then this
    mean that the container did not stop cleanly, because `docker stop` forces
    the container stop after 10 seconds with a `SIGKILL` signal. This is bad
    because we do not want to play with the devil and corrupt our data.
    
    Check the output of the container:

    ```
    docker-compose logs
    ```

    There is effectively no messages of etherpad shutting down. It did not
    catch the termination signal from the docker engine.

    Restart the container, display the list of processes inside the container.
        
    ```
    docker-compose up -d
    (wait until ready)
    docker top etherpad_etherpad_1
    ```
    
    There should be only one process: the nodejs server. If you have a *bash*
    (or *sh*) process above it, then it means that node is a child process of
    *bash*. When docker stops a container, it sends the termination signall
    (`SIGTERM`) to the root process *only*. Because the shell script does not
    forward this signal, the node server cannot know it has to stop.

    Look at `bin/run.sh` and find the place where nodejs is run.

    ```
    docker-compose run etherpad cat bin/run.sh | less
    ```

    Its actually the last line and the command is prefixed with `exec` for
    replacing the current process (which is right), therefore run.sh is not the
    culprit.

    If it is not, then the culprit is the Dockerfile. Check your `CMD`
    instruction. It may look like one of:

    *   `CMD ["bin/run.sh"]`
    *   `CMD bin/run.sh`

    The first form is ok, it tells docker to execute ["bin/run.sh"]. The second
    is not. When provides as a string, the CMD command is run inside a shell.
    Thus the second form is equivalent to `CMD ["sh", "-c", "bin/run.sh"]`. 
    
    If in this case, fix your `CMD` line, rebuild the image, start the
    container, check with `docker top` that the sh is gone and stop with
    `docker-compose stop`.
    
    ...

    Then container does not stop properly, again!
    
    We have actually a second issue (or a first issue, if you did the CMD line
    right), but there is a hint : when we started the container in the
    foreground (`docker-compose up`), it stopped gracefully when hitting
    Ctrl-C. In fact nodejs expects to be terminated with the `SIGINT` signal,
    but not with `SIGTERM` (the default signal sent by docker and by the unix
    `kill` command).

    Add a `STOPSIGNAL` line to your Dockerfile to tell docker that this
    container must be stopped with `SIGINT`. Then rebuild and test your image,
    the container should now stop properly.
    
7.  **(healthcheck)** When running in production, if our container crashed and
    the process terminates, docker can report it to us (the container has
    terminated!). However if it freezes, the docker engine will have no clue
    that something wrong is happening inside the container.

    We can provide a `HEALTHCHECK` command in our Dockerfile. When present,
    docker will run the command periodically when the container is up and
    display its result in the status of the container.

    Add a `HEALTHCHECK` command that ensures that the app is responding (you
    can use the *curl* utility for that). The rebuild and restart, the
    healthcheck status should show in `docker ps`.

    
8.  **(session key)** Our container is running and our data are in the `var/`
    external. We want to make sure we did not miss anything, we will check that
    with `docker diff`.

    ```
    docker-compose up -d
    (wait until ready)
    docker diff etherpad_etherpad_1
    ```
    
    This command shows the differences in the container filesystem (from the
    docker image). In the list we can see that the session key `SESSIONKEY.txt`
    is stored outside the external volume, thus it will be dropped across
    restarts.

    While this is not critical for the app (a new key is regenerated anyway),
    it can be a burden for the users because their session will be closed when
    the server is restarted.

    Modify the Dockerfile to have the session key stored in the var volume (for
    example with a symbolic link).

9. **(metadata)** You should add `EXPOSE` and a `VOLUME` line to your
   Dockerfile to tell about the listened tcp port and the external volume.


### etherpad (mysql backend)

As a second step, will will extend our etherpad image to make it use a mysql
server as a storage backend.

This exercise is located in the **etherpad-mysql** directory. The Dockerfile
starts with `FROM etherpad_etherpad` to use the image from the previous
exercise as a base. For the mysql container we will use the official image
provided by the mariadb project.

You will need to:

1.  configure the mysql container to provide a database and a user account for
    etherpad. The documentation of the mariadb image is available at
    <https://store.docker.com/images/mariadb>. You may have to tune some
    [environment variables](https://hub.docker.com/_/mariadb#environment-variables)
    for the mysql container in your `docker-compose.yml`.

2.  provide a `settings.json` file in your etherpad image to use the mysql
    database.

3.  once done, run `docker-compose up` and see what happens.

4.  if you have extra time, you can modify the etherpad image to delay the
    execution of nojs until the mysql server is ready and responding.


### mini-httpd

In this exercise, we will build
[mini-httpd](https://acme.com/software/mini_httpd/) from sources and install it
in a docker image. This implies a **multi-stage** build: your Dockerfile will build two images:

1.  a first image with the development toolchain (compiler,...) that will build
    the binary
2.  a second image which will be the final image, it just needs to contain the
    binary we built in stage 1 + the minimal runtime dependencies.

Once it is working, you can modify the first stage to patch the sources of
mini-httpd (for example to change the background color of the directory
indexes).



<!-- vim:et:sts=4:sw=4:nosta:
-->
