# Vagrant howto

    vagrant box add --provider virtualbox puppetlabs/centos-7.2.64-nocm

This plugin ensures that the VirtualBox extensions on the virtual machine are kept up to date especially when a new kernel is installed:

    vagrant plugin install vagrant-vbguest

The learning repository contains a Vagrantfile that lists the systems we'll use in this book. Start these systems up using:
    vagrant up client

There are multiple machines that can be started via vagrant. Use
vagrant status
or
vagrant status NAME

to see the status of all machines.

Start a VM use 'vagrant up NAME':
    vagrant up client

You can suspend, resume, and destroy instances. e.g.
    vagrant suspend client
    vagrant resume client
    vagrant destroy client

If you destroy a machine then run 'vagrant up NAME' to launch a new instance

# Log in via ssh: 'vagrant ssh NAME' e.g.
    vagrant ssh client

