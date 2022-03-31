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
To install linux rootfs on the created disk image, we have to _mount_ the image on a folder (so we can access it) and then install the desired version of rootfs. So now we create a folder, named `mnt` (of course again you can name it whatever you want), and _mount_ the disk image on it.
```bash
mkdir mnt
sudo mount -o loop ubuntu_18.04.img mnt
```
Now we have access to the disk image using `mnt` folder. We now install rootfs on the disk image using `debootstrap` tool. We use `--include=ssh,vim` to have both `ssh` and `vim` installed on our virtual machine. We also can specify any debian release by passing its name (`bionic` ubuntu 18.04 in our example).
```bash
sudo debootstrap --arch amd64 --include=ssh,vim bionic mnt
```
Now we should add a user (we need this to logging in via ssh to the virtual machine) to the created partition. For that, we first need to change Root to the mnt directory (using `chroot` command). After adding a user, We exit chroot by `exit` command. 
```bash
sudo chroot mnt
passwd
adduser USERNAME
usermod -aG sudo USERNAME
exit
```
The image is ready. Let's _unmount_ it from mnt directory.
```bash
sudo umount mnt
```
### **2. Preparing Kernel Image**
In this part, a linux kernel source should be downloaded, configured, and compiled.  
Browse [https://kernel.org/pub/linux/kernel](https://kernel.org/pub/linux/kernel), find your desired release, copy the address of a linux kernel source _tarball_ (\*.tar.gz), download it via `wget` command, and extract it via `tar`.
```bash
wget https://mirrors.edge.kernel.org/pub/linux/kernel/v5.x/linux-5.14.1.tar.gz
tar -xf linux-5.14.1.tar.gz
```
To configure the kernel easily, go to the kernel directory, configure it via `menuconfig`.
```bash
cd linux-5.14.1/
sudo make menuconfig
```
You can select/include your desired modules. For network connection make sure you include `Device Drivers -> Virtio drivers` and `Device Drivers -> Network device support -> Virtio netwrok driver` by pressing `y`.
Once the kernel is configured, compile it via `make`. To speed up the compilation, add -jN, while N is the number of cores dedicated for the compilation:
```bash
sudo make -j8
```
If everything goes well without errors, you can find the compiled kernel image under `arch/x86_64/boot/bzImage`. If there is no image there, something has gone wrong during the compilation.  
So far, we have collected requirements for the virtual machine. cd back to the directory where disk image exists and make the virtual machine via `qemu-system-x86_64`:  
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
However, your virtual machine is not yet connected to the network. To connect it, use `ip a` command and find the name of your ethernet interface (it is usually named eth1000, ens, etc.). My ethernet interface was named ens3. We should create a file named `50-cloud-init.yaml` under `/etc/netplan/` directory.
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
if everything goes well, your vm is now connected to the network and can be reached using ssh. You should now be able to `ping 8.8.8.8`.
In order to ssh to the created virtual machine, open another terminal/commandline and run the following command (note that we defined port 5556 when we created our virtual machine using `qemu-system-x86_64`):  
```bash
ssh USERNAME@localhost -p 5556
```
Enter your password and enjoy.
