Honeytrap-POT
=========

How does it work?
------------
The Honeytrap-POT daemon acts as a smart proxy between the attacker and the attacked service in a linux container. So for example in the case of an SSH attacker the Honeytrap-POT daemon will accept the SSH connection. It will then start a linux container based of a template and proxy the attacker to this container. After the attacker is done it will keep a snapshot (delta) of this container on the system for further analysis.

Installation Honeytrap-POT
------------
The honeytrap-POT component can be installed with an Ansible script. Install Ansible on the host you want to run honeytrap-POT. After this checkout the Ansible scripts and run the anansi.yml playbook:

    apt-get install git ansible
    git clone https://github.com/honeytrap/docs-honeytrap-io.git
    cd docs-honeytrap-io/ansible
    ansible-playbook -i inventory anansi.yml

The steps this Ansible script will take are roughly described here:
 1. Change the SSH listening port to 8022 and reload the SSH daemon.
 2. Install the necessary prerequisite software (LXC, git, tor).
 3. Create a new linux container for the honeypot to run in.
 4. Configure the linux container.
 5. Setup the honeytrap directory structure.
 6. Generate a self-signed certificate for the honeypot.
 7. Install a startup script in rc.local.
 8. Reboot the machine

Starting/running Honeytrap-POT
----------------------
The installation script will install a script in /etc/rc.local which can be used to start (or restart) honeytrap. This script will kill any running honeytrap, start a new session in tmux (Terminal multiplexer) for honeytrap and run the honeytrap binary in this new session.

The current running session(s) can be listed by issuing the following command:

    tmux list-sessions

Viewing the output of the honeytrap binary can be done by opening the honeytrap session with the following command:

    tmux a -t honeytrap

This will show you the output of the honeytrap binary and any logging that it produces. Detaching from this session can be done by pressing ctrl+b d.

Configuration
-------------
The default configuration will configure honeytrap to listen to 6 ports of which 4 are attack vectors and 2 are management ports. The management ports are:

 - Port 3000 is used for gathering statistics.
 - Port 6887 as the communication channel with any active honeytrap agents.

The attack vectors are:

 - Port 22 for SSH
 - Port 25 for SMTP
 - Port 80 for HTTP
 - Port 5060 for SIP

These ports can also be configured in the honeytrap.yml configuration file (located in */opt/honeytrap/honeytrap.yml*) under the proxies section of the config.
