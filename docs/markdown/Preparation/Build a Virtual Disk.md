# Build a Virtual Disk

## Disk Partition

Disk partitioning is the creation of one or more logical regions on one stor-age  device  so  that  each  region  can  be  managed  separately.   Historically,  disk partitioning was used to overcome the limitations of filesystem, especially FAT16, which is used in Windows 95.  The FAT 16 filesystem can support maximum volume size from 1 MB to 32 MB so for any storage devices larger than 32 MB,it was necessary to create partitions on the device

However,  with  modern  filesystem  like  NTFS,  which  can  support  up  to  $16$ EB or $1.153∗10^{12}$ MB, partitioning no longer serves the above purpose.  Then why is disk partitioning still relevant today? There are two reasons:

* Faster File System Processing
* Install Multiple Operating System on one storage device

## Partition Table

Partition Table is the structure on the storage devices that describes each partition size and some other metadata.

### MBR

In [Boot Sequence](Boot Sequence.md#master-boot-record), we talked about MBR. In this section, we are going to talk about the partition entries inside MBR. 

??? question "What are CHS, LBA"
    **CHS** (Cylinder-Head-Sector) and **LBA** (Logical Block Addressing) are two addressing methods used to address hard disks.

    ## CHS

    CHS is the addressing mode using cylinder, head and sector.

    * Track: Each concentric ring on the platter is called a track
    * Cylinder: A hard disk consists of one or more platters. Each platter has some number of tracks. The vertical stack of tracks aligning is called a cylinder.
    * Head: Head is used to read information and write information to the disk. Each platter has its own heads
    * Sector: Each track is divided into multiple sectors
    ## LBA

    In LBA, instead of using cylinder, head and sector to address a sector, each sector is assigned an unique number and can be addressed with that number.

    ??? "Limitations of CHS"
        ## BIOS Interrupt Interface
        When the IBM engineer designed the AT or IBM personal computer, they put the interface to access the hard disk in the BIOS. Specifically, the interface is "Int 13h" or interrupt 13. You can think of it as a "function call"   to BIOS if you do not know what interrupt is. Remember, they were using 8088, which is a 16 bits processor and thus have register size of 16 bits. The interrupt call 13 requires the user to put numbers in some registers in terms of CHS to do operation on the hard disk. And the IBM engineer designed the call so that the call can accept maximum 1024 cylinders, 256 heads and 63 sectors. If 512 bytes per sector, then BIOS can only address up to   8 GB hard disk.

        ## ATA Specification
        Most computers today use SATA to connect to storage devices such as hard disk (SATA is gradually replaced by a newer standard called PCIe). It was an interface standard for storage devices designed by Western Digital for AT computer and was called ATA or AT Attachment. If using devices compliant to ATA specification, the BIOS can only address up to 504 MB. How?
        This is due to ATA specification has different limits than the BIOS:

        |                   | BIOS | ATA/IDE  | Combined Limit |
        |-------------------|------|----------|----------------|
        | Max. Sector/track | 63   | 255      |       63       |
        | Max. Heads        | 256  | 16       |       16       |
        | Max. Cylinders    | 1024 | 65536    |      1024      |
        | Max. Capacity     | 8 GB | 127.5 GB |     504 MB     |


        ## Solution
        The early BIOS Int 13h interface directly pass the CHS address to the hard disk controller, and thus creating above limitation. To overcome such limitation, we need a translating BIOS which can translate the CHS address to different address format.

        The CHS which DOS uses to call the BIOS is called L-CHS (Logical CHS) and the CHS which the BIOS uses to control the drive is the P-CHS (Physical CHS).

        There are two translation methods:

        ### Directly translate L-CHS to P-CHS

        ATA-2 specification specifies that al EIDE drives up to 8 GB should conform to the BIOS sector limit of 63 sectors/track. Then only the number of cylinders will be above the BIOS limit of 1024.

        ### Translate L-CHS to LBA (LBA-assisted Method)

The layout of each entry:

| Offset | Size    | Function                                                                                  |
|--------|---------|-------------------------------------------------------------------------------------------|
| 0x00   | 1 byte  | Status of drive (bit 7 indicates if the drive is bootable; 0: no, 0x80: bootable(active)) |
| 0x01   | 1 byte  | Starting head                                                                             |
| 0x02   | 6 bits  | Starting sector (bits 6,7 are the upper two bits of the next field)                       |
| 0x03   | 10 bits | Starting cylinder                                                                         |
| 0x04   | 1 byte  | System ID, indicating the type of filesystem on the partition                             |
| 0x05   | 1 byte  | Ending head                                                                               |
| 0x06   | 6 bits  | Ending Sector                                                                             |
| 0x07   | 10 bits | Ending cylinder                                                                           |
| 0x08   | 4 bytes | Relative Sector (LBA of first sector, equivalent to the LBA of the partition)             |
| 0x0C   | 4 bytes | Number of sectors in partition                                                            |
<!--TODO: draw a diagram for above table -->

### GPT

## Build Virtual Disk

### Virtual Disk

Virtual disk is the software component (binary file) that emulate an actual disk storage device. We are going to create a disk image which contains the exact data structure of an actual storage device. A virtual hard disk typically consists of sector-by-sector structure. There are many file format existing for disk image, one example being ISO image. 

### Tools

#### fdisk

"fdisk is a dialog-driven program for creation
 and manipulation of partition tables." (fdisk man page). We will be using fdisk to create partition table for our virtual disk.

#### mke2fs

"mke2fs is used to create an ext2, ext3, or ext4 filesystem, usually in a disk partition." (mke2fs man page). We will be using this tool to create filesystem on our virtual disk.

#### dd

dd is used to copy a file, converting and formatting according to the operands. We will be using dd to write our boot code into MBR of our disk.

### Concept

#### Loop Device

In Unix-like operating system, a loop device is a pseudo-device that makes a file accessible as a block device. A block device is a special file that provides buffered access to hardware devices, which means whatever the programmer writes to this device file will be buffered before send to actual hardware device. As mke2fs only operate on block devices, it is necessary to setup out disk with loop device.

### Creating Virtual Disk

FIrst, we are going to create an empty disk. Type the following command in your shell:

```shell
$ dd if=/dev/zero of=disk.img bs=1k count=32760
```
This line of command creates a virtual disk of 32760*1024 bytes. 

* "if=" specify the file that dd should read from -- in our case, "/dev/zero", which is a special file in Unix-like operating system that if read from it, the operating system will return sequence of null characters. 
* "of=" specify the file that dd should write to, which is "disk.img" in our case. 
* "bs=" specify number of bytes per block should be read and write at a time
* "count=" specify the total number of input blocks, in our case -- number of blocks of null characters

Next, let's create MBR on our disk image.

```shell
$ fdisk disk.img
$ Command (m for help): x
$ Expert command (m for help): h
Number of heads (1-256, default 255): 16

$ Expert command (m for help): s
Number of sectors (1-63, default 63): 63
Expert command (m for help): c
Number of cylinders (1-1048576): 60

$ Expert command (m for help): r

$ Command (m for help): n
Command action
   e   extended
   p   primary partition (1-4)
p
<!-- TODO: Rewrite this part (New version of fdisk no longer requires CHS but sector)-->
Partition number (1-4): 1
First cylinder (1-60, default 1): 1
Using default value 1
Last cylinder or +size or +sizeM or +sizeK (1-60, default 60): 60
Using default value 60

$ Command (m for help): t
Selected partition 1
Hex code (type L to list codes): 83

$ Command (m for help): a
Partition number (1-4): 1

$ Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.

WARNING: Re-reading the partition table failed with error 25: Inappropriate
ioctl for device.
```
<!-- TODO: Write Explanation-->
Finally, we will be creating a filesystem on the disk.
``` shell
$ modprobe loop
$ losetup -o 32256 /dev/loop0 disk.img
$ mke2fs /dev/loop0
$ losetup -d /dev/loop0
```

You can mount the disk if you wish to.
``` shell
$ modprobe ext2
$ mkdir /mnt/vdisk
$ mount disk.img /mnt/vdisk -text2 -o loop,offset=32256
```
