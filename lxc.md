LXC
=========

Cheat sheet
------------
Show all containers available and their status

    lxc-ls -f

Start/stop a container

    lxc-start -n <name>
    lxc-stop -n <name>
Start a container in foreground mode

    lxc-start -n <name> -F

Attach to a running container

    lxc-attach -n <name>
Running a command in a container

    lxc-attach -n <name> -- ps -ef
Interactively creating a new container

    lxc-create -t download -n <name>
Non-interactively creating a new container

    lxc-create -t download -n <name> -- -d ubuntu -r trusty -a amd64

Files and locations
-------
The containers you create are physically located on the filesystem in */var/lib/lxc/<name>/*. This directory contains the configuration file of the container and the root filesystem of the container.

    root@anansi:~# ls -lart /var/lib/lxc/kippo/
    total 24
    drwx------ 127 root root 12288 Dec  7 11:50 ..
    -rw-r--r--   1 root root   522 Dec  7 11:51 config
    drwxrwx---   3 root root  4096 Dec  7 11:51 .
    drwxr-xr-x  21 root root  4096 Dec  7 11:52 rootfs

Creating a container
-------
In this example we will be creating a container for one of our detection methods (in this case, Kippo).

First we will need to create our base container

    root@localhost:/opt# lxc-create -t download -n kippo -- -d ubuntu -r trusty -a amd64

In the next step we will update the container and install Kippo with a simple shell script

    #!/bin/bash
    NAME=kippo
    lxc-start -n $NAME
    sleep 5
    lxc-attach -n $NAME -- apt-get update -y
    lxc-attach -n $NAME -- apt-get upgrade -y
    lxc-attach -n $NAME -- apt-get install -y python-twisted git
    lxc-attach -n $NAME -- git clone https://github.com/desaster/kippo.git /opt/kippo/
    lxc-attach -n $NAME -- cp /opt/kippo/kippo.cfg.dist /opt/kippo/kippo.cfg
    lxc-attach -n $NAME -- addgroup --system kippo
    lxc-attach -n $NAME -- adduser --shell /bin/bash --no-create-home --home /opt/kippo/ --system --ingroup kippo kippo
    lxc-attach -n $NAME -- chown -R kippo:kippo /opt/kippo/

This will have configured Kippo in the most basic configuration. It is ready to run, but if we want we can tweak the Kippo configuration any way we like.

After this we can configure Kippo to start whenever the container is started. Now because Kippo is not allowed to run as root, we will need to write an extra startup script we can use to hook into the lxc startup procedure.  

    #!/bin/bash
    cd /opt/kippo/
    sudo -u kippo ./start.sh

We will name this script kippostart.sh and put it in /opt/kippo/ (inside the lxc container). Attach yourself to the still running container and save the script in /opt/kippo/kippostart.sh.

    root@localhost:/opt# lxc-attach -n kippo
    root@kippo:/opt# vim /opt/kippo/kippostart.sh
    root@kippo:/opt# chmod +x /opt/kippo/kippostart.sh

Now we will need to edit the Kippo lxc container configuration file (on the host system: */var/lib/lxc/kippo/config*) and add the following line:

    lxc.hook.start = /opt/kippo/kippostart.sh

Now restart the container and it should automatically startup Kippo.
