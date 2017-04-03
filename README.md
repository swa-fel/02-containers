## Software Architecture 2017 - Linux containers
This lab consist of several exercises in which students learn about linux containers. From basics like what is a linux container and how to get and run containers to more advanced features, such as how to apply different kinds of linux namespaces and cgroups. Students don't interact with underlying kernel features directly, but rather use the docker tool, that makes working with linux containers easier.

### Install the docker tool
Use the `yum` command to download the docker package and add your user to the `docker` group so you don't need to type `sudo` every time you invoke the command.
```
sudo yum install docker
sudo groupadd docker
sudo gpasswd -a $(whoami) docker
sudo systemctl start docker
```
If you're wandering why this is necessary [this](http://www.projectatomic.io/blog/2015/08/why-we-dont-let-non-root-users-run-docker-in-centos-fedora-or-rhel/) article is for you.

Consult the manual page for the `run` subcommand: `man docker run`.

### The Lab
#### 1) Hello World
```
docker run hello-world
```
#### 2) First steps
```
docker run -i -t fedora:24 /bin/bash
pwd
ls
cat /etc/hostname
uname -r
cat /etc/fedora-release
ls -al /
```
Look around, anything interesting?
As the last command, delete something important...
```
rm -rf /usr/bin
ls
```
This system is broken! Let's get rid of it and start a new one:
```
exit
docker ps -a
docker rm <id>
docker run -i -t fedora bash
ls
exit
```
That's better, isn't it?

#### 3) Namespaces
Install some basic tools that we'll use to gather information about the container at runtime:
```
docker run -it fedora bash
dnf install -y procps-ng iproute hostname
```
* Network namespace. Compare outputs of the following commands inside the container and on the host.
```
# in container:
ip a
# on the host:
ip a
```
* PID namespace
```
# in container:
ps ax
sleep 10000
# on the host:
ps axf | grep -v grep | grep -B 2 sleep
```
* UTS namespace
```
# in container:
hostname
# on the host:
hostname
# or cat /etc/hostname
```

#### 4) CGroups
* [Memory limit](https://docs.docker.com/engine/reference/run/#/user-memory-constraints)
```
# start a new container
docker run -it --rm --memory 256m pschiffe/docker101-fedora bash
cat /sys/fs/cgroup/memory/memory.limit_in_bytes
# in container:
stress --vm 2 --vm-bytes 512M
# on the host:
systemd-cgtop | grep docker
```
* [CPU limit](https://docs.docker.com/engine/reference/run/#/cpu-period-constraint)
```
# start a new container
docker run -it --rm --cpu-period=50000 --cpu-quota=25000 pschiffe/docker101-fedora bash
# in container:
stress --cpu 4
# on the host:
systemd-cgtop -n 2 | grep docker
```
* [CPU set](https://docs.docker.com/engine/reference/run/#cpuset-constrain#### t) and [CPU shares](https://docs.docker.com/engine/reference/run/#cpu-share-constraint)
```
# start the two following containers
docker run -it --rm --cpuset-cpus=1 --cpu-shares=1024 pschiffe/docker101-fedora bash
docker run -it --rm --cpuset-cpus=1 --cpu-shares=512 pschiffe/docker101-fedora bash
# run the following command inside each of them
stress --cpu 1
# on the host:
systemd-cgtop -n 2 | grep docker
```
#### 5) Networking
* Publish all ports
```
docker run -d -P --name my-nginx nginx
# see container metadata
docker inspect my-nginx
# use go templating to get only specific fields
docker inspect --format {{.NetworkSettings.Ports}} my-nginx
# visit nginx's welcome page in browser (use the http port, will not work in gcloud by default)
# there are better ways how to get list of exposed ports
docker ps
docker port my-nginx
# kill container
docker rm -f my-nginx
```
* Publish only specific port (students do on their own)

Publish port 80 inside of the container as port 8080 on the host. Use the man page `man docker run`.
Is there any difference between publishing port 80 and 8080? [Hint](https://www.w3.org/Daemon/User/Installation/PrivilegedPorts.html)
* See logs of a container
```
# my-nginx must be properly started
docker logs --follow my-nginx
# visit welcome page in browser, refresh, watch the logs
```
#### 6) Volumes
```
docker run -d -p 80:80 -v $PWD/nginx:/usr/share/nginx/html:ro,Z --name my-nginx nginx
# visit the page in browser
# edit index.html in nginx dir and see the changes in browser
# clean up
```
#### 7) Building container images
In this section you will build two container images using the [Dockerfile](https://docs.docker.com/engine/reference/builder/)
* Building image from scratch
In this exercise you'll build container image that contains only statically built binary and its configuration. All data you need is in the `caddy` directory of this repository, go ahead checkout its content.
```
# is caddy really statically compiled?
ldd caddy
	not a dynamic executable
# what would happen if it wasn't?
docker build --tag=swa/caddy ./caddy
docker run -d -p 80:8080 --name caddy swa/caddy
```
* Build image on top of fedora base image
```
# see the Dockerfile in nginx directory. Consult its content with the documentation.
docker build -t swa/nginx ./nginx
docker run -d -p 80:80 swa/nginx
# visit the welcome page in browser
```
#### 8) Docker Hub
Visit hub.docker.com.
