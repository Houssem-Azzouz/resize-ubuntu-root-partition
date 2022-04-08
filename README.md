# Resize the root partition of a VirtualBox Ubuntu VM running on Windows

On the Windows host, open a CMD instance and execute the following commands: 

1. Add vboxmanage to path:
```
set PATH=%PATH%;"C:\Program Files\Oracle\VirtualBox"
```

2. Resize disk (from 10G to 15G in this example):
```
vboxmanage modifymedium disk "C:\Users\<user>\VirtualBox VMs\<vm-name>\<disk-name>.vdi" --resize 15000
```

3. Verify resize went well:
```
vboxmanage list hdds
```

Note:
To be able to resize a disk it should be created as a dynamically allocated disk.
You cannot resize of virtual disk that is created as a fixed size disk.
To convert a fixed disk to dynamic, run:
```
vboxmanage clonemedium disk "C:\Users\<user>\VirtualBox VMs\<vm-name>\<disk-name>.vdi" "C:\Users\<user>\VirtualBox VMs\<vm-name>\<disk-NEW-name>.vdi" --variant Standard
```

4. Power up the VM
5. Execute the following commands on the VM's Terminal:
    1. List disk disks and partitions and choose the adequate devices
    ```
    lsblk
    ```
    2. Add a partition to the chosen disk:
    ```
    sudo fdisk /dev/sda
    ```
	3. In the fdisk command prompt:

        i. Enter n to create a new partition

        ii. Enter to choose default partition number (4 in my case)

	    iii. Press ENTER to accept default for first sector

	    iv. Press ENTER to accept default for Last sector. This will create a partition with all the remaining available space on the disk. Alternatively you could specify the size, for example +2G will add a partition of size 2GB
        
	    v. Enter t to set the type of the new partition to Linux LVM

	    vi. Select partition number of step ii

	    vii. Enter 8e as the Hex code

	    viii. Finally enter the command w to save the changes to disk and exit fdisk

6. Reboot the Virtual Machine and Initialize LVM:
```
sudo pvcreate /dev/sda4
```

7. Add partition to volume group:
	1. Identify the root volume group with:
    ```
    sudo pvdisplay
    ```
	2. Then extend the VG:
	```
    sudo vgextend ubuntu-vg /dev/sda4
    ```

8. Extend the root logical volume:
	1. Identify the path for the root logical volume:
    ```
    sudo lvdisplay
    ```
    2. Extend the logical volume and resize to filesystem: 
    ```
    sudo lvextend --size +4G --resizefs /dev/ubuntu-vg/ubuntu-lv
    ```

9. Verify disk is resized:
```
df -h
```

source: [Increasing the size of a root partition on a Linux VM](https://www.opentechguides.com/how-to/article/linux/172/vm-extend-root.html)