## **How to turn off hardware preftechers in linux?**

First, make sure **MSR** kernel module is installed. If the module is not installed, you should select this module in menuconfig and recompile the kernel.

Install msr-tools:
```bash
sudo apt install msr-tools
```

To turn off the hardware prefetchers, bits 0-3 in 0x1A4 should be set to 1. 

```bash
sudo wrmsr -p 0 0x1a4 15 //for core 0
sudo wrmsr -p 1 0x1a4 15 //for core 1
sudo wrmsr -p 2 0x1a4 15 //for core 2
.
.
.
sudo wrmsr -p 7 0x1a4 15 //for core 7
```
To turn them on again, write 0 in 0x1A4 for each core. 
```bash
sudo wrmsr -p 0 0x1a4 0 //for core 0
sudo wrmsr -p 1 0x1a4 0 //for core 1
sudo wrmsr -p 2 0x1a4 0 //for core 2
.
.
.
sudo wrmsr -p 7 0x1a4 0 //for core 7
```
