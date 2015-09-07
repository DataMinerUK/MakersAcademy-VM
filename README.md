Makers Academy
===============
Vagrant VM box for Makers Academy projects.


Preliminaries: VirtualBox and Vagrant
-------------------------------------
[Vagrant][vagrant] is a tool to "create and configure lightweight, reproducible,
and portable development environments." Vagrant itself is a virtual instance
creation and startup tool on top of Oracle VirtualBox which takes care of the
virtualisation.

Download and install the Open Source Edition of VirtualBox from [virtualbox].

Then download and install Vagrant from [vagrant]. The Linux packages install
the `vagrant` executable at `/opt/vagrant/bin` and you will need to add this to
your path.


Starting the Vagrant
--------------------
To build and start the Vagrant, run:

    vagrant up

This will build and install the Vagrant if it hasn't been created already. This
step may take some time the first time it is run. When it is complete, you can
SSH onto the instance using:

    vagrant ssh

The directory containing the `Vagrantfile` is available on the Vagrant under
`/vagrant`:

    cd /vagrant

When finished, you should exit the SSH session and destroy the VM:

    vagrant destroy


Connecting to PostgreSQL
------------------------
The Vagrant comes with a PostgreSQL database installed. Connect as the
`postgres` user which is set to have password `password`. There is a
pre-existing database named `postgres` than can be used for developing.

When connecting, you may also need to use host `localhost` and port `5432`.
This port is exported from Vagrant to the machine on which the Vagrant is
running so should be available there too.

Running Rails
--------------

You need to bind to host 0.0.0.0

```
rails s -b 0.0.0.0
```


Running Sinatra
---------------
Port 4567 is exported from the Vagrant to the machine running the Vagrant so
Sinatra applications can be run on the Vagrant and available on the machine
hosting the Vagrant.

The Sinatra application needs to be started with settings to listen on the
external interface. e.g. start the application using:

    rackup -p4567 --host 0.0.0.0


Backing up a Vagrant Box
------------------------
To take a copy of a Vagrant box, e.g. for backup or quick restore, you need
to export the package:

    vagrant package --output MakersAcademy.box

To restore an previously created box, first delete any existing box with
the same name in your Vagrant environment:

    vagrant box remove MakersAcademy virtualbox

And then up the instance without rerunning the installation provisioning:

    vagrant up --no-provisioning MakersAcademy


Vagrant Commmands
-----------------
* `vagrant suspend`: Disable the virtual instance. The allocated disc space
  for the instance is retained but the instance will not be available. The
  running state at suspend time is saved for resumption.
* `vagrant resume`: Wake up a previously suspended virtual instance.
* `vagrant halt`: Turn off the virtual instance. Calling `vagrant up` after
  this is the equivalent of a reboot.
* `vagrant up --no-provision`: Bring up the virtual instance without doing
  the provisioning step. Useful if the provisioning step is destructive.
* `vagrant destroy`: Hose your virtual instance, reclaiming the allocated disc
  space.
* `vagrant provision`: Rerun puppet or chef provisioning on the virtual instance.


Vagrant SSH X Forwarding
------------------------
X applications on VMs can be displayed on the host machine by specifying a
Vagrant SSH connection with X11 forwarding in the `Vagrantfile`:

    config.ssh.forward_x11 = true

On the host machine, add an `xhost` for the Vagrant VM:

    xhost +10.0.0.2

Then X applications started from the VM should display on the host machine.


Vagrant Troubleshooting
-----------------------
To see more verbose output on any vagrant command, add a VAGRANT_LOG environment
variable setting, e.g.:

    VAGRANT_LOG=INFO /opt/vagrant/bin/vagrant up

Further help troubleshooting can be obtained by editing your `Vagrantfile` and
enabling the `config.vm.boot_mode = :gui` setting. This will pop up a VirtualBox
GUI window on boot.

There have been some issues getting 64bit instances to start. The error is
apparent in GUI boot:

    VT-x/AMD-V hardware acceleration has been enabled, but is not
    operational. Your 64-bit guest will fail to detect a 64-bit CPU and
    will not be able to boot.

Some BIOS setting changes can help. The changes are described at
`http://dba-star.blogspot.com/2011/11/how-to-enable-vtx-and-vtd-on-hp-compaq.html`
but briefly:

1. Restart your host machine.
2. Press F10 for BIOS settings at the boot splash.
3. Edit Security -> System Security
4. Enable VT-x and VT-d settings.
5. Save and exit.


[virtualbox]: https://www.virtualbox.org/wiki/Downloads
[vagrant]: http://vagrantup.com
[skel]: https://github.com/DataMinerUK/skel
