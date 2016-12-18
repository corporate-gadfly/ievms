Overview
========

Microsoft provides virtual machine disk images to facilitate website testing 
in multiple versions of IE, regardless of the host operating system. 
With a single command, you can have IE8,
IE9, IE10, IE11 and MSEdge running in separate virtual machines.


Quickstart
==========

Just paste this into a terminal: 

    curl -s https://raw.githubusercontent.com/esaron/ievms/master/ievms.sh | bash


Requirements
============

* VirtualBox > 5.0 (http://virtualbox.org), select 'command line utilities' during installation
* Curl (Ubuntu: `sudo apt-get install curl`)
* Linux Only: unar (Ubuntu: `sudo apt-get install unar`)
* Patience

**NOTE** Use [ievms version 0.2.1](https://github.com/esaron/ievms/raw/v0.2.1/ievms.sh) for VirtualBox < 5.0.


Installation
============

**1.)** Install [VirtualBox](http://virtualbox.org) and check the [Requirements](#requirements)

**2.)** Download and unpack ievms:

   * To install IE versions 8, 9, 10, 11 and EDGE use:

        curl -s https://raw.githubusercontent.com/esaron/ievms/master/ievms.sh | bash

   * To install specific IE versions (IE9 and EDGE only for example) use:

        curl -s https://raw.githubusercontent.com/esaron/ievms/master/ievms.sh | env IEVMS_VERSIONS="9 EDGE" bash

**3.)** Launch Virtual Box.

**4.)** Choose ievms image from Virtual Box.

The OVA images are massive and can take hours or tens of minutes to 
download, depending on the speed of your internet connection. You might want
to start the install and then go catch a movie, or maybe dinner, or both. 


Recovering from a failed installation
-------------------------------------

Each version is installed into `~/.ievms/` (or `INSTALL_PATH`). If the installation fails
for any reason (corrupted download, for instance), delete the appropriate ZIP/ova file
and rerun the install.

If nothing else, you can delete `~/.ievms` (or `INSTALL_PATH`) and rerun the install.


Specifying the install path
---------------------------

To specify where the VMs are installed, use the `INSTALL_PATH` variable:

    curl -s https://raw.githubusercontent.com/esaron/ievms/master/ievms.sh | env INSTALL_PATH="/Path/to/.ievms" bash


Passing additional options to curl
----------------------------------

The `curl` command is passed any options present in the `CURL_OPTS` 
environment variable. For example, you can set a download speed limit:

    curl -s https://raw.githubusercontent.com/esaron/ievms/master/ievms.sh | env CURL_OPTS="--limit-rate 50k" bash


Disk requirements
-----------------

A full ievms install will require approximately 65G:

    Servo:.ievms esaron$ du -ch *
     11G    IE10 - Win7-disk1.vmdk
     22M    IE10-Windows6.1-x86-en-us.exe
     11G    IE11 - Win7-disk1.vmdk
     28M    IE11-Windows6.1-x86-en-us.exe
    1.6G    IE8 - WinXP-disk1.vmdk
     16M    IE8-WindowsXP-x86-ENU.exe
     11G    IE9 - Win7-disk1.vmdk
    4.7G    IE9 - Win7.ova
    4.7G    IE9_Win7.zip
     10G    MSEdge - Win10-disk1.vmdk
    5.1G    MSEdge - Win10.ova
    5.0G    MSEdge_Win10.zip
    3.4M    ievms-control-0.3.0.iso
    4.6M    lsar
    4.5M    unar
    4.1M    unar1.5.zip
    ~65G    total
   
You may remove all files except `*.vmdk` after installation and they will be
re-downloaded if ievms is run again in the future:

    $ find ~/.ievms -type f ! -name "*.vmdk" -exec rm {} \;

If all installation related files are removed, around 45G is required:

    Servo:.ievms esaron$ du -ch *
     11G    IE10 - Win7-disk1.vmdk
     11G    IE11 - Win7-disk1.vmdk
    1.6G    IE8 - Win7-disk1.vmdk
     11G    IE9 - Win7-disk1.vmdk
     10G    MSEdge - Win10-disk1.vmdk
    ~45G    total


Bandwidth requirements
----------------------

A full installation will download roughly 12.5G of data.

**NOTE:** Did not verify this after fixing repo! It's possible this is no
longer the case, as most of the vms are now Win7. Reusing the Win7 VM (the
default), saves tons of bandwidth but pretty much breaks even on disk space.
Disable it with `REUSE_WIN7=no`.


Features
========


Clean Snapshot
--------------

A snapshot is automatically taken upon install, allowing rollback to the
pristine virtual environment configuration. Anything can go wrong in 
Windows and rather than having to worry about maintaining a stable VM,
you can simply revert to the `clean` snapshot to reset your VM to the
initial state.


Guest Control
-------------

VirtualBox guest additions are installed after each virtual machine is created
(and before the clean snapshot) and the appropriate steps are taken to enable
guest control from the host machine.


Resuming Downloads
------------------

~~If one of the comically large files fails to download, the `curl` 
command used will automatically attempt to resume where it left off.~~
Unfortunately, the modern.IE download servers do not support resume.

Reusing Win7 VMs
----------------

Currently there exists a [bug](https://www.virtualbox.org/ticket/11134) in 
VirtualBox (or possibly elsewhere) that disables guest control after a Windows 8
virtual machine's state is saved. To better support guest control and to
eliminate yet another image download, ievms will re-use the IE9 Win7 image for
IE11 by default. In addition, the Win7 VMs are the only ones which can
be successfully "rearmed" to extend the activation period.

**NOTE:** If you'd like to disable Win7 VM reuse for IE11, set the environment 
variable `REUSE_WIN7` to anything other than `yes`:

    curl -s https://raw.githubusercontent.com/esaron/ievms/master/ievms.sh | REUSE_WIN7="no" bash

Control ISO
-----------

Microsoft's XP image uses a blank password for the `IEUser`, which disallows
control via Virtualbox's guest control by default. Changing a value in the
Windows registry enables guest control, but requires accessing the VM's hard
drive. A solution is to boot the VM with a special boot CD image which attaches
the hard disk and edits the registry. A custom linux build has been created
based on [the ntpasswd bootdisk](http://pogostick.net/~pnh/ntpasswd/) which
makes the required registry edits and simply powers off the machine. The ievms
script may then use Virtualbox guest controls to manage the VM.

The control ISO is built within a [Vagrant](http://vagrantup.com) Ubuntu VM.
If you'd like to build it yourself, clone the ievms repository, install
Vagrant and run `vagrant up`. The base ntpasswd boot disk will be downloaded, 
unpacked and customized within the Vagrant VM. A custom linux kernel is 
cross-compiled for the image as well.


Acknowledgements
================

* [modern.IE](http://modern.ie) - Provider of IE VM images.
* [ntpasswd](http://pogostick.net/~pnh/ntpasswd/) - Boot disk starting point
and registry editor.
* [regit-config](https://github.com/regit/regit-config) - Minimal Virtualbox
kernel config reference.
* [uck](http://sourceforge.net/projects/uck/) - Used to (re)master control ISO.
* [xdissent](http://github.com/xdissent/ievms) - The original maintainer of the
ievms project

License
=======

None. (To quote Morrissey, "take it, it's yours")
