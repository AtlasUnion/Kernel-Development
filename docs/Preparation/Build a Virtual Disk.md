# Build a Virtual Disk

## Disk Partition

Disk partitioning is the creation of one or more logical regions on one stor-age  device  so  that  each  region  can  be  managed  separately.   Historically,  diskpartitioning was used to overcome the limitations of filesystem, especially FAT16, which is used in Windows 95.  The FAT 16 filesystem can support maximunvolume size from 1 MB to 32 MB so for any storage devices larger than 32 MB,it was necessary to create partitions on the device

However,  with  modern  filesystem  like  NTFS,  which  can  support  up  to  $16$ EB or $1.153∗10^{12}$ MB, partitioning no longer serves the above purpose.  Thenwhy is disk partitioning still relevant today? There are two reasons:

* Faster File System Processing
* Install Multiple Operating System on one storage device

<!--TODO: -->

## Partition Table

Partition Table is the structure on the storage devices that describes each partition size and some other metadata.

### MBR



### GPT