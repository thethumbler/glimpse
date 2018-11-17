---
layout: post
title:  "Hello, World! without BIOS"
date:   2018-11-16
categories: boot bios
---
You probably studied (or pursued) Computer Organization at some point, you know how hardware works basically, maybe you even tried playing with Arduino or AVR (or even better, ARM). But how does your computer work? what happens when you push that power button?

You probably also heard of BIOS and the magic stuff it does to boot your computer.. but how does it work under the hood? could you write your own BIOS? That's exactly what we will do, make our computer do something without the BIOS (i.e. write our BIOS-like thingy).

In this article, I'll assume that you know the basics of Computer Organization and Assembly. Also you are using a Linux based distro (if not, you know what you are doing) and have `nasm` and `qemu` installed.

## CPU Architecture
/* TODO */
## Power on sequence
When you push the power button, the motherboard powers on the CPU (there is no defined way of how this occurs, we only concern ourselves with how the CPU sees it). The process starts by doing some initialization steps (like choosing the bootstrap processor in a multi-processor setup) and then goes into a known defined state (it wouldn't make sense otherwise, imagine the processor goes executing code from mars on power up). To know that state, we should consult the manual of the processor.

For our experiment, the most concerning values are
```
EIP = 0000FFF0H

CS:
Selector = F000H
Base     = FFFF0000H
Limit    = FFFFH
```

## Simple BIOS
Let's write the simplest code ever, one that does absolutely nothing
{% highlight nasm %}
; Our file is loaded at 0xFFFF0000, we offset that by 0xFFF0 to get 0xFFFFFFF0
times 0xFFF0 - ($ - $$) db 0

; Execution starts here
jmp $

; Our file must be exactly 64k in size
times 0x10000 - ($ - $$) db 0
{% endhighlight %}

to compile this file into a (flat) binary image, use the following command
{% highlight bash %}
$ nasm -f bin -o bios.bin bios.asm
{% endhighlight %}

and to run it, use
{% highlight bash %}
$ qemu-system-i386 -bios bios.bin
{% endhighlight %}


## Using UART for output
/* TODO */

## Putting it all together

{% highlight nasm %}
%define fill(x) times x - ($ - $$) db 0

%macro inb 1    ; (port) => returns al
    mov dx, %1  ; port
    in  al, dx
%endmacro

%macro outb 2   ; (port, value)
    mov dx, %1  ; port
    mov al, %2  ; value
    out dx, al
%endmacro

%define PORT 0x3F8

uart_init:
    ; Disable all interrupts
    outb (PORT + 1), 0x00
    ; Enable DLAB to set baudrate divisor (base=115200)
    outb (PORT + 3), 0x80
    ; Set divisor low byte to 1 
    outb (PORT + 0), 0x01
    ; Set divisor high byte to 0
    outb (PORT + 1), 0x00
    ; 8 bits, no parity, one stop bit
    outb (PORT + 3), 0x03
    ret

uart_putc:
    inb (PORT + 5)
    and al, 0x20
    jz uart_putc
    outb PORT, bl
    ret

uart_puts:
    mov bl, byte [ds:si]
    or bl, bl
    jz .done
    call uart_putc
    inc si
    jmp uart_puts
.done:
    ret

_start:
    call uart_init
    mov ax, 0xF000
    mov ds, ax
    mov si, msg
    call uart_puts
    jmp $

msg: db "Hello, World!", 0

; Our file is loaded at 0xFFFF0000, we offset that by 0xFFF0 to get 0xFFFFFFF0
fill(0xFFF0)

; Execution starts here
jmp 0xF000:_start

; Our file must be exactly 64k in size
fill(0x10000)
{% endhighlight %}
