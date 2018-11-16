---
layout: post
title:  "Hello, World! without BIOS"
date:   2018-11-16
categories: boot bios
---
In this article, I'll assume that you know the basics of Computer Organization and Assembly.

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
