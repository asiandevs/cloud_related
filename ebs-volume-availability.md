## Check filesystem
```
root@ ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        7.8G     0  7.8G   0% /dev
tmpfs           7.8G     0  7.8G   0% /dev/shm
tmpfs           7.8G  412K  7.8G   1% /run
tmpfs           7.8G     0  7.8G   0% /sys/fs/cgroup
/dev/nvme0n1p1  8.0G  5.3G  2.8G  66% /
tmpfs           1.6G     0  1.6G   0% /run/user/0
```
## EBS volumes as NVMe block devices
The root device is /dev/nvme0n1, which has two partitions named nvme0n1p1 and nvme0n1p128. The attached volume is /dev/nvme1n1, which has no partitions and is not yet mounted.
```
[root@HOST ~]# lsblk
NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTHOST
nvme1n1       259:0    0  400G  0 disk
nvme0n1       259:1    0    8G  0 disk
├─nvme0n1p1   259:2    0    8G  0 part /
└─nvme0n1p128 259:3    0    1M  0 part
```
## Determine whether there is a file system on the volume.
[root@HOST ~]# sudo file -s /dev/nvme1n1
/dev/nvme1n1: data
```
If the output shows simply data, as in the following example output, there is no file system on the device

## Use the lsblk -f command to get information about all of the devices attached to the instance.
```
[root@HOST ~]# sudo lsblk -f
NAME          FSTYPE LABEL UUID                                 MOUNTPOINT
nvme1n1
nvme0n1
├─nvme0n1p1   xfs    /     33e2260f-ca11-4835-8455-89cfa1ba7201 /
└─nvme0n1p128
```

## create a file system on the volume
```
[root@HOST ~]# sudo mkfs -t xfs /dev/nvme1n1
meta-data=/dev/nvme1n1           isize=512    agcount=16, agsize=6553600 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=0 inobtcount=0
data     =                       bsize=4096   blocks=104857600, imaxpct=25
         =                       sunit=1      swidth=1 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=51200, version=2
         =                       sectsz=512   sunit=1 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```
## create a mount point directory for the volume
```

[root@HOST ~]# sudo mkdir /u03
```
## Mount the volume or partition at the mount point
```
[root@HOST ~]# sudo mount /dev/nvme1n1 /u03
```
## Validate
```
[root@HOST ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        7.8G     0  7.8G   0% /dev
tmpfs           7.8G     0  7.8G   0% /dev/shm
tmpfs           7.8G  412K  7.8G   1% /run
tmpfs           7.8G     0  7.8G   0% /sys/fs/cgroup
/dev/nvme0n1p1  8.0G  5.3G  2.8G  66% /
tmpfs           1.6G     0  1.6G   0% /run/user/0
/dev/nvme1n1    400G  2.9G  397G   1% /u03
```

Reference: https://docs.aws.amazon.com/ebs/latest/userguide/ebs-using-volumes.html#ebs-format-mount-volume
