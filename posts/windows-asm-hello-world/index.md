<!--
.. title: Windows ASM hello world
.. slug: windows-asm-hello-world
.. date: 2021-01-22 22:35:07 UTC+01:00
.. tags: #assembly, #windows, #win32api, #win64api, #assembler #linker
.. category: Exploit Development
.. link: 
.. description: 
.. type: text
-->

I have several notes about exploit development in linux system but today
I decided to start some test in windows OS, so like the first time on 
linux I tried to study shellcoding.
After some minutes I figured out that I missed a step: simple assembly and
calling conventions exploration.
So these are my first tests with code and tools 

__________________________________________________________________________

## With C function calls

### Solution 1 (nasm + gcc)

```assembly
bits 64
default rel

segment .data
    msg db "Hello world!", 0xd, 0xa, 0

segment .text
global main
extern ExitProcess

extern printf

main:
    push    rbp
    mov     rbp, rsp
    sub     rsp, 32

    lea     rcx, [msg]
    call    printf

    xor     rax, rax
    call    ExitProcess
```

Assembling:
```console
C:\Users\nobody\Desktop\assembly>nasm -f win64 hello.asm -o hello.obj
```

Linking:
```console
C:\Users\nobody\Desktop\assembly>gcc -o hello.exe hello.obj

C:\Users\nobody\Desktop\assembly>hello.exe
Hello world!
```

__________________________________________________________________________


### Solution 2 (nasm + link)

Thanks to:

- [stackoverflow](https://stackoverflow.com/questions/64413414/unresolved-external-symbol-printf-in-windows-x64-assembly-programming-with-nasm)
- [microsoft docs](https://docs.microsoft.com/en-us/cpp/porting/visual-cpp-change-history-2003-2015)

"The definitions of all of the printf and scanf functions have been moved
inline into <stdio.h>, <conio.h>, and other CRT headers. This breaking 
change leads to a linker error (LNK2019, unresolved external symbol) for
any programs that declared these functions locally without including the 
appropriate CRT headers.If possible, you should update the code to include
the CRT headers (that is, add #include <stdio.h>) and the inline functions,
but if you do not want to modify your code to include these header files, 
an alternative solution is to add an additional library to your linker
input, legacy_stdio_definitions.lib."

```assembly
bits 64
default rel

segment .data
    msg db "Hello world!", 0xd, 0xa, 0

segment .text
global main

extern printf
extern ExitProcess
extern _CRT_INIT

main:
    push    rbp
    mov     rbp, rsp
    sub     rsp, 32

    call _CRT_INIT  ;CRT initialization

    lea     rcx, [msg]
    call    printf

    xor     rax, rax
    call    ExitProcess
```

Assembling: 
```console
C:\Users\nobody\Desktop\assembly>nasm -fwin64  hello.asm -o hello.obj 
```

Linking:
```console
C:\Users\nobody\Desktop\assembly>link hello2.obj /subsystem:console /entry:main /out:hello2.exe kernel32.lib legacy_stdio_definitions.lib  msvcrt.lib
```

__________________________________________________________________________


## With Win32API calls

32-bit windows api uses stdcall calling convention, so the parameters are
passed on the stack from right to left and the return value is stored into
eax register.

```assembly
   global  go

        extern  _ExitProcess@4
        extern  _GetStdHandle@4
        extern  _WriteConsoleA@20

        section .data
msg:    db      "Hello, World", 10
handle: db      0
written:
        db      0

        section .text
go:
        ; handle = GetStdHandle(-11)
        push    dword -11
        call    _GetStdHandle@4
        mov     [handle], eax

        ; WriteConsole(handle, &msg[0], 13, &written, 0)
        push    dword 0
        push    written
        push    dword 13
        push    msg
        push    dword [handle]
        call    _WriteConsoleA@20

        ; ExitProcess(0)
        push    dword 0
        call    _ExitProcess@4
```

Assembling:
```console
C:\Users\nobody\Desktop\assembly>nasm -fwin32 -o hello_win32.obj hello_win32.asm
```

Linking:
```console
C:\Users\nobody\Desktop\assembly>link  hello_win32.obj "C:\Program Files (x86)\Windows Kits\10\lib\10.0.18362.0\um\x86\kernel32.lib" /subsystem:console /entry:go /machine:x86
```

__________________________________________________________________________

## With win64API calls

64-bit windows api uses fastcall calling convention, so the first four
parameters are passed left to right into:

1. rcx
2. rdx
3. r8
4. r9

The caller must reserve space on the stack even for these parameters, and
the stack must be aligned

```assembly
global main
extern GetStdHandle
extern WriteFile

section .text
main:
    mov     rcx, 0fffffff5h
    call    GetStdHandle

    mov     rcx, rax
    mov     rdx, NtlpBuffer
    mov     r8, [NtnNBytesToWrite]
    mov     r9, NtlpNBytesWritten
    sub     rsp, 40
    mov     dword [rsp + 32], 00h
    call    WriteFile
    add     rsp, 40
ExitProgram:
    xor     eax, eax
    ret

section .data
NtlpBuffer:        db    'Hello, World!', 00h
NtnNBytesToWrite:  dq    0eh

section .bss
NtlpNBytesWritten: resd  01h
```

Assembling:
```console
C:\Users\nobody\Desktop\assembly>nasm -f win64 test.asm
```

Linking:
```console
C:\Users\nobody\Desktop\assembly>gcc -s -o test.exe test.obj
```

or linking with:
```console
C:\Users\nobody\Desktop\assembly>link hello_win64.obj "C:\Program Files (x86)\Windows Kits\10\lib\10.0.18362.0\um\x64\kernel32.lib"  /subsystem:console /entry:main /LARGEADDRESSAWARE:NO /out:test.exe
```
__________________________________________________________________________

## Tools

- link and gcc are included into VS C/C++ toolchain
- [nasm] (https://nasm.us)

