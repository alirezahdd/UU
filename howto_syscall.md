## **How to write a HelloWorld linux system call?**   
briefly, to write a linux system call and make it working:
1. [A linux kernel source should be downloaded and extracted](#1-preparing-kernel-image).  
2. [A custom system call should be added to the kernel directory](#2-writing-a-system-call).
3. [Some files(e.g., system calls table, Makefile, etc.) should be updated](#3-updating-makefiles-systemcall-tables-etc).  
4. [Then, the linux kernel source should be compiled and installed](#4-compiling-the-kernel).  
  
In this simple step-by-step tutorial we are going to write a system call and use it in a virtual machine.  
### **1. Preparing Kernel Image**
In this part, a linux kernel source should be downloaded and extracted.  
Browse [https://kernel.org/pub/linux/kernel](https://kernel.org/pub/linux/kernel), find your desired release, copy the its address (linux kernel source _tarball_ \*.tar.gz), download it via `wget` command, and extract it via `tar -xf` command. When extracting is finished, cd to the linux directory.
```bash
wget https://mirrors.edge.kernel.org/pub/linux/kernel/v5.x/linux-5.14.1.tar.gz
tar -xf linux-5.14.1.tar.gz
cd linux-5.14.1/
```
### **2. Writing a System Call**
Make sure you are in the linux directoy. Create a directory(via `mkdir FOLDER_NAME` command) in which your custom system call is going to reside. cd into it and create your system call's c source code. 
```bash
mkdir hellofolder
cd hellofolder
sudo vim hello.c
```
There are many ways to write a system call. I chose the simplest one. Make sure you include `linux/kernel.h` and `linux/syscall.h`.  
In order to define your system call, we use `SYSCALL_DEFINEn` macro, where _n_ is the number of arguments (inputs of the system call). The first argument of this macro is name of the system call, and the next two arguments are _type_ and _name_ of the first input. Below you can find examples of using this macro:
```c
SYSCALL_DEFINE0(SYSCALL_NAME){
  //do something
  return 0;
}

SYSCALL_DEFINE1(SYSCALL_NAME, TYPE, ARGUMENT_NAME1){
  //do something
  return 0;
}

SYSCALL_DEFINE3(SYSCALL_NAME, TYPE, ARGUMENT_NAME1, TYPE, ARGUMENT_NAME2, TYPE, ARGUMENT_NAME3){
  //do something
  return 0;
}
```
Considerig examples above, our hello world system call would be like this:
```c
#include <linux/kernel.h>
#include <linux/syscalls.h>

SYSCALL_DEFINE1(hello, char *, msg)
{
  char buffer[256];
  strncpy_from_user(buffer, msg, sizeof(buffer));
  printk(KERN_INFO "your syscall called with \"%s\"\n", buffer);
  return 0;
}
```
Its name is `hello` and accepts a `char*` input, named `msg`. To use the inputs, we have to copy them somewhere first. So we use `strncpy_from_user` function to do this for us. Our hello world system call, prints a kernel message into the kernels log.  
Our system call defenition is completed. In order to introduce it to the kernel, we are going to create or update some files.
### **3. Updating Makefiles, Systemcall Tables, etc.**
First cd to hellofolder, where you created you hello.c file, create a Makefile, and write `obj-y := hello.o` into it.
```bash
cd hellofolder/
sudo vim Makefile
```
and
```makefile
obj-y := hello.o
```
Now cd back to the parent directory and edit the Makefile. To do that, find the line `core-y += kernel/ certs/ mm/ fs/ ipc/ security/ crypto/ block/` and add the folder you created at the end of the line. After the change, the line should look like the line below:
```bash
cd ..
sudo vim Makefile
```
and
```makefile
core-y += kernel/ certs/ mm/ fs/ ipc/ security/ crypto/ block/ hellofolder/
```
At this moment, we should add the new system call to the system call table. For x86 machines, it exist under `arch/x86/entry/syscalls/`. So cd to it, and edit `syscall_64.tbl`. Note that if you have a 32bit system, you should edit `syscall_32.tbl` instead. To add your system call to this table, go the end of the table and add to following line to it:
```bash
cd arch/x86/entry/syscalls/
sudo vim syscall_64.tbl
```
and add the following entry as the last entry of the table:
```
548 64  hello sys_hello
```
Notice that, there are tabs (not spaces) between 548, 64, hello, and sys_hello. The number `548` is the next number after the last entry of the table (which was 547). There should be a `sys_` prefix before the name of your system call in the last column. Now save your change and exit.  
Now, we should add our system call to the system call's header file. To that, cd back to the linux folder and go to `include/linux/` directory and edit the `syscalls.h` file. 
```bash 
cd ../../../..
cd include/linux/
sudo vim syscalls.h
```
Now, at the end the file, before `#endif` add the line bellow
```c
asmlinkage long sys_hello(char * msg);
```
Save and exit.The updates required for introducing the new system call are made now. Let's compile the kernel.
### **4. Compiling The Kernel**
cd back to the linux folder and compile the kernel using `sudo make -jn` where n is the number of your cores.
```bash
cd ../..
sudo make -j8
```
If everything goes well, you'll get a bzImage of your kernel under `arch/x86_64/boot` directory. Using this image you can update your kernel or use it via a virtual machine and enjoy your hello world system call.  
In order to use your system call, write a c program, and use `syscall(548,"THE INPUT");` to call your systemcall.
