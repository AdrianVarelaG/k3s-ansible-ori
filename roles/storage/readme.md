# Storage Management

## Disk Information Commands

### lsblk (List Block Devices)

The `lsblk` command lists information about all available block devices in a tree-like format.

#### Basic Usage
```bash
lsblk
```

#### Common Options
- `-f` : Show filesystem information
- `-m` : Show permissions information
- `-o` : Select output columns
- `-S` : Show SCSI devices only
- `-d` : Don't show holder devices or slave devices

#### Example Output
```
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0   500G  0 disk 
├─sda1   8:1    0   100G  0 part /
├─sda2   8:2    0    16G  0 part [SWAP]
└─sda3   8:3    0   384G  0 part /home
```

#### Output Fields Explanation
- `NAME`: Device name
- `MAJ:MIN`: Major and minor device numbers
- `RM`: Removable device (0 for no, 1 for yes)
- `SIZE`: Size of the device
- `RO`: Read-only device (0 for no, 1 for yes)
- `TYPE`: Device type (disk, part, rom, loop, etc.)
- `MOUNTPOINT`: Device mount point


### fdisk -l (List Partition Tables)

The `fdisk -l` command displays partition tables for all disks and provides detailed information about disk partitions. Root privileges are required.

#### Basic Usage
```bash
sudo fdisk -l
```

#### Example Output
```
Disk /dev/sda: 500 GiB, 536870912000 bytes, 1048576000 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt

Device         Start        End    Sectors   Size Type
/dev/sda1        2048    204800     202753   99G Linux filesystem
/dev/sda2     204801   237568      32768   16G Linux swap
/dev/sda3     237569  1048575     811007  384G Linux filesystem
```

#### Output Fields Explanation
- `Device`: Path to the partition device
- `Start`: Starting sector of the partition
- `End`: Ending sector of the partition
- `Sectors`: Total number of sectors
- `Size`: Size of the partition in human-readable format
- `Type`: Partition type (filesystem type)

#### Key Information Displayed
- Total disk size and geometry
- Sector size and count
- Partition table type (MBR/GPT)
- List of all partitions with their details
- Partition types and sizes
- Boot flags and partition flags

### fdisk (Partition Table Manipulator)

The `fdisk` command is an interactive tool for creating and manipulating partition tables. Use with extreme caution as incorrect usage can lead to data loss.

#### Basic Usage
```bash
sudo fdisk /dev/sda
```

#### Common Interactive Commands
- `m` : Display help menu
- `p` : Print the partition table
- `n` : Add a new partition
- `d` : Delete a partition
- `t` : Change partition type
- `w` : Write changes to disk
- `q` : Quit without saving changes

#### Example Interactive Session
```
Command (m for help): p
Disk /dev/sda: 500 GiB, 536870912000 bytes, 1048576000 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt

Device         Start        End    Sectors   Size Type
/dev/sda1        2048    204800     202753   99G Linux filesystem
/dev/sda2     204801    237568      32768   16G Linux swap
/dev/sda3     237569   1048575     811007  384G Linux filesystem
```

#### Important Notes
- Always backup data before modifying partition tables
- Changes are only written when using the `w` command
- Use `q` to exit without saving changes
- Root privileges (sudo) are required
- Incorrect usage can result in data loss
- Only modify partitions that are not currently mounted

