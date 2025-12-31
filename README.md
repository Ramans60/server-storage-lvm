###Server Storage Fundamentals: Combining Multiple Physical Disks into One Logical Volume Using LVM (Linux)

=> This is a self-study project exploring how servers combine multiple physical disks into a single logical storage unit using Linux Logical Volume Manager (LVM).  
The project includes hands-on implementation, real-world troubleshooting, and practical storage concepts used in production systems.

## Introduction

* Have you ever wondered how data center servers manage storage?  
* Most data center servers run on Linux and typically contain dozens of physical disks. Instead of using each disk separately, these disks are logically combined and managed as a single storage unit.
* This documentation explains how physical disks in servers are logically combined into one logical volume using Linux Logical Volume Manager (LVM), which is the same concept used in real-world data center environments.
  
##Why servers need logical storage ?  

  * Physical disks are limited – A single disk has a fixed size (modern HDDs can reach 30TB+, and SSDs even higher), with models up to 100TB); logical storage lets multiple disks act as one large storage.
  * Easy expansion – storage can be increased without stopping the server.
  * Better utilization – unused space from different disks can be pooled and used efficiently.
  * High availability – logical storage works well with redundancy (failure of one disk doesn’t stop the service).
  * Simpler management – admins manage one logical volume instead of many disks.

    
## What is LVM?

*Logical Volume Manager (LVM) is a Linux storage management technology that allows multiple physical disks to be combined and managed as flexible logical storage units.

<img width="1024" height="1024" alt="Gemini_Generated_Image_bob1n9bob1n9bob1" src="https://github.com/user-attachments/assets/eabbc3ee-bfe3-4f05-b69d-5afa347fdfcf" />



##What is a Physical Volume?
A Physical Volume (PV) is a disk that is initialized for LVM usage.After this step, the disk can be used by LVM, but it is still not usable storage.

##What is a Volume Group?
A Volume Group (VG) is a storage pool created by combining multiple Physical Volumes.Once a VG is created, individual disk boundaries disappear.


##What is a Logical Volume?
A Logical Volume (LV) is the virtual disk created from a Volume Group.This is what the operating system finally treats like a real disk.

##What is a filesystem?
A filesystem defines how data is stored, named, and retrieved on a disk. Without a filesystem, a disk (even an LV) cannot store files.

##What is mounting?
Mounting attaches a filesystem to the Linux directory tree.Only after mounting can users read/write files.

#step 1 : open terminal (ctrl+alt+terminal)

<img width="1920" height="1080" alt="Screenshot from 2025-12-31 13-49-51" src="https://github.com/user-attachments/assets/9e48eb96-34b1-4a08-871d-53c347a28a4f" />

#step 2 : Identifying available disks
  ```bash
    lsblk
  ```

<img width="1920" height="1080" alt="Screenshot from 2025-12-31 10-17-18" src="https://github.com/user-attachments/assets/17f52365-c5c8-401f-8fc7-c6ab6b1aa888" />

#step 3 : Unmount the disks first 
  ```bash
      sudo umount /dev/sdd* 2>/dev/null
      sudo umount /dev/sde* 2>/dev/null
      sudo umount /dev/sdf* 2>/dev/null
   ```
<img width="1918" height="904" alt="Screenshot from 2025-12-31 10-18-38" src="https://github.com/user-attachments/assets/3a11fb3b-7a15-4f28-b9e5-8eeafd344f18" />


<img width="1920" height="1080" alt="Screenshot from 2025-12-31 10-21-58" src="https://github.com/user-attachments/assets/7eb5789c-aa15-48dd-9239-9df57a68cef8" />

  >(No output = success)
  Verification :
   Run:
    ```bash
        lsblk
    ```
  => Before initializing disks for LVM, all existing filesystem, partition, and LVM metadata were wiped using wipefs . This ensures a clean storage state and prevents conflicts from previously used configurations.



#step 4 : Cleaning old disk metadata
  ```bash
  	sudo wipefs -a /dev/sdd
    sudo wipefs -a /dev/sde
    sudo wipefs -a /dev/sdf

```
Enter password when asked.
	
Expected output :
No error
Some “bytes erased” message (or silent success)
  
Verification :
```bash
sudo wipefs /dev/sdd
sudo wipefs /dev/sde
sudo wipefs /dev/sdf

```
Expected output:
Nothing printed
That means disks are 100% clean.

