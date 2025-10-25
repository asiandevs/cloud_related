# Adding Swap Space for Oracle Installation

## Problem: Insufficient Swap Space
When running the Oracle installer, the swap space check fails:

```bash
oracle@hostname client]$ ./runInstaller -silent -responseFile /u03/app/oracle/client19c/client/response/client_install.rsp
Starting Oracle Universal Installer...

Checking Temp space: must be greater than 415 MB.   Actual 2416 MB    Passed
Checking swap space: 0 MB available, 150 MB required.    Failed <<<<

Some requirement checks failed. You must fulfill these requirements before
continuing with the installation,

Exiting Oracle Universal Installer, log for this session can be found at /tmp/OraInstall2025-01-08_05-30-18AM/installActions2025-01-08_05-30-18AM.log
```

## Solution: Configure Swap File

### Step 1: Check Current Swap Space
```bash
[root@hostname ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:          15802         139       12501           0        3161       15398
Swap:             0           0           0
```

### Step 2: Create Swap File
Create a 500 MB swap file (recommended for Oracle Database installation):
```bash
[root@hostname ~]# dd if=/dev/zero of=/swapfile count=500 bs=1MiB
500+0 records in
500+0 records out
524288000 bytes (524 MB) copied, 0.202619 s, 2.6 GB/s
```

### Step 3: Set Proper Permissions
```bash
[root@hostname ~]# chmod 600 /swapfile
```

### Step 4: Format as Swap Space
```bash
[root@hostname ~]# mkswap /swapfile
Setting up swapspace version 1, size = 500 MiB (524283904 bytes)
no label, UUID=b61dc323-e7d3-4480-bb3b-c48cf07a3002
```

### Step 5: Activate Swap File
```bash
[root@hostname ~]# swapon /swapfile
```

### Step 6: Verify Revised Swap Space
```bash
[root@hostname ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:          15802         139       12000           0        3662       15397
Swap:           499           0         499
```

### Step 7: Make Swap Permanent
Add this entry to `/etc/fstab`:
```
/swapfile swap swap defaults 0 0
```

##  Success
The swap space has been successfully increased and made permanent!
