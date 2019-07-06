# Bootloader

!!! Note
    This section can be skipped if you choose to use existing bootloader such as GRUB.

```asm
    Mmov $0x2, %ah                        ## function code
    mov $0x1, %al                        ## number of sectors to read
    mov $0x0, %ch                        ## track/cylinder number
    mov $0x0, %dh                        ## head number
    mov $0x2, %cl                        ## sector number
    mov $0x80, %dl                       ## drive number (0x80=drive 0)
    mov $0x7E00, %bx                     ## es:bx point to buffer                     
    int $0x13
```