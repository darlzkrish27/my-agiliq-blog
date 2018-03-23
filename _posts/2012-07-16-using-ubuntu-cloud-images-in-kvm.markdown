---
layout: post
title:  "Using Ubuntu cloud images in KVM"
date:   2012-07-16 00:00:00
author:   thejaswi
tags:   virtualization
---

Quite a few of our clients are powered by Amazon EC2 or Rackspace and we
use Ubuntu LTS releases for our servers. Canonical
[provides](http://cloud-images.ubuntu.com/) EC2 AMIs and Openstack
images for all their releases. By using these [JeOS
images](https://en.wikipedia.org/wiki/Just_enough_operating_system) on
the server as well as on the development platform reduces the dev/prod
parity which we discussed in a
[previous](http://agiliq.com/blog/2012/06/libvirt-and-kvm/) post.

In this post, we\'ll see how to setup the Ubuntu cloud images in the
local KVM hypervisor. Unlike the previous post, we\'ll use only the
virt-manager to do the initial provisioning but you can be assured that
there is a command line way to do all of the same.

Head over to [Ubuntu cloud images](http://cloud-images.ubuntu.com/) and
select the appropriate release (or daily build). At the time of writing
the article, 12.04 is the latest LTS release and can be downloaded from
[here](http://cloud-images.ubuntu.com/releases/precise/release/). I have
downloaded the 64-bit (or amd64) images
(ubuntu-12.04-server-cloudimg-amd64-root.tar.gz) since I use a 64-bit
machine for my development and the servers are all 64-bit file. This
compressed file contains a virtual hard disk, virtual floppy disk and a
kernel.

Let us now use this virtual hard disk as a base for our images and
create new guest VM based on this disk.:

    $ qemu-img create -b ubuntu-12.04-server-cloudimg-amd64.img -f qcow2 new_vm.img

The [qemu-img](https://en.wikibooks.org/wiki/QEMU/Images) command
creates a new\_vm.img image with qcow2 format using the
ubuntu-12.04-server-cloudimg-amd64.img as the backing image.

Right click on the connection in virt-manager and select New and provide
a name for the new virtual machine and select the
Import existing disk image option. In the next step, provide the path of
the image just created (ie new\_vm.img) and optionally select the OS
type and version. In the third step, select the RAM allocated to the VM
and the number of cores and in the final step check the
Customize configuration before install before filling other options.

In the Customization screen, click on the section that says Disk 1 and
set the storage format to qcow2 and apply the changes.

> [![Configuring the disk](http://agiliq.com/static/dumps/images/20120716/disk_configuration.png){.align-center
> width="90.0%"}](http://agiliq.com/static/dumps/images/20120716/disk_configuration.png)

Now all that is left is to set the boot order and there are two methods:

-   Using the floppy disk image
-   Using cloud-init on the hard disk image

Using the floppy disk for booting
---------------------------------

If you don\'t see a floppy disk in the list of devices in the
customization screen, click on Add Hardware and add a new Storage device
of type Floppy disk and check the
Select managed or other existing storage and provide the path to the
floppy disk image from the download and set the Storage format to raw.

> [![Adding Floppy Disk](http://agiliq.com/static/dumps/images/20120716/floppy_disk_create.png){.align-center
> width="90.0%"}](http://agiliq.com/static/dumps/images/20120716/floppy_disk_create.png)

Select the Boot Options section in the customization screen after
creating the floppy disk and select the Floppy in the Boot device order.
Start the installation and after that run the VM and you are done!

After booting the new VM, you should see the GRUB screen in a few
seconds as below.

> [![Grub Screen](http://agiliq.com/static/dumps/images/20120716/grub_screen.png){.align-center
> width="90.0%"}](http://agiliq.com/static/dumps/images/20120716/grub_screen.png)

Choose the option that you prefer and you will be redirected to the
login prompt after the bootup.

This method is useful if you have a provisioning or a metadata service
like [Orchestra](https://help.ubuntu.com/community/Orchestra/Overview)
or [Cobbler](http://cobbler.github.com/) though not a compulsion.

Using cloud-init on the hard disk
---------------------------------

[CloudInit](https://help.ubuntu.com/community/CloudInit) is an init
script that performs some basic configuration and house keeping tasks on
guest VMs like setting the hostname, generating SSH keys etc

Go to the Boot Options section and select the Hard disk from the
Boot device order and change the Direct kernel boot sub section.

Provide the kernel file (the file ending with vmlinuz-virtual) from the
download in the Kernel path and the following values in the \`Kernel
arguments\`:

    ro init=/usr/lib/cloud-init/uncloud-init root=/dev/vda ds=nocloud ubuntu-pass=initialpassword

[![Configuring the hard disk](http://agiliq.com/static/dumps/images/20120716/hard_disk_boot.png){.align-center
width="90.0%"}](http://agiliq.com/static/dumps/images/20120716/hard_disk_boot.png)

Start the installation and in a few moments you have a brand new guest
VM.

You now have exactly the same packages as an official and freshAmazon
AMI or Rackspace image and so you don\'t have to break your head
worrying about the dependency hell and enjoy more time developing.
