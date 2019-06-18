# Boot Sequence

Have you ever wondered what happen after you press the power button? In this section, we are going to demystify the process.

## Booting

Booting is starting up a computer until it is ready for users to use it. It is usually initiated by the press of power button on a personal computer although it is possible to initiate booting via software command. After you press the power button, the CPU or Central Processing Unit is powered on with no execution logic in the RAM, which is volatile memory and thus lose information once power off and on. So, to let the CPU do work, some logic must be loaded into the memory to let CPU execute.

But how does CPU load anything if nothing in the memory instructs CPU to do so? To overcome this dilemma, the computer architects design the cpu so that when someone press the power button and a reset signal is sent to the CPU, the CPU start reading instructions from a predefined address. Typically, that address point to a ROM (Read Only Memory), which contains BIOS.

??? question "Why CPU needs RAM to execute instruction?"
    Why RAM is needed for CPU to be able to execute instruction? This has to do with how CPU was designed. The instructions have to be stored somewhere so CPU can read them and then execute. The ideal candidate should be fast so it does not impact the CPU performance and also relatively cheap to produce.

    CPU itself contains some on chip storage known as registers. Those registers are fast in terms of read and write speed. Modern registers' speed are nearly the same as CPU's. However, registers are expensive to produce. Moreover, if registers get larger in capacity, speed will be negatively impacted and thus impact the performance of the CPU. Therefore, those on chip registers are not suited for the task of storing instructions. So RAM comes in place -- it is a lot less expensive than registers and much faster than secondary storage devices such as hard disk or SSD. Therefore, RAM gives CPU a fast storage medium for instructions while not letting anyone go bankrupt.

## BIOS

**BIOS**, stands for Basic Input Output System, is a collection of logic resides on a ROM (Read-Only Memory) of the motherboard. It is the first software to run when powered on.

Why do we need it? There are varieties of PC motherboards -- some have hardware that others do not or even if they have the same type of hardware, it is possible they need to be configured differently. BIOS was created to offer generalized low-level services to system programmers. BIOS, as provided by the motherboard manufacturer, provides system programmer an coherent view of the motherboard. Some of the functionality BIOS provided are:

* Video Display
* Storage Devices Access
* Memory Probe
* etc

??? question "Does CPU read BIOS from ROM?"
    The answer is yes and no. The CPU does need to read BIOS from ROM initially. However, the reading speed of ROM is too slow for modern needs of fast booting process. So a technique called shadowing is used.

    Shadowing essentially copy data from ROM into RAM for faster execution. The RAM area used is called shadow RAM.

## POST

When the computer is switched on, the BIOS does a series of diagnostics called **POST** - Power On Self Test. For IBM-compatible PC POST, the principal duties of BIOS during POST includes:

* Verify CPU registers
* Verify the integrity of the BIOS code
* Verify some basic components like timer, interrupt controller
* Find and verify main memory

The boot will proceed if and only if POST finds no problem.

## Master Boot Record

Right after POST, the BIOS checks bootable devices (Any  piece of hardware that can store files) for a boot signature, which is in a boot sector known as **MBR**. MBR is assumed to reside on the 1st sector on 1st track of a cylinder, under first head (See "Typical Hard Disk Geometry"). The boot signature contains bytes sequence 0x55, 0xAA at bytes offset 510 and 511 respectively. When the BIOS find such MBR, it is loaded into memory at 0x7c00 and the execution is transferred to MBR. Note the MBR cannot exceeds 512 bytes or one sector due to historical reasons (Someone arbitrarily decided the size. Then the design got popular and it became a standard.)  

The MBR contains a bootstrap program and Partition Table. The first 440 bytes of the MBR contains so called bootstrap code.
BIOS will load MBR to physical address 0x7c00 and then instruct CPU jumps to the beginning of the loaded MBR to start execute.

MBR table entries:

| Offset | Size(bytes) | Function |
|--------|:-----------:|---------:|
| 0x000  | 440[^1]         | MBR Bootstrap code |
| 0x1B8  | 4           | Optional: "Unique Disk ID" (Used to identify the drive) |
| 0x1BC  | 2           | Optional: Reserved 0x0000[^2] |
| 0x1BE  | 16          | First partition table entry   |
| 0x1CE  | 16          | Second partition table entry  |
| 0x1DE  | 16          | Third partition table entry   |
| 0x1EE  | 16          | Fourth partition table entry  |
| 0x1FE  | 2           | (0x55, 0xAA) "Valid bootsector" |

??? question "Why MBR is loaded at 0x7c00?"
    This has to be traced back to Intel's first x86 processor 8088. The IBM PC 5150 used this chip. The operating system on that PC was DOS 1.0, which requires minimal of 32 KB of memory. 

    ??? Note "Minimal Memory Requirement"
        Note minimal memory is just suggestion from the OS developer. But since most people goes with this suggestion, it is rather safe to make assumption about minimal available memory.
    Although 8088 can support up to 1 MB of memory, given it is a 16 bit processor, IBM needs to take in consideration that no everyone will have the budget to afford 1 MB of memory. So they assumed there were in total 32 KB memory. Then the BIOS developer team decided the following memory layout:

    | Offset | Size         | Function             |
    |--------|--------------|----------------------|
    | 0x0000 | 1 KB         | Real Mode IVT        |
    | 0x0400 | 256 bytes    | BDA (BIOS Data Area) |
    | 0x0500 | Almost 30 KB | OS load area         |
    | 0x7C00 | 512 bytes    | Boot Sector          |
    | 0x7E00 | 512 bytes    | Boot Data/Stack      |

    The first entry is IVT or interrupt vector table. If you know what is an interrupt, this table is for CPU to find the address of code to execute for each interrupt call. If not, you can just see it as a structure needed for using functions provided by BIOS for now. Next is the BIOS data area, which BIOS will be using to do operation. Third is where the OS will be loaded. Next two are the boot sector and data area it uses to boot. This layout was created to give as much contiguous memory to the operating system as possible so the Boot sector was stored at the end of the memory. How? IVT and BDA cannot be overwritten as they will be needed for BIOS to operate properly so those two has to stay at the top. <!-- TODO: finish this-->

??? "Typical Hard Disk Geometry"
    ![CHS](/img/CHS.png)
    *Typical Hard Disk Geometry(https://en.wikipedia.org/wiki/Cylinder-head-sector#/media/File:Hard_drive_geometry_-_English_-_2019-05-30.svg)*

## Bootloader

Give the limited size of the MBR bootstrap code, a typical bootstrap code will load other code from the disk and transfer control to that code to do other booting jobs. Those different code parts are called booting stages with code in MBR being stage 1. The combination of those code is typically called **bootloader**.

## CPU Mode

CPU modes are operating modes for the CPU, which specify the type and scope of operation that can be performed by the CPU. One example of type is the length of operation (16 bit or 32 bit) and scope can be if certain operation is allowed.

One of the remaining jobs of modern booting code is to switch from Real Mode to Protected Mode.

### Real Mode

Real Mode is a simple 16 bit mode that is present on all x86 processors. It was the first x86 mode design and this for compatibility purposes, all x86 processors begin execution in Real Mode.

In Real Mode, Segmentation is used to address memory. To be explicitly, Segment:Offset pair is used to address memory. Why do we have to do this instead of directly addressing the memory. The reason being the 8086 processor has 16 bit data bus width but also has 20 bit address lines connected to the main memory. This means the maximum memory the processor can address is $2^{20}$ byes or 1 MB.If using 16 bit to do direct memory addressing, then the total amount of addressable memory is restricted to $2^{16}$ bytes or 64 KB. To overcome this restriction, the engineers at Intel came up with so called Segmentation.

Since there are in total 1 MB addressable memory, we can divide them into 64 KB segments -- in total 16 segments. To address bytes in each segments, we would need $log_2(64*1024)=16$ bits as offset. So by using 16 bit segment and 16 bit offset, the processor now is able to address entire 1 MB memory. 

### Protected Mode

As the size of the memory get larger and the price per bytes drop drastically, Real Mode can no longer satisfy people who demand more memory. Also, Real Mode provides no means of protection as one program can override any of the 1 MB region and any program can execute any instructions they want. With those needs, Intel devise so called Protected Mode. Protected Mode gives programmers access to larger addressable memory and Privilege level to protect execution. We will further discuss Protected Mode in later sections.

[^1]: Can be extended to 446 bytes if override next two fields
[^2]: 0x0000 indicating read-write; 0x5A5A indicating read-only
