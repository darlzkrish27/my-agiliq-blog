---
layout: post
title:  Libvirt and KVM
date:   2012-06-25 00:00:00
author:   thejaswi
tags:   virtualization
---

Libvirt and KVM
---------------

*\"But it works on my local setup!\"* We have heard or probably said
this tens of times after something that we deployed to the production
server breaks. After fire fighting for hours we learn that a particular
package\'s version varies from the local setup. This is a fairly common
problem that plagues every developer. Off late, quite a lot of interest
and work is going into maximizing the [dev/prod
parity](http://www.12factor.net/dev-prod-parity) to prevent such
problems.

Software like virtualenv help a great deal in compartmentalizing python
dependencies but most web applications nowadays have to deal with loads
of other dependencies like web servers, application servers etc.

We use Ubuntu at work (not a rigid requirement) and they have biannual
releases with long term releases LTS every two years. We use LTS
releases for our servers and the general releases as our development
desktops.

In such a case, how do we reduce the parity with our server setup? This
was already explained in a blog post
[earlier](http://agiliq.com/blog/2012/01/brief-overview-vagrant/) by
Dheeraj. While Vagrant is a fine piece of software, Virtualbox on Linux
based operating systems tends to be very problematic. There are a lot of
random crashes, kernel dumps, disk corruptions etc and you spend a lot
of time repairing it rather than concentrating on developing. This is
where libvirt and KVM come in.

[Libvirt](http://libvirt.org/) is a common API that helps manage
virtualization platforms (or hypervisors in it\'s terminology). Through
this single API, you will be able to talk to a host of hypervisors like
Virtualbox, KVM, Xen, LXC, OpenVZ, VMware based hypervisors and also
Microsoft Hyper-V. Libvirt also provides for network and disk
management, authentication and access control etc.

[KVM](http://linux-kvm.org/) (Kernel Virtual Machine) is a Linux kernel
module (requires no compilation like Xen) that supports native
(hardware) virtualization. KVM based guests almost perform as well as
the host and since it is part of the mainline kernel since 2.6.20, you
may consider it the \'official\' virtualization platform on most Linux
based distros.

[Qemu](http://qemu.org/) is a userspace program that talks to the KVM
module and emulates some hardware devices.

A guest (or client/domain as some may refer) is a virtual machine
running on top of a host (or server).

So far we have just dealt with the definitions, let\'s install something
to play around with. But before that, here\'s the customary disclaimer.

::: {.note}
::: {.admonition-title}
Note
:::

KVM works **only** on Linux based distributions and virtualization ready
hardware.
:::

Installing software on ubuntu is fairly easy thanks to apt-get.

-   Let\'s install libvirt and qemu-kvm (the userspace program mentioned
    above) first:

        $ sudo apt-get install libvirt-bin qemu-kvm

-   You will have to add the user who can access libvirt to the libvirtd
    group:

        $ sudo adduser <username> libvirtd

    If you have added the current user (specified by \<username\>), log
    out and login to have the groups addition to get refreshed:

        $ groups <username>
        <username> : <username> libvirtd

-   After you have successfully installed libvirt, check if virsh
    (virtualization shell) works fine:

        $ virsh
        virsh # 

-   Now, you may create your guest VM in either of the two ways
    specified below, through the CLI tool (virsh) or the graphical
    interface (virt-manager).
-   If you prefer a graphical interface over \`virsh\`:

        $ sudo apt-get install virt-manager

-   If you are using virsh (**note:** not applicable if you are using
    virt-manager), you need the virt-install package to create a new
    guest and virt-viewer to view the VNC console:

        $ sudo apt-get install python-virtinst virt-viewer
        $ sudo virt-install -n guest_vm -r 512 --disk path=/path/to/store/vm,size=1 -c /path/to/guest_vm.iso -v --virt-type=kvm --connect=qemu:///system --vnc

    The virt-install command creates a VM with the name guest\_vm,
    assigns it 512 MB RAM and creates an associated disk using the
    arguments provided to \--disk path (size 1GB here). For
    installation, it makes use of the ISO file provided. This ISO may be
    any OS image that you may have downloaded or generated. The -v
    option instructs to make the guest \'fully\' virtualized with the
    \'kvm\' hypervisor and start the VNC console. There are a multitude
    of options (like setting the network etc) and you may want to check
    the man page.

-   In virt-manager (**note:** not applicable if you are using virsh),
    go to File \> Add Connection and set the Hypervisor as Qemu (or KVM)
    and save the connection.

    [![Virt-manager screen](http://agiliq.com/static/dumps/images/20120625/libvirt_screen_1.png){.align-center
    width="90.0%"}](http://agiliq.com/static/dumps/images/20120625/libvirt_screen_1.png)

    Then, let\'s create a guest VM. Right click on the just created
    connection in the host summary window and create a new VM by right
    clicking on your connection and selecting \'New\'.

    [![Creating a new VM](http://agiliq.com/static/dumps/images/20120625/libvirt_screen_2.png){.align-center
    width="90.0%"}](http://agiliq.com/static/dumps/images/20120625/libvirt_screen_2.png)

    Select the method of your choice to install a new VM. I selected the
    Local install
    media (ISO image or CDROM) option, set the name of the VM as
    guest\_vm and provided the path to the ISO image in the next screen.
    The steps 3 and 4 are fairly straightforward and you can safely use
    the defaults. In the step 5, you may modify the network (the default
    should be fine) and have the ability to customize the hardware
    emulated by the hypervisor. I prefer not to fiddle with those and
    click on Finish and wait for the VM to get created.

-   Now we are done creating a new guest VM and installing it. We can
    start the guest using virsh (**note**: Not applicable if you are
    using virt-manager):

        $ virsh start guest_vm
        $ virt-viewer -c qemu:///system guest_vm

-   If you want to start the VM using virt-manager, right click on the
    VM and click on Run.

    [![Running a VM](http://agiliq.com/static/dumps/images/20120625/libvirt_screen_3.png){.align-center
    width="90.0%"}](http://agiliq.com/static/dumps/images/20120625/libvirt_screen_3.png)

-   After running the VM, right click on the running VM and select Open
    to view the console.

    [![Running a VM](http://agiliq.com/static/dumps/images/20120625/libvirt_screen_4.png){.align-center
    width="90.0%"}](http://agiliq.com/static/dumps/images/20120625/libvirt_screen_4.png)

So now we have guest VMs running and a way to install an OS of our
choice. For example, you can install the LTS release of ubuntu in a VM
and use it for development or Windows XP (or Windows 7) to test against
Internet Explorer or create one VM per project and manage your
dependencies easily.
