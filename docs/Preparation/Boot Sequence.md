Have you ever wondered what happen from the moment you press the power botton to the familar Operating System shows up in front of you? If you do, then it is good news for you, we are going to demystify the whole process in this section.

## BIOS
**BIOS**, stands for Basic Input Output System, is a collection of logic resides on a ROM (Read-Only Memory) of the motherboard. It is the first software to run when powered on.

Why do we need it? There are varieties of PC motherboards -- some have hardware that others do not or even if they have the same type of hardware, it is possible they need to be configured differently. BIOS was created to offer generalized low-level services to system programmers. BIOS, as provided by the motherboard manufacturer, provides system programmer an coherent view of the motherboard. Some of the functionality BIOS provided are:

* Video Display
* Storage Devices Access
* Memory Probe
* etc

## POST
When the computer is switched on, the BIOS does a series of diagnostics called **POST** - Power On Self Test. For IBM-compatible PC POST, the principal duties of BIOS during POST includes:

* Verify CPU registers
* Verify the integrity of the BIOS code
* Verify some basic components like timer, interrupt controller
* Find and verify main memory

## Master Boot Record
Right after POST, the BIOS checks bootable devices (Any  piece of hardware that can store files) for a boot signature, which is in a boot sector known as **MBR** (sector number 0, See Below Image). The boot signature contains bytes sequence 0x55, 0xAA at bytes offset 510 and 511 respectively. When the BIOS find such MBR, it is loaded into memory at 0x7c00 and the executation is transfered to MBR. Note the MBR cannot exceeds 512 bytes or one sector due to historical reasons (Someone arbitrarily decided the size. Then the design got popular and it became a standard.)  

<!-- Remember to add link to Partition Table-->
The MBR contains a bootstrap program and Partition Table. The first 440 bytes of the MBR contains so called bootstrap code. Give the limited size of the code, a typical bootstrap code job is to load other code from the disk and transfer control to that code to do other booting jobs, especially loading kernel into memory.

![CHS](/img/CHS.png)

*Typical Hard Disk Geometry*

## CPU Mode
One of the remaining jobs of modern bootstrapping code is to switch from Real Mode to Protected Mode. What is Real Mode and Protected Mode and what are they for?
### Real Mode
Real Mode is a simple 16 bit mode that is present on all x86 processors. It was the first x86 mode design and this for compatibility purposes, all x86 processors begin execution in Real Mode.

In Real Mode, Segmentation is used to address memory. To be explicitly, Segment:Offset pair is used to address memory. Why do we have to do this instead of directly addressing the memory. The reason being the 8086 processor has 16 bit data bus width but also has 20 bit address lines connected to the main memory. This means the maximum memory the processor can address is $2^{20}$ byes or 1 MB.If using 16 bit to do direct memory addressing, then the total amount of addressable memory is restricted to $2^{16}$ bytes or 64 KB. To overcome this restriction, the engineers at Intel came up with so called Segmentation.

Since there are in total 1 MB addressable memory, we can divide them into 64 KB segments -- in total 16 segments. To address bytes in each segments, we would need $log_2(64*1024)=16$ bits as offset. So by using 16 bit segment and 16 bit offset, the processor now is able to address entire 1 MB memory. 
### Protected Mode
As the size of the memory get larger and the price per bytes drop drastically, Real Mode can no longer satisfy people who demand more memory. Also, Real Mode provides no means of protection as one program can override any of the 1 MB region and any program can execute any instructions they want. With those needs, Intel devise so called Protected Mode. Protected Mode gives programmers access to larger addressable memory and Privilege level to protect execution. We will further discuss Protected Mode in later sections.