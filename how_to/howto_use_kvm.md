## **How to make and run a Kernel-based Virtual Machine (KVM) via linux command line and _ssh_ to it?** 
First of all, I took the material and commands from https://chpresearch.wordpress.com/2018/10/04/making-simple-kvm-image/ and made it a little bit more clear for newbees.  
  
  
Secondly, there should be several ways to this. I've chosen this way, since I wanted to create a sandbox environment for kernel hacking. It might be easier to change the kernel source, compile it and run the vm with new kernel image.
  
### Let's begin
in this simple _step-by-step_ tutorial we are going to build and run a kvm via linux command line, without having/using any graphical interface, and ssh to it. This requires four steps:   
1. [Preparing an ext4 disk image on which linux (Debian) rootfs is installed](#1-preparing-disk-image).  
2. [Preparing a linux kernel image (bzImage)](#2-preparing-kernel-image).
3. [Running the virtual machine using the disk and kernel images](#3-running-the-virtual-machine)
4. [Setting up the virtual machine's Network](#4-setting-up-the-network-connection-static-ip)  

### **1. Preparing Disk Image**
We create raw 20G disk image, named `ubuntu_18.04.img` (you can name it whatever you want!), and format it with the _ext4_ file system type.
```bash
qemu-img create -f raw ubuntu_18.04.img 20G     
mkfs.ext4 ubuntu_18.04.img                      
```
To install linux rootfs on the created disk image, we have to _mount_ the image on a folder (so we can access it) and then install the desired version of rootfs. So now we create a folder, named `mnt` (of course again you can name it whatever you want), and _mount_ the disk image on it.
```bash
mkdir mnt
sudo mount -o loop ubuntu_18.04.img mnt
```
Now we have access to the disk image using `mnt` folder. We now install rootfs on the disk image using `debootstrap` tool. We use `--include=ssh,vim` to have both `ssh` and `vim` installed on our virtual machine. We also can specify any debian release by passing its name (`bionic` ubuntu 18.04 in our example).
```bash
sudo debootstrap --arch amd64 --include=ssh,vim bionic mnt
```
Now we should add a user (we need this for logging in into the virtual machine) to the created partition. For that, we first need to change Root to the mnt directory (using `chroot` command), add a user, make the user a sudoer, and exit the chroot by `exit` command. 
```bash
sudo chroot mnt
passwd
adduser USERNAME
usermod -aG sudo USERNAME
exit
```
The disk image is ready. Let's _unmount_ it from mnt directory.
```bash
sudo umount mnt
```
### **2. Preparing Kernel Image**
In this part, a linux kernel source should be downloaded, configured, and compiled.  
Browse [https://kernel.org/pub/linux/kernel](https://kernel.org/pub/linux/kernel), find your desired release, copy the its address (linux kernel source _tarball_ \*.tar.gz), download it via `wget` command, and extract it via `tar -xf` command.
```bash
wget https://mirrors.edge.kernel.org/pub/linux/kernel/v5.x/linux-5.14.1.tar.gz
tar -xf linux-5.14.1.tar.gz
```
To configure the kernel easily, go to the kernel directory, configure it via `make menuconfig`.
```bash
cd linux-5.14.1/
sudo make menuconfig
```
You can select/include your desired modules. In order to have a network connection, make sure you include:  
* `Device Drivers -> Virtio drivers`  
* `Device Drivers -> Network device support -> Virtio netwrok driver`  

by pressing `y`.
Once the kernel is configured, compile it via `make`. To speed up the compilation, add -jN, where N is the number of cores dedicated for the compilation:
```bash
sudo make -j8
```
If everything goes well without errors, you can find the compiled kernel image under `arch/x86_64/boot/bzImage`. If there is no image there, something has gone wrong during the compilation.
### **3. Running the Virtual Machine**
So far, we have collected requirements for the virtual machine. cd back to the directory where disk image exists and run the virtual machine via `qemu-system-x86_64`:  
```bash
cd ..
sudo qemu-system-x86_64 -m 70G \
-kernel linux-5.14.1/arch/x86_64/boot/bzImage \
-drive file=ubuntu_18.04.img,index=0,media=disk,format=raw \
-append "root=/dev/sda rw transparent_hugepage=never console=ttyS0 no-kvmclock" \
-netdev user,id=hostnet0,hostfwd=tcp::5556-:22,hostname=vm0 \
-device virtio-net-pci,netdev=hostnet0,id=net0,bus=pci.0,addr=0x3 \
--enable-kvm \
--nographic \
-cpu host
```
Now your virtual machine is up and running. Enter the USERNAME and password you set at the begining part of the tutorial to login.  
### **4. Setting up the Network Connection (Static IP)**
Your virtual machine is not yet connected to the network. To connect it, use `ip a` command and find the name of your ethernet interface (it is usually named eth1000, ens, etc.). My ethernet interface was named ens3. We should create a file named `50-cloud-init.yaml` under `/etc/netplan/` directory.
```bash
sudo vim /etc/netplan/50-cloud-init.yaml
```
In the created _yaml_ file type the following (note that instead of `ens3` you should put your ethernet interface name) and save the changes:
```yaml
network:
  ethernets:
    ens3:
      dhcp4: true
```
Now run the following commands to apply the changes to the network:
```bash
sudo netplan try
sudo netplan apply
```
if everything goes well, your vm is connected to the network and can be reached using ssh. You should now be able to `ping 8.8.8.8`.
In order to ssh to the created virtual machine, open another terminal/commandline and run the following command (note that we defined port 5556 when we created our virtual machine using `qemu-system-x86_64`):  
```bash
ssh USERNAME@localhost -p 5556
```
Enter your password and enjoy.  
### Turning the Virtual Machine On and Off
To turn you virtual machine off just type `sudo poweroff`.
To turn it on again, go to the directory where the disk image exists, and type the `qemu-system-x86_64` command used in the [third step](#3-running-the-virtual-machine).
