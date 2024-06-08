To partition an SSD as a swap partition in Linux, you can follow these steps:

1. Open a terminal or access the command-line interface on your Linux system.

2. Identify the SSD device you want to partition by using the lsblk command. It will display a list of available storage devices. Note the name of your SSD device (e.g., /dev/sdX), where 'X' represents the specific drive letter assigned to your SSD.

3. Run the sudo fdisk /dev/sdX command, replacing "sdX" with the actual device name of your SSD.

4. In the fdisk interface, type n and hit Enter to create a new partition.

5. Choose the partition type. For a swap partition, type p for primary partition or e for an extended partition (if you have already used the maximum number of primary partitions). Press Enter to continue.

6. When prompted for the partition number, accept the default value by pressing Enter.

7. Specify the partition size. For a swap partition, you can allocate the desired size. If you want to use the entire disk as a swap partition, press Enter to allocate the maximum available size.

8. Set the partition type to Linux swap by typing t and pressing Enter. Enter the partition number if prompted.

9. Choose the partition type code for Linux swap. Type 82 and press Enter.

10. Save the changes and exit fdisk by typing w and pressing Enter. This will write the partition table to the disk.

11. Format the newly created swap partition with the mkswap command. Run sudo mkswap /dev/sdX1, replacing "sdX1" with the actual partition name of your swap partition.

12. Activate the swap partition using the swapon command. Run sudo swapon /dev/sdX1, again replacing "sdX1" with the correct partition name.

13. Verify that the swap partition is active by running the swapon --show command. It should display information about the swap partition, including its size.

14. Optionally, you can add an entry for the swap partition in the /etc/fstab file to ensure it is automatically mounted during system startup. Open the file using a text editor (e.g., sudo nano /etc/fstab`) and add a line like this: `/dev/sdX1 none swap sw 0 0, replacing "sdX1" with the appropriate partition name.

That's it! You have successfully partitioned your SSD as a swap partition in Linux. The swap partition will be used by the system for virtual memory, providing additional memory resources when needed.
