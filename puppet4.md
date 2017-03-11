# Puppet 4

## Common Directories

Puppet is now installed using a new All-in-One (AIO) puppet-agent package
This AIO package includes private versions of Facter, Hiera, MCollective, and
Ruby.

Executables are in /opt/puppetlabs/bin/
All executables have been moved to /opt/puppetlabs/puppet/bin. A subset of exe‐
cutables has symlinks in /opt/puppetlabs/bin, which has been added to your path.
(You may need to restart your shell.)

A private copy of Ruby is installed in /opt/puppetlabs/puppet
Ruby and supporting commands such as gem are installed in /opt/puppetlabs/
puppet, to avoid them being accidentally called by users.

The configuration directory stored in $confdir is now /etc/puppetlabs/puppet
Puppet Open Source now uses the same configuration directory as Puppet Enter‐
prise. Files in /etc/puppet will be ignored.

The $ssldir directory is inside $confdir
On many platforms, Puppet previously put TLS keys and certificates in /var/lib/
puppet/ssl. With Puppet 4, TLS files will be installed inside the confdir/ssl/ direc‐ tory on all platforms.

The MCollective configuration directory is now /etc/puppetlabs/mcollective
Files in /etc/mcollective will be checked if the former directory doesn’t exist.

The $vardir for Puppet is now /opt/puppetlabs/puppet/cache/
This new directory is used only by puppet agent and puppet apply . You can
change this by setting $vardir in the Puppet config file.

The $rundir for Puppet agent is now /var/run/puppetlabs
This directory stores PID files only. Changing this directory will cause difficulty
with the installed systemd and init service scripts.

Modules, manifests, and the Hiera config file have a new directory: /etc/puppetlabs/code
Puppet code and data has moved from $confdir to a new directory $codedir .
This directory contains:
* The environments directory for $environmentpath
* The modules directory for $basemodulepath
* The hiera.yaml config file for $hiera_config

Useful command to display directory locations:
sudo /opt/puppetlabs/bin/puppet config print |grep dir

## Writing Manifests

A manifest is a file containing Puppet configuration language that describes how resources should be configured. The manifest is the closest thing to what one might consider a Puppet program. It uses resources to define a policy to be enforced on a node. It is therefore the base component for Puppet configuration policy, and a building block for complex Puppet modules.

## Writing

Resources are the smallest building block of the Puppet configuration language. They
represent a singular element that you wish to evaluate, create, or remove. Puppet
comes with many built-in resource types.

Example notify resouce:
'''
notify { 'greeting':
   message => 'Hello, world!'
}
'''