<img width="1920" height="1080" alt="Screenshot from 2025-12-31 10-24-55" src="https://github.com/user-attachments/assets/1947175b-f1f1-40f4-8115-7e219ff6ae88" />

#step 5 : Creating Physical Volumes (PV)
  
  ```bash
  sudo pvcreate /dev/sdd /dev/sde /dev/sdf
  
  ```
  
 Verification :
  ```bash
  sudo pvs
  ```
  <img width="1920" height="1080" alt="Screenshot from 2025-12-31 10-25-32" src="https://github.com/user-attachments/assets/6611defa-c60a-488e-97c1-26d05c20f192" />

<img width="1920" height="1080" alt="Screenshot from 2025-12-31 10-26-16" src="https://github.com/user-attachments/assets/1fb9cc75-430a-40bf-a503-ada1368a3cbc" />

#step 6 : Creating a Volume Group 

   Create a Volume Group named server_vg:
   ```bash
   sudo vgcreate server_vg /dev/sdd /dev/sde /dev/sdf
   
   ```
  Verification :
   ```bash
   sudo vgs
   
   ```


<img width="1920" height="1080" alt="Screenshot from 2025-12-31 10-26-56" src="https://github.com/user-attachments/assets/0b30dcd8-e61d-4c19-9b4f-bb3b7edd0b80" />

<img width="1920" height="1080" alt="Screenshot from 2025-12-31 10-27-07" src="https://github.com/user-attachments/assets/438283bd-5641-4480-a364-d1c5c35cb2c0" />

=> The Physical Volumes were combined into a single Volume Group named server_vg . The Volume Group acts as a unified storage pool from which logical storage units can be allocated.  

#step 7 : create Logical Volume


Create one LV using full space:
  ```bash
  sudo lvcreate -l 100%FREE -n data_lv server_vg
  
  ```
Verification  :

   ```bash
   sudo lvs
   
   ```

<img width="1920" height="1080" alt="Screenshot from 2025-12-31 10-27-50" src="https://github.com/user-attachments/assets/a707121e-ea8a-45e3-8284-f1d761333646" />

<img width="1920" height="1080" alt="Screenshot from 2025-12-31 10-28-18" src="https://github.com/user-attachments/assets/c83359a3-c1e2-4252-8687-cb101eaf2ca7" />

=> A Logical Volume named data_lv was created using all available space from the Volume Group server_vg . The Logical Volume represents a virtual block device that abstracts the underlying physical disks.   


#step 8 : Creating a file system

Filesystem choice :
  We will use EXT4 (industry standard, stable).

 ```bash
  sudo mkfs.ext4 /dev/server_vg/data_lv
 
 ```
 
Verification :
 ```bash
 sudo blkid /dev/server_vg/data_lv
 
 ```
   Expected:
      "TYPE="ext4"".
	 


<img width="1920" height="1080" alt="Screenshot from 2025-12-31 10-29-25" src="https://github.com/user-attachments/assets/db4317f8-0fd3-4303-abe2-854e079788e2" />


<img width="1920" height="1080" alt="Screenshot from 2025-12-31 10-29-39" src="https://github.com/user-attachments/assets/2a2f8d79-4ebd-4391-8d5d-259407c7999b" />

=> An EXT4 filesystem was created on the Logical Volume data_lv . This step enables the logical storage to organize and manage files in a structured format.

#step 9 : Mounting the disks. 
  Create mount directory
   
   ```bash
   sudo mkdir /mnt/server_data
   
   ```
  Mount the logical volume

  ```bash
  sudo mount /dev/server_vg/data_lv /mnt/server_data
 
  ```

  Verification :
 
  ```bash
  df -h | grep server_data
 
  ```


<img width="1920" height="1080" alt="Screenshot from 2025-12-31 10-30-36" src="https://github.com/user-attachments/assets/de5c817e-482b-4e2e-b41c-20b2e262fcce" />


<img width="1920" height="1080" alt="Screenshot from 2025-12-31 14-16-05" src="https://github.com/user-attachments/assets/3f2c9ce0-3273-4fb8-ae53-faaff35c6df8" />



 => Finally we have correctly and cleanly completed a server-grade LVM setup from scratch.Everything you executed is 100% correct.

 SUMMARY :
 > Three physical storage devices were combined using Logical Volume Manager (LVM). Each device was initialized as a Physical Volume and pooled into a single Volume Group. A Logical Volume was created using the entire available space, formatted with the EXT4 filesystem, and mounted to provide unified logical storage access.

