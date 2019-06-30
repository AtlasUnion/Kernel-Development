# Primer: Probe Memory

The first thing to do in booting is almost always checking the available memory. Why? Because without a complete map of the memory, it is impossible to decide where to load our kernel. So in this section, we are going to write a program in assembly to probe usable memory.

## Project Description

For this project, we are going to write some assembly code to get a list of memory region and their status (starting address, length, etc). Then we are going to print those information on screen.

## What you should know before getting start

### Why coding in assembly?

It is certainly a pain to code in assembly. Then why do we need to do it? There is an argument that coding in assembly results in the program run faster than those coding in higher level language like C. But that is not the reason as that argument is not longer valid today as modern compliers are sophisticated enough to generate highly optimized program. The performance difference between a human written assembly and machine generated assembly is subtle and it would requires a highly skilled assembly programmer to outperform modern complier.

Then, why do we need to code in assembly for this section? The reason is C complier like gnu gcc generates code for protected mode machine and we are working in real mode. So till someone writes a complier for real mode, it would be necessary to write assembly.

### How to probe memory

To get our list of memory region, we need to visit our friend, BIOS. Why don't we detect the memory ourselves? After all, if BIOS can do it, we can just do what BIOS does. Unfortunately, the answer is disappointing -- first, BIOS cannot use any RAM until it detect the type of RAM installed, then detect the size of each memory module, then configure the chipset specific methods. The RAM is totally unusable during this process. As BIOS is initally running from ROM, it can do the all above without RAM. So you see it is impossible for us to do our own memory probing. 

#### Probe Memory with ```int 0x15h, eax = 0xE820h```

int 0x15h is the assembly instruction we will use to let BIOS detect the memory for us. int stands for interrupt. When CPU encounter any of int instructions, it will go to a table in RAM called interrupt vector table and find the entry associated with the number after int, which contains an address to a function. In real mode, BIOS set up this table in RAM for us to use. 

To use this function that BIOS provides us, we need to put some parameters in some registers as follows:

| Register | Function       | Contains                                                                                                                                                                                 |
|----------|----------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| eax      | Function code  | e820h                                                                                                                                                                                    |
| ebx      | Continuation   | Contains the "continuation value" to  get the next run of physical memory. This value is returned by a previous call to this function. If this is the first call, ebx must contain zero. |
| es:di    | Buffer Pointer | Pointer to a buffer where BIOS will returns a structure that describes a certain  memory range                                                                                           |
| ecx      | Buffer Size    | To be requested structure size in bytes                                                                                                                                                  |
| edx      | Signature      | 'SMAP' - used by the BIOS to verify the caller is requesting the system map                                                                                                              |

And after the function call, the BIOS will return as following:

| Registers | Function       | Contains                                                                                                                         |
|-----------|----------------|----------------------------------------------------------------------------------------------------------------------------------|
| cf        | Carry Flag     | If no carry, indicates no error. If try to make call after final structure while ecx not zero, cf will be set.                   |
| eax       | Signature      | 'SMAP' - returned to verify correct BIOS revision                                                                                |
| es:di     | Buffer Pointer | Same as on input                                                                                                                 |
| ecx       | Buffer Size    | Number of bytes returned by BIOS                                                                                                 |
| ebx       | Continuation   | Contains continuation value to get the structure of next memory range. A return value of zero indicates the end of memory range. |

Let's demonstrate how to use this via some code.

```asm
probe_mem:
    mov $0, %eax
    mov %eax, %es           ## clear %es
    movl $0, 0x8004
    mov $0x8008, %di        ## set di to 0x8008 so first 4 bytes can be used to store number of entries and next 4 used to store total memory length
    xor %ebx, %ebx          ## first call, ebx must be 0
    xor %bp, %bp            ## use bp to stors # of entires
    mov $0x0534D4150, %edx  ## move 'SMAP' signature to edx, used by BIOS to verify the caller is requesting the syste map to be returned in ES:DI
    mov $0xe820, %eax       ## function code 
    movl $1, %es:20(%di)    ## force a valid ACPI 3.x entry
    mov $24, %ecx           ## request 24 bytes
    int $0x15               
```

Most of the code ard aided by the comments and should be self-explanatory. There is only one thing that needs to be addressed -- why do we choose to set di to 0x8008?
<!-- Consider moving from 0x8008 to other places as there is no guarantee current memory address exist in RAM-->
This has to do with the memory map of x86 machine -- with our 512 bytes bootsector loaded at 0x7c00, it spans from that address to 0x7dff. Right after the bootsector, starting from 0x7e00 to 0x7FFFF are RAM guaranteed free to use.

Note that this is just the first call to BIOS so there will only be one structure returned from BIOS. To get complete memory map, we still need to make few more calls to BIOS. 

Be sure to try to implement your own solution before checking the below complete code.

??? Note "Complete code"
    ```asm
    probe_mem:
        mov $0, %eax
        mov %eax, %es           ## clear %es
        movl $0, 0x8004
        mov $0x8008, %di        ## set di to 0x8008 so first 4 bytes can be used to store number of entries and next 4 used to store total memory length
        xor %ebx, %ebx          ## first call, ebx must be 0
        xor %bp, %bp            ## use bp to stors # of entires
        mov $0x0534D4150, %edx  ## move 'SMAP' signature to edx, used   by BIOS to verify the caller is requesting the syste map to be returned in ES:DI
        mov $0xe820, %eax       ## function code 
        movl $1, %es:20(%di)    ## force a valid ACPI 3.x entry
        mov $24, %ecx           ## request 24 bytes
        int $0x15               
        jmp in_entry 
    loop_entry:
        mov $0xe820, %eax       ## function code
        movl $1, %es:20(%di)    ## force ACPI 3.X entry
        mov $24, %ecx           ## ask for 24 bytes
        int $0x15 
        jc loop_done            ## if carry set -> end of the list
        mov $0x0534D4150, %edx  ## SMAP
    in_entry:
        jcxz skipentry          ## jcxz: jump if ecx is zero => skip empty enrty
        cmp  $20, %cl           ## get a 24 byte ACPI 3.x entry?
        jbe notext              ## if %cl <= 20
        testl $1, %es:20(%di)   ## is "ingore the entry" bit set
        je skipentry            ## is set
    notext:
        mov %es:(%di), %ecx     ## get lower uint32_t of memory region  length
        or %es:12(%di), %ecx    ## or with upper uint32_t to test for   zero
        jz skipentry            ## length is zero, skip the entry
        inc %bp                 ## entry_count++
        mov %es:8(%di), %eax    ## store length into eax
        add %eax, 0x8004        ## add length to 0x8004 TODO: init  0x8004
        add $24, %di
    skipentry:
        test %ebx, %ebx         ## AND %ebx, %ebx => check if ebx is zero => if ebx is zero, the list is complete
        jne loop_entry
    loop_done:
        mov %bp, 0x8000
        clc                     ## clear carry flag
        ret
    ```
