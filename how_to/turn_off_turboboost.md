## **How to turn on/off turbo boost in linux?**

First, make sure **MSR** kernel module is installed. If the module is not installed, you should select this module in menuconfig and recompile the kernel.

Install msr-tools:
```bash
sudo apt install msr-tools
```

Use the following script. (Credit: https://askubuntu.com/questions/619875/disabling-intel-turbo-boost-in-ubuntu)
```bash
#!/bin/bash

if [[ -z $(which rdmsr) ]]; then
    echo "msr-tools is not installed. Run 'sudo apt-get install msr-tools' to install it." >&2
    exit 1
fi

if [[ ! -z $1 && $1 != "enable" && $1 != "disable" ]]; then
    echo "Invalid argument: $1" >&2
    echo ""
    echo "Usage: $(basename $0) [disable|enable]"
    exit 1
fi

cores=$(cat /proc/cpuinfo | grep processor | awk '{print $3}')
for core in $cores; do
    if [[ $1 == "disable" ]]; then
        sudo wrmsr -p${core} 0x1a0 0x4000850089
    fi
    if [[ $1 == "enable" ]]; then
        sudo wrmsr -p${core} 0x1a0 0x850089
    fi
    state=$(sudo rdmsr -p${core} 0x1a0 -f 38:38)
    if [[ $state -eq 1 ]]; then
        echo "core ${core}: disabled"
    else
        echo "core ${core}: enabled"
    fi
done
```
save above script as turbo-boost.sh and make it executable via:
```bash
sudo chmod +x turbo-boost.sh
```
now you can turn on/off turbo boost by the following commands:
```c
./turbo-boost.sh disable
./turbo-boost.sh enable
```
