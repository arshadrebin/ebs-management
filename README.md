### EBS Management - Increasing/ decreasing

Elastic Block Store (EBS) is a block storage service provided by Amazon Web Services (AWS). It is designed to provide persistent block-level storage volumes for use with EC2 instances. EBS is a fundamental component of the AWS infrastructure and is widely used for various purposes, such as hosting databases, storing application data, and supporting other storage-intensive workloads in the cloud.

Let's see some key characteristics and features of EBS:

* **Block-Level Storage**: EBS offers block storage, which means it provides raw storage volumes that function similarly to physical hard drives or SSDs. These volumes can be formatted with a file system and used as persistent storage for EC2 instances.

* **Persistence and Durability**: EBS volumes are designed for durability and data persistence. They provide long-term storage for your data even if the associated EC2 instance is stopped or terminated.

* **Elasticity and Scalability**: With EBS, you can easily scale the size of your storage volumes as your needs change. You can increase or decrease the volume size without impacting the running EC2 instances.

* **High Availability and Durability**: EBS volumes are designed for high availability and durability. They are replicated within a specific Availability Zone (AZ) to protect against failures, ensuring that your data remains accessible and protected.

* **Scalability**: You can easily adjust the size and performance of EBS volumes to meet the changing needs of your applications. You can also create multiple volumes and stripe them together to achieve higher performance and capacity.

### Creating a Volume and attaching to an Instance.

In order to create an EBS volume and attach it to an EC2 instance, you can follow these steps:

1. Open the AWS Management Console and navigate to the EC2 service.
2. Click on "Volumes" in the left-hand menu under "Elastic Block Store."
3. Click on the "Create Volume" button.
4. In the "Create Volume" dialog, specify the desired settings for the volume:

    * Availability Zone: Choose the same Availability Zone as the EC2 instance you want to attach the volume to.

    * Volume Type: Select the appropriate volume type based on your performance and cost requirements.
    
    * Size: Specify the size of the volume in gigabytes (GB).
    
<img width="451" alt="Screenshot 2023-05-31 125348" src="https://github.com/arshadrebin/ebs-management/assets/116037443/38aefa0e-d7a9-43f1-abc3-cf41d8925971">

Now let us see how to attach this volume to an Instance which we already have.

* Once the volume is created, go back to the "Volumes" page and select the newly created volume.
* Click on the "Actions" dropdown menu and choose "Attach Volume."
* In the "Attach Volume" dialog, select the EC2 instance you want to attach the volume to from the "Instance" dropdown menu.
* Click on the "Attach" button to attach the volume to the instance.

<img width="587" alt="Screenshot 2023-05-31 125703" src="https://github.com/arshadrebin/ebs-management/assets/116037443/6e18023e-41ee-45dd-8519-0e38ab9cf6e4">

Once attached, the volume will appear as a new block device on the EC2 instance.

Inorder to check this, log in to the EC2 instance, and use commands lsblk.


```
[ec2-user@ip-172-31-46-3 ~]$ lsblk
NAME      MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
xvda      202:0    0   8G  0 disk
├─xvda1   202:1    0   8G  0 part /
├─xvda127 259:0    0   1M  0 part
└─xvda128 259:1    0  10M  0 part /boot/efi
xvdf      202:80   0   1G  0 disk
[ec2-user@ip-172-31-46-3 ~]$
```

From the above snippet you can see an additional disk named xvdf with 1G. Now we need to format and mount the additional volume.

Here showing a scenario that a client want an additional volume mount to their website document root.


