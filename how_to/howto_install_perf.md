## **How to install and use the perf tool in linux?**
The simplest way to install perf is to download a linux kernel image, extract that, cd to `tools/perf/`. compile it via `sudo make`. 
Then use `sudo cp perf /usr/bin` to copy the perf program to the bin folder, so it will be accessible from anywhere.  
```bash
wget https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/snapshot/linux-5.17.tar.gz
tar -xf linux-5.17.tar.gz
cd linux-5.17/tools/perf
sudo make
sudo install
sudo cp perf /usr/bin/
```
