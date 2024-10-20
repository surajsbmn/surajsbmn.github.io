+++
title= "Ubuntu Cloud Init Template in Proxmox"
date= "2023-05-29"
tags = ["Virtual Machines", "templates", "Proxmox", "Homelab"]
draft = false
+++

A template is a fully pre-configured operating system image that can used to deploy KVM virtual machines. Creating a special template is usually preferred over cloning an existing VM.
Deploying virtual machines from templates is blazing fast, very comfortable and if you use linked clones you can optimize your storage by using base images and thin-provisioning. 

#### Steps to create Ubuntu cloud init template in Proxmox

Choose your [Ubuntu Cloud Image](https://cloud-images.ubuntu.com/) (replace with the url of the one you chose from above)

```bash
wget https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-amd64.img
```

Create a new virtual machine. Here 9000 is just a VM id for the template, you can set it to anything you want but make sure change the id in other commands. The memory and cpu you set will be the initial value of the clone and can be increased later.

```bash
qm create 9000 --memory 2048 --core 2 --name ubuntu-cloud --net0 virtio,bridge=vmbr0
```

Import the downloaded Ubuntu disk to local-lvm storage

```bash
qm importdisk 9000 jammy-server-cloudimg-amd64.img local-lvm
```

Attach the new disk to the VM as a scsi drive on the SCSI controller

```bash
qm set 9000 --scsihw virtio-scsi-pci --scsi0 local-lvm:vm-9000-disk-0
```

Add cloud init drive

```bash
qm set 9000 --ide2 local-lvm:cloudinit
```

Make the cloud init drive bootable and restrict BIOS to boot from disk only

```bash
qm set 9000 --boot c --bootdisk scsi0
```

Add serial console

```bash
qm set 9000 --serial0 socket --vga serial0
```

**DO NOT START YOUR VM**

Now, configure hardware and cloud init, then create a template and clone. If you want to expand your hard drive you can on this base image before creating a template or after you clone a new machine.

Create template.

```bash
qm template 9000
```

Clone template.

```bash
qm clone 9000 150 --name myclone --full
```
