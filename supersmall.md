# SuperSmall

A really small executable program.

## Index
 - [Index](#index)
 - [Intro](#intro)
 - [The Program](#the-program)
 - [Making it Smaller](#making-it-smaller)
   - [Stripping](#stripping)
   - [--omagic](#--omagic)
   - [Optimizing the Assembly](#optimizing-the-assembly)
   - [ELFkickers](#elfkickers)
 - [Bonus](#bonus)

## Intro

At first this started as a learning adventure after finding that there is a CPUID instruction on x86. After getting it to print the CPU Vendor ID on the screen, I noticed that my ~1kB assembly got transformed into a 4.6kB program.

## The Program
*You can ignore this and move to [Making it smaller](#making-it-smaller)*

We want to explore the CPUID instruction on x86. The instruction can be used by a OS or program to get information about the CPU it is running on.

We only care about testing it out, so we will only request the Vendor ID (Who is the manufacturer?).

As you probably guessed, we will have to use assembly for this.

Because we will use only the GNU Assembler and Linker we need to provide the _start function. I will provide the equivalent to each syscall in C to make the code easier to understand.

```asm
.global _start
_start:
    # Get some space on the stack (12 bytes, that is the max size for the vendor ID)

    sub $8, %rsp

    # to request the vendor id, we run cpuid with 0 on rax
    mov $0, %rax
    cpuid

    # the cpu should place the string on ebx, edx, and ecx (In this exact order)
    # we need to move the string to the stack, so we can print it
    movl %ebx, 0(%rsp)
    movl %edx, 4(%rsp)
    movl %ecx, 8(%rsp)

    # to print we will use the write syscall
    # following is equiv to : write(STDOUT_FILENO, str, 12)
    mov $1, %rax            # write syscall is n. 1
    mov $1, %rdi            # stdout is file n. 1
    mov %rsp, %rsi          # the pointer to the string is the current value of the stack
    mov $12, %rdx           # length of the string (write does no check for null termination)
    syscall

    # restore the stack to its original size
    add $8, %rsp

    mov $60, %rax   #
    mov $0, %rdi    # equiv to : exit(0);
    syscall         #
```

To compile this I used:
```sh
as cpuid.s -o cpuid.o
ld cpuid.o -o cpuid
```

## Making it Smaller
As I said before, the resulting executable has 4.6 kB. So I went on a journey to make it even smaller.

### Stripping
The first thing we can do is remove all debugging simbols from the executable. This is usually called stripping.

You can do this manually with the strip command, or pass the `-s` flag to the linker for it to do it for you:

```sh
ld cpuid.o -o cpuid -s
```

With this we got a executable with 4.3 kB. A 300 byte saving.

### --omagic
After some searching and trial and error, I found out about the `--omagic` flag for ld. *Read more [here](https://ftp.gnu.org/old-gnu/Manuals/ld-2.9.1/html_node/ld_3.html).*

This sets the text and data section to be both readable and writable. This also removes the page aligning of the data segment.

```sh
ld cpuid.o -o cpuid -s --omagic
```

With this we got all the way down to 400 bytes. Saving us 3.9 kB.

### Optimizing the Assembly
Most of you will be happy with saving 4.2 kB with just 2 flags, but not me.

I decided to start optimizing the assembly.

I found out that replacing the *mov $v, %r* instructions with *xor %r, %r* (set the register to 0) and then *inc %r* saved 1 byte.

I also found that reserving the space on the stack is not really needed. (Don't know why, but it doesn't SEGFAULT)

Also, we don't really need to exit with 0, we can exit with any value, as long as we exit (To avoid SEGFAULT).

With all this I got to:
```asm
.global _start
_start:
    xor %eax, %eax      # request the vendor id
    cpuid

    # move the vendor id to the stack
    movl %ebx, 0(%rsp)
    movl %edx, 4(%rsp)
    movl %ecx, 8(%rsp)

    # write(STDOUT_FILENO, "AuthenticAMD", 12)
    xor %rax, %rax          # syscall 1 : write
    inc %rax
    xor %rdi, %rdi          # fd 1 : stdout
    inc %rdi
    mov %rsp, %rsi          # pointer to the string
    mov $12, %rdx           # length of the string
    syscall

    # exit(1)       it is probably 1 because of the stdout file number
    mov $60, %rax           # exit with whatever is on the rdi register
    syscall
```

This gets us all the way down to 384 bytes. A mere 16 byte saving.

### ELFkickers
After some time I found out about a collection of programs called [ELFkickers](http://www.muppetlabs.com/~breadbox/software/elfkickers.html)

We only care about *sstrip*, as it removes some extra stuff the linker left when stripping

```sh
sstrip cpuid
```

This finnaly gets us down to 168 bytes, saving 216 bytes.

## Bonus
I recommend reading this article (also from muppetlabs.com) about a really small ELF executable, even smaller than ours: [link](http://www.muppetlabs.com/~breadbox/software/tiny/teensy.html).

If we only had a call to exit, like this:
```asm
.global _start
_start:
    mov $60, %rax
    syscall
```
Following the previous steps, it would only take 129 bytes. This also means that the actual functional part of the program (without the exit) only takes 39 bytes.