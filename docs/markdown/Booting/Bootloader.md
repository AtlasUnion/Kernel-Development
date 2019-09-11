# Bootloader

## Objective of our bootloader

In this section, we are going to code up a minimal bootloader. The bootloader will do the following:

* Enable A20 line
* Load Global Descriptor Table (GDT)
* load a minimal kernel
* Enter Protected Mode

## A20 line

The A20 line is the physical 21st bit line of memory access. A20 is disabled by default for historical reason. 

Intel 8086, 8088, 80186 process has 20 address line so in total we have A0 to A19 line to address memory. But each of the processor internal registers has only 16 bits. This means the maximum memory the processor can address is $2^{20}$ byes or 1 MB. If using 16 bit to do direct memory addressing, then the total amount of addressable memory is restricted to $2^{16}$ bytes or 64 KB. As we discussed in section Boot Sequence, in order to utilize 20 bits address bus, engineer at Intel devises what is called **Segmentation Addressing**.

### Segmentation Addressing

Segmentation Addressing divides 1 MB memory into 16 segments of 64 KB. Then the cpu will use Segment:Offset to address memory.

* Physical Address = Segment Value x 16 + Offset

Example Code:
``` asm
movb $0x00, %es:(%di)
```

Above code moves value 0x00 into physical address at (%es * 16 + %di).

### How is Segmentation Addressing related to A20 line

Since there were only 20 address lines, any segment:offset pair results in value larger than 1 MB will result in their 21st bit get truncated. This truncation has an effect of wrapping memory addresses. 

Take pair F800:8000 for example. The physical address of pair is F8000 + 8000 = 0x100000. But since there is not 21st line, the address will actually be 0x00000. 

Some engineers decided to use this wrap in this program. Thus, to provide backward compatibility, modern CPUs choose to disable A20 line by default to create the same effect of wrapping up.

But, our 32 bit kernel do need to address above 1 MB, and thus our bootloader needs to enable A20 line.

??? "What happen if I do not enable A20 line?"
    If A20 line is not enabled, every memory access's 21st bit will be asserted as 0. Thus can leads to unintended memory overwrite or read.

## Protected Mode



