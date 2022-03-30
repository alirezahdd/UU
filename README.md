## **How to run a Kernel-based Virtual Machine (KVM) via linux command line and _ssh_ to it?** 
in this simple _step-by-step_ tutorial we are going to build and run a kvm via linux command line, without having/using any graphical interface. For that, two things are required:   
1. [An ext4 disk image on which linux (Debian) rootfs is installed](#1-preparing-disk-image).  
2. [A linux kernel bzImage](#2-preparing-kernel-image).  

### **1. Preparing Disk Image**
We create raw 20G disk image, named `ubuntu_18.04.img` (you can name it whatever you want!), and format it with the _ext4_ file system type.
```bash
qemu-img create -f raw ubuntu_18.04.img 20G     
mkfs.ext4 ubuntu_18.04.img                      
```
To install linux rootfs on the created disk image, we have to _mount_ the image on a folder and then install the desired version of rootfs. So now we create a folder, named `mnt` (of course again you can name it whatever you want), and _mount_ the disk image on it.
```bash
mkdir mnt
sudo mount -o loop ubuntu_18.04.img mnt
```
Now we have access to the disk image using `mnt` folder. We now install rootfs on the disk image using `debootstrap` tool. We use `--include=ssh,vim` to have both `ssh` and `vim` installed on our virtual machine. We also can specify any debian release by passing its name (`bionic` ubuntu 18.04 in our example).
```bash
sudo debootstrap --arch amd64 --include=ssh,vim bionic mnt
```
Now we should add a user (we need this to logging in via ssh to the virtual machine) to the system.
```bash
sudo chroot mnt
passwd
adduser USERNAME
usermod -aG sudo USERNAME
exit
```
The image is ready. Let's _unmount_ it.
```bash
sudo umount mnt
```
### **2. Preparing Kernel Image**