#### example
```bash
# Create a new GPT partition table
Command (m for help): g
Created a new GPT disklabel (GUID: 0D4ED62C-2753-44DB-A0BC-F0063D9A4669).

# Print current partition table state
Command (m for help): p
Disk /dev/sda: 465.76 GiB, 500107862016 bytes, 976773168 sectors
Disk model: WDC WD5000AAKX-6
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 0D4ED62C-2753-44DB-A0BC-F0063D9A4669

# Create a new partition using defaults
Command (m for help): n
Partition number (1-128, default 1): 
First sector (2048-976773134, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-976773134, default 976773119): 

Created a new partition 1 of type 'Linux filesystem' and of size 465.8 GiB.

# Confirm removal of old filesystem signature
Do you want to remove the signature? [Y]es/[N]o: Y

# Write changes to disk
Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

#### Step-by-Step Explanation

1. `g` - Creates a new empty GPT (GUID Partition Table) partition table
2. `p` - Shows the current state of the disk and partition table
3. `n` - Starts the process of creating a new partition
   - Accepts default partition number (1)
   - Accepts default first sector (2048)
   - Accepts default last sector (uses all available space)
4. `Y` - Confirms removal of old filesystem signature
5. `w` - Writes all changes to disk and exits

This sequence creates a single partition that spans the entire disk using GPT partitioning scheme, which is recommended for modern systems and disks larger than 2TB.

### Creating an ext4 Filesystem

After creating a partition, you need to create a filesystem on it. The `mkfs.ext4` command creates a new ext4 filesystem on the specified partition.

#### Basic Usage
```bash
sudo mkfs.ext4 /dev/sda1
```

#### Example Output with Explanation
```bash
sudo mkfs.ext4 /dev/sda1 
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 122096384 4k blocks and 30531584 inodes
Filesystem UUID: e3d0d91e-9219-478b-bebb-5f5616dac0a6
Superblock backups stored on blocks: 
    32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
    4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968, 
    102400000

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (262144 blocks): done
Writing superblocks and filesystem accounting information: done  
```

#### Output Fields Explanation
- `4k blocks`: Size of each block in the filesystem (4 kilobytes)
- `inodes`: Number of inodes available for file indexing
- `Filesystem UUID`: Unique identifier for the filesystem
- `Superblock backups`: List of block locations where backup superblocks are stored
- `journal`: Transaction log that helps maintain filesystem integrity

#### Key Steps in Filesystem Creation
1. Allocation of group tables
2. Writing inode tables for file indexing
3. Creating journaling system for crash recovery
4. Writing superblocks and filesystem metadata

#### Important Notes
- Requires root privileges (sudo)
- Will destroy any existing data on the partition
- Process cannot be interrupted safely
- Filesystem must be mounted to be used

####
```
 sudo mkdir /mnt/hdd1
 sudo mount /dev/sda1 /mnt/hdd1/
 df -h
Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              764M  3.5M  761M   1% /run
efivarfs                           192K   65K  123K  35% /sys/firmware/efi/efivars
/dev/mapper/ubuntu--vg-ubuntu--lv   98G   27G   67G  29% /
tmpfs                              3.8G     0  3.8G   0% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
/dev/nvme0n1p2                     2.0G  101M  1.7G   6% /boot
/dev/nvme0n1p1                     1.1G  6.2M  1.1G   1% /boot/efi
shm                                 64M     0   64M   0% /run/k3s/containerd/io.containerd.grpc.v1.cri/sandboxes/47426f65eaafe32ca2e676ccd1569fcf443e008d99ebe23100643a0501f0198d/shm
shm                                 64M     0   64M   0% /run/k3s/containerd/io.containerd.grpc.v1.cri/sandboxes/d221cdba7024eac481cf3ecae4fbf9963cbbe031a3e57b570abff20133b1a599/shm
shm                                 64M     0   64M   0% /run/k3s/containerd/io.containerd.grpc.v1.cri/sandboxes/3bb929d50e3fa3f1f9012ccbe95e51b8504ccb1fb8d4c1f647c92e64a6338265/shm
shm                                 64M     0   64M   0% /run/k3s/containerd/io.containerd.grpc.v1.cri/sandboxes/7b19f16287ab968b4d5ab3bf9bc1739079938d6970795da64d7588dfcb009f8a/shm
shm                                 64M     0   64M   0% /run/k3s/containerd/io.containerd.grpc.v1.cri/sandboxes/0bae97ff191cf438b52d24caa9689e7de05d8e8d37d900b154f2db3bba81e592/shm
shm                                 64M     0   64M   0% /run/k3s/containerd/io.containerd.grpc.v1.cri/sandboxes/cb51646e1e5b35efe7b6a88b5a5b8de81bc63d43a5f26b208d6d5aa54a43120d/shm
shm                                 64M     0   64M   0% /run/k3s/containerd/io.containerd.grpc.v1.cri/sandboxes/d5c80d8bf9a7a37446e12f19da6d7aa34cdeca5628b990625774de67413efb04/shm
shm                                 64M     0   64M   0% /run/k3s/containerd/io.containerd.grpc.v1.cri/sandboxes/2fdf0961f565d86e594e734eabcc93bb3de0287777766690c2622459b35c2868/shm
shm                                 64M  6.0M   59M  10% /run/k3s/containerd/io.containerd.grpc.v1.cri/sandboxes/469393fed7875249b5bcb77224f1a23c54a035aee7e409aa7f27f6351ce612e2/shm
tmpfs                              764M   16K  764M   1% /run/user/1000
/dev/sda1                          458G   28K  435G   1% /mnt/hdd1
```