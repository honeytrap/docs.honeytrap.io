Installing Wordpress in your Honeytrap-POT
=========

**NOTE**: This page assumes you already have installed the honeytrap-pot software with the Ansible scripts.

First we will need to make sure we can SSH into our lxc container. To do this generate an SSH key on your honeytrap-pot host:

    ssh-keygen
    cat /root/.ssh/id_rsa.pub

Now add this key to the lxc container we are using as the base template for honeytrap-pot. In the default state this should be the troje_base container. Startup the container:

    lxc-start -n troje_base
    lxc-attach -n troje_base

By doing the **lxc-attach** we are now inside the running *troje_base* container. We can now install openssh-server (if that has not been done yet) and add our SSH key:

    apt-get update
    apt-get install openssh-server python-apt
    mkdir /root/.ssh/ && chmod 700 /root/.ssh/
    vim /root/.ssh/authorized_keys
    chmod 600 /root/.ssh/authorized_keys

Now we will need to add our running container to our Ansible inventory. Check which IP has been assigned to your container by running the following command:

    lxc-ls -f --active | grep troje_base
    root@anansi:~# lxc-ls -f --active
    troje_base  RUNNING  10.0.3.178  -     -       NO        

Now add this host (10.0.3.178) to your Ansible inventory by opening the inventory file in your Ansible directory and adding the following lines:

    [container]
    10.0.3.178

We can now run the wordpress playbook on this container by running the following command:

    ansible-playbook -i inventory wordpress.yml
