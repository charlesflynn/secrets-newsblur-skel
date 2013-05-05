# secrets-newsblur-skel
This project is an attempt to get NewsBlur running on a cluster of VMs using Vagrant. Please note, currently this project merely produces a clean installation on three VMs (app, db, and task). I will be working next on testing/troubleshooting the application. I thought others might want to help vagrantize NewsBlur, so the current state is a decent foundation.

## Prerequisites
My host OS is Lubuntu, so I needed the following packages via apt-get:

 * python-pip
 * python-dev
 * build-essential
 * rubygems
 * virtualbox (4.1+)
 * vagrant (1.1.x+)

**Python (via pip)**

 * boto
 * fabric
 * pyyaml

**Ruby (via gem)**

 * jammit (0.6.5)

I encountered an error with version 0.6.6, so I had to specify 0.6.5 with `sudo gem install jammit -v 0.6.5`


## Installing
    $ git clone git://github.com/charlesflynn/NewsBlur.git ~/projects/newsblur
    $ git clone git://github.com/charlesflynn/secrets-newsblur-skel.git ~/projects/secrets-newsblur
    $ cd ~/projects/newsblur
    $ vagrant up
    $ fab -R app setup_app
    $ fab -R db setup_db
    $ fab -R task setup_task

You should be prompted for the vagrant user password on each setup invocation. The password is 'vagrant' (this is the convention for vagrant boxes). See the <a href="#supervisor-errors">supervisor errors</a> section for details on an error you may encounter for each setup invocation.

## Viewing VM details
### Checking status
    $ vagrant status
    Current machine states:

    app                      running (virtualbox)
    db                       running (virtualbox)
    task                     running (virtualbox)

### Checking hardware configuration
    $ VBoxManage list runningvms -l | egrep 'Name:|Memory|CPUs'
    Name:            app
    Memory size:     768MB
    Number of CPUs:  1
    Name: 'vagrant-root', Host path: '~/projects/newsblur' (machine mapping), writable
    Name:            db
    Memory size:     768MB
    Number of CPUs:  1
    Name: 'vagrant-root', Host path: '~/projects/newsblur' (machine mapping), writable
    Name:            task
    Memory size:     768MB
    Number of CPUs:  1
    Name: 'vagrant-root', Host path: '~/projects/newsblur' (machine mapping), writable

### Checking network configuration
    $ for node in app db task; do VBoxManage guestproperty enumerate $node --patterns '*/1/V4/IP'; done
    Name: /VirtualBox/GuestInfo/Net/1/V4/IP, value: 10.100.100.11, timestamp: 1367715318996026000, flags:
    Name: /VirtualBox/GuestInfo/Net/1/V4/IP, value: 10.100.100.12, timestamp: 1367715376357595000, flags:
    Name: /VirtualBox/GuestInfo/Net/1/V4/IP, value: 10.100.100.13, timestamp: 1367715430879786000, flags:

## Supervisor errors
On my machine, the following error occurs the first time `setup_supervisor` is invoked on a new VM:

    [10.100.100.11] sudo: /etc/init.d/supervisor start
    [10.100.100.11] out: Starting supervisor: Error: The minimum number of file descriptors required to run this process is 10000 as per the "minfds" command-line argument or config file setting. The current environment will only allow you to open 4096 file descriptors.  Either raise the number of usable file descriptors in your environment (see README.txt) or lower the minfds setting in the config file to allow the process to start.
    [10.100.100.11] out: For help, use /usr/bin/supervisord -h
    [10.100.100.11] out:

    Fatal error: sudo() received nonzero return code 2 while executing!

    Requested: /etc/init.d/supervisor start
    Executed: sudo -S -p 'sudo password:'  /bin/bash -l -c "/etc/init.d/supervisor start"

    Aborting.
    Disconnecting from 10.100.100.11... done.

File descriptor limits are raised in `setup_ulimit` but supervisor didn't get the memo for some reason. I didn't want to unnecessarily mess around without verifying that other environments are affected, so I've just rerun the setup on failure (this isn't so awful, subsequent runs are faster). This is an ugly workaround until I get further into the project.