![241898560-523e044c-8a3d-46c3-bb35-a3d1196a0659](https://github.com/arshadrebin/ebs-management/assets/116037443/7e458e19-0958-4d7a-96d9-95200df846d0)

```
[ec2-user@ip-172-31-46-3 ~]$ sudo fdisk /dev/xvdf

Welcome to fdisk (util-linux 2.37.4).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0xd4636e6b.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1):
First sector (2048-2097151, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-2097151, default 2097151):

Created a new partition 1 of type 'Linux' and of size 1023 MiB.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

[ec2-user@ip-172-31-46-3 ~]$ lsblk
NAME      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
xvda      202:0    0    8G  0 disk
├─xvda1   202:1    0    8G  0 part /
├─xvda127 259:0    0    1M  0 part
└─xvda128 259:1    0   10M  0 part /boot/efi
xvdf      202:80   0    1G  0 disk
└─xvdf1   202:81   0 1023M  0 part
[ec2-user@ip-172-31-46-3 ~]$ sudo mkfs -t ext4 /dev/xvdf1
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 261888 4k blocks and 65536 inodes
Filesystem UUID: 5b392e98-f5cf-4605-95b1-e394eb98b4cd
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

[ec2-user@ip-172-31-46-3 ~]$
```

Now mounting the additional volume to /var/www/html.

```
[ec2-user@ip-172-31-46-3 ~]$  sudo mount /dev/xvdf1 /mnt/
[ec2-user@ip-172-31-46-3 ~]$
[ec2-user@ip-172-31-46-3 ~]$
[ec2-user@ip-172-31-46-3 ~]$ lsblk
NAME      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
xvda      202:0    0    8G  0 disk
├─xvda1   202:1    0    8G  0 part /
├─xvda127 259:0    0    1M  0 part
└─xvda128 259:1    0   10M  0 part /boot/efi
xvdf      202:80   0    1G  0 disk
└─xvdf1   202:81   0 1023M  0 part /mnt
[ec2-user@ip-172-31-46-3 ~]$ sudo cp -ra /var/www/html/* /mnt/
[ec2-user@ip-172-31-46-3 ~]$ sudo umount /mnt
[ec2-user@ip-172-31-46-3 ~]$ sudo systemctl stop httpd.service
[ec2-user@ip-172-31-46-3 ~]$
```

Then add the entry in fstab for persistence.

```
[ec2-user@ip-172-31-46-3 ~]$ sudo vim /etc/fstab
```

<img width="736" alt="Screenshot 2023-05-31 130907" src="https://github.com/arshadrebin/ebs-management/assets/116037443/cfdb132c-cb3b-46da-ae82-adf962766133">

```
[ec2-user@ip-172-31-46-3 ~]$ sudo mount -a
[ec2-user@ip-172-31-46-3 ~]$ sudo systemctl start httpd.service
[ec2-user@ip-172-31-46-3 ~]$ lsblk
NAME      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
xvda      202:0    0    8G  0 disk
├─xvda1   202:1    0    8G  0 part /
├─xvda127 259:0    0    1M  0 part
└─xvda128 259:1    0   10M  0 part /boot/efi
xvdf      202:80   0    1G  0 disk
└─xvdf1   202:81   0 1023M  0 part /var/www/html
[ec2-user@ip-172-31-46-3 ~]$
```

Now we can see that the newly added 1G volume is mounted to /var/www/html.

How to increase the additional volume size for an Ec2 instance.

Here we are going to see how we can increase the attached 1G additional volume in the above session to 2G.

To increase the size of an existing EBS volume, you can follow these steps:

* Open the AWS Management Console and navigate to the EC2 service.
* Click on "Volumes" in the left-hand menu under "Elastic Block Store."
* Select the EBS volume that you want to increase in size.
* Click on the "Actions" dropdown menu and choose "Modify Volume."

In the "Modify Volume" dialog, update the size of the volume to the desired new size. Note that you can only increase the size, not decrease it.

<img width="932" alt="Screenshot 2023-05-31 131556" src="https://github.com/arshadrebin/ebs-management/assets/116037443/caceb830-d40e-479c-80fc-cb5b79c25bc5">

* Click on the "Modify" button to apply the size increase.
* Once the modification is complete, the EBS volume will be expanded to the new size.

After increasing the volume size, you may need to perform additional steps to resize the file system and make use of the additional space. Let us see how do that. Log in to the instance.

```
[ec2-user@ip-172-31-46-3 ~]$ lsblk
NAME      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
xvda      202:0    0    8G  0 disk
├─xvda1   202:1    0    8G  0 part /
├─xvda127 259:0    0    1M  0 part
└─xvda128 259:1    0   10M  0 part /boot/efi
xvdf      202:80   0    1G  0 disk
└─xvdf1   202:81   0 1023M  0 part /mnt

[ec2-user@ip-172-31-46-3 ~]$ lsblk
NAME      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
xvda      202:0    0    8G  0 disk
├─xvda1   202:1    0    8G  0 part /
├─xvda127 259:0    0    1M  0 part
└─xvda128 259:1    0   10M  0 part /boot/efi
xvdf      202:80   0    2G  0 disk
└─xvdf1   202:81   0 1023M  0 part /var/www/html
```

In the above snippet you can see the output of the disk before modifying the additional volume and after modifying. Here the addition of the additional 1G is reflected to the disk but not to the mount point. In oder to do this please follow the steps below.

```
[ec2-user@ip-172-31-46-3 ~]$ sudo growpart /dev/xvdf 1
CHANGED: partition=1 start=2048 old: size=2095104 end=2097152 new: size=4192223 end=4194271
[ec2-user@ip-172-31-46-3 ~]$ sudo lsblk -f
NAME      FSTYPE FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
xvda
├─xvda1   xfs          /     55a1aebd-f196-4f84-8afe-075f5d1dda63    6.3G    20% /
├─xvda127
└─xvda128 vfat   FAT16       0383-1543                               8.7M    13% /boot/efi
xvdf
└─xvdf1   ext4   1.0         5b392e98-f5cf-4605-95b1-e394eb98b4cd  916.2M     1% /var/www/html
[ec2-user@ip-172-31-46-3 ~]$ sudo resize2fs /dev/xvdf1
resize2fs 1.46.5 (30-Dec-2021)
Filesystem at /dev/xvdf1 is mounted on /var/www/html; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 1
The filesystem on /dev/xvdf1 is now 524027 (4k) blocks long.

[ec2-user@ip-172-31-46-3 ~]$ lsblk
NAME      MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
xvda      202:0    0   8G  0 disk
├─xvda1   202:1    0   8G  0 part /
├─xvda127 259:0    0   1M  0 part
└─xvda128 259:1    0  10M  0 part /boot/efi
xvdf      202:80   0   2G  0 disk
└─xvdf1   202:81   0   2G  0 part /var/www/html
```

Please be noted that once the volume is expanded, we can only re expand the volume after 6 hours.

### How to increase the root EBS volume

* Open the AWS Management Console and navigate to the EC2 service.
* Click on the Instance, then under the storage section note down the Instance root volume ID.
* Make size to 10G and Click on modify.

<img width="506" alt="Screenshot 2023-05-31 132420" src="https://github.com/arshadrebin/ebs-management/assets/116037443/10fc79c0-655b-4ca9-ac8d-5e5ee0eb5fbb">

Once the modification is complete, the root EBS volume will be expanded to the new size.
After increasing the volume size, you may need to perform additional steps to resize the file system and make use of the additional space. Let us see how do that. Log in to the instance.

```
[ec2-user@ip-172-31-46-3 ~]$ lsblk
NAME      MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
xvda      202:0    0  10G  0 disk
├─xvda1   202:1    0   8G  0 part /
├─xvda127 259:0    0   1M  0 part
└─xvda128 259:1    0  10M  0 part /boot/efi
xvdf      202:80   0   2G  0 disk
└─xvdf1   202:81   0   2G  0 part /var/www/html
```

As we can see that the root volume is expanded to 10G. But it is not reflected to the partition. So please follow the below steps to reflect it.

```
[ec2-user@ip-172-31-46-3 ~]$ sudo growpart /dev/xvda 1
CHANGED: partition=1 start=24576 old: size=16752607 end=16777183 new: size=20946911 end=20971487
[ec2-user@ip-172-31-46-3 ~]$ sudo lsblk -f
NAME      FSTYPE FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
xvda
├─xvda1   xfs          /     55a1aebd-f196-4f84-8afe-075f5d1dda63    6.3G    20% /
├─xvda127
└─xvda128 vfat   FAT16       0383-1543                               8.7M    13% /boot/efi
xvdf
└─xvdf1   ext4   1.0         5b392e98-f5cf-4605-95b1-e394eb98b4cd    1.8G     0% /var/www/html
```

Here we can see that the file system for the root volume is xfs. So we can't use resize2fs to expand this volume like before. Here we are using xfs_growfs to resize the root volume.

```
[ec2-user@ip-172-31-46-3 ~]$ sudo xfs_growfs -d /
meta-data=/dev/xvda1             isize=512    agcount=2, agsize=1047040 blks
         =                       sectsz=4096  attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1
data     =                       bsize=4096   blocks=2094075, imaxpct=25
         =                       sunit=128    swidth=128 blks
naming   =version 2              bsize=16384  ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=4096  sunit=4 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 2094075 to 2618363
[ec2-user@ip-172-31-46-3 ~]$ lsblk
NAME      MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
xvda      202:0    0  10G  0 disk
├─xvda1   202:1    0  10G  0 part /
├─xvda127 259:0    0   1M  0 part
└─xvda128 259:1    0  10M  0 part /boot/efi
xvdf      202:80   0   2G  0 disk
└─xvdf1   202:81   0   2G  0 part /var/www/html
````
