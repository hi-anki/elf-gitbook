---
description: >-
  This article explores Executable & Linkable File Format (popularly known as
  ELF)
layout:
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: false
---

# Understanding ELF

The article is structured into two parts.

1. How the source code is compiled into an executable file?
2. What is an executable file?

## Part 1 - The Journey Of From Source Code To Executable

This is a simple C program

```c
// hello.c
#include <stdio.h>
​
int main(){
  printf("Hello, World!\n");
  return 0;
}
```

which we can compile using GNU C Compiler or `gcc` with the following command

```bash
gcc hello.c -o hello_executable
./hello_executable
​
Hello, World!
```

This simple process involves a lot of complexity, which is hidden under abstractions. The first thing we need to do is to remove this abstraction and understand what happens under the hood.

A source code becomes an executable file after following these 4 steps -

1. Preprocessing
2. Compilation
3. Assembling
4. Linking

Lets dive into each.

### Preprocessing

Every C code at least includes this one line, #include \<stdio.h>, where #include is a preprocessing directive.

These directives are needed to be processed (or extended) before we can move any further.

This preprocessing is carried out using `gcc -E hello.c -o hello.i`.

The file obtained here is an intermediate file with `.i` extension, which is a raw-c file with all the preprocessing directives resolved into actual references.

**Note:** If you look at `hello.i` and `stdio.h` simultaneously, you will find that it is not exactly copied. It is because the header file itself has got various macros. And the extension is based on those macros.

### Compilation

The intermediate C code is compiled into assembly instructions, which is the closest we can get to CPU yet retaining some readability.

The flavor of assembly (Intel or AT\&T) depends upon the assembler program used to compile the source code. For example,

* If GNU Assembler (or`as`) is used, it generates AT\&T assembly by default. Although it can be configured to generate Intel assembly as well. Same is followed by `gcc`.
* If netwide assembler (or`nasm`) is used, it generates Intel assembly.

The architecture specific things (x86 and x86\_64) are taken care by the assembler itself.

To compile intermediate C code into assembly code, we can run the following command:

```bash
gcc -S -M masm=intel hello.i -o hello.s
```

### Assembling

The assembly code is converted into object code, or machine code. This is the actual code which gets executed on the CPU.

The object code can be obtained as:

```bash
gcc -c hello.s -o hello.o
# OR
as hello.s -o hello.o
```

Here, the assembly instructions are translated into binary opcodes (operation codes).

The file obtained in this step is an object file with `.o` extension.

Object files are strict in their structure. They follow a format known as **Executable File Format**, popularly known as ELF.

But this object file is not an executable yet. It needs to be linked.

### Linking

To give an object code execution power, we need to link it with specific libraries.

In the above program, we are using a function called `printf` for printing `Hello, World!` to the output.&#x20;

* Where is that function coming from? The header file!
* Where is the header file coming from? `glibc`!
* Where is `glibc`? Somewhere in the OS!

An object code has unresolved references to various library functions. Until these are resolved, how can you execute a file?

Linking can be static or dynamic. Both have their use cases.

Dynamic linking is the preferred linking mechanism by a compiler. But it can be used to statically link as well.

Now we are ready to execute our binary.

It is a surface level overview to what actually happens under the hood. We will dive deeper into this when we start reverse engineering. Until then, we have enough knowledge to move forward.

## Part 2 - Understanding An Executable

An executable file is a type of computer file containing instructions that the computer's processor can directly execute.

Executable files differ greatly on Windows and Linux. The reason is that both the environments are designed differently.

| Property                   | Windows                                      | Linux                                   |
| -------------------------- | -------------------------------------------- | --------------------------------------- |
| Binary File Format         | Executable and Linkable File Format (or ELF) | Portable Executable File Format (or PE) |
| File Extension             | No extension required                        | `.exe`, `.dll`                          |
| System Call Interface      | Linux System Calls (`syscall`)               | Windows NT syscall layer or WinAPI      |
| Dynamic Linker/Loader      | ld-linux.so                                  | Windows Loader                          |
| C Runtime Library          | GNU C Library (or `glibc`)                   | Microsoft C Runtime (`MSVCRT`)          |
| Linker                     | `ld`                                         | `link.exe`                              |
| Calling Convention         | System V AMD64 ABI                           | Microsoft x64 ABI                       |
| Format For Dynamic Linking | Shared objects (`.so`)                       | Dynamic Link Library (`.dll`)           |

This time, we are learning about Linux environment.

### Brief History

It is a common standard file format for executable files, object code, shared libraries, and core dumps on Linux.

It was first released in "_System V ABI, Release 4_".

In 1999, it was chosen as the standard binary file format for Unix and Unix-like systems on x86 processors by the 86open project.

### Modern ELF

ELF is a general-purpose file format in Linux ecosystem, which defines the structure for binaries, libraries, and core files.

It is used throughout the build and execution pipeline.

* Different "types" of ELF files (relocatable, executable, shared object, core) exist to serve different roles in this pipeline.
* These are typically created during specific phases.

### Structure Of An ELF File

Regardless of the type of ELF, the structure of an ELF file remains mostly the same.

An ELF file can be divided into 4 parts.

<table><thead><tr><th width="222">Part</th><th>Importance</th></tr></thead><tbody><tr><td>ELF Header</td><td>Identifies the file as ELF and file metadata.</td></tr><tr><td>Program Header Table</td><td>Used by the dynamic loader at runtime.</td></tr><tr><td>Section Header Table</td><td>Used by the linker at build time.</td></tr><tr><td>Data (segments/sections)</td><td>Includes things which are referred by the above 2 tables, such as, text (code), data (globals), bss (uninitialized data), and others.</td></tr></tbody></table>

Lets take an example to understand ELF.

```c
// hello.c
#include <stdio.h>
​
int main(){
  printf("Hello, World!\n");
  return 0;
}
```

The object code can be obtained by

```bash
gcc -c hello.c hello_object.o
```

Object files are binary representations of a program's source code, intended to be executed directly on a processor, which is why they are required to follow a consistent structure.

ELF is not reserved for executable files only. It is used by a variety of other files of the same genre, and object code (.o), shared object (.so) libraries are two of them.

ELF files aren't readable by a text editor. Therefore, we use some command line utilities. The two most widely used and versatile utilities are `objdump` and `readelf`. There exist other like `nm`, `xxd`, `hexdump` and so on.  And they'll be introduced as needed.

### Understanding The Result Of \`file\`

We can check the type of this object file using the `file` utility.

```bash
file hello_object.o

hello_object.o: ELF 64-bit LSB relocatable, x86-64, version 1 (SYSV), not stripped
```

* `LSB` tells that the binary is structured in `little-endian` format, which means that the the least significant bit (or LSB) comes first.
  * It is different from `big-endian` where the MSB comes first.
  * In simple words, normal arithmetic is big-endian based and cpu arithmetic is little-endian based.
* `relocatable` means that the file is ready to be linked with shared libraries but not for execution.
  * A relocatable ELF is one which has unresolved references to symbols whose definition lies in shared libraries.
  * For example, `printf` comes from `glibc`.
* `not stripped` means that file still contains items which are not necessary and the code will still function the same if they are removed.
* `version 1 (SYSV)` means it uses System V AMD64 ABI (for conventions).
* `x86-64` tells we are on 64-bit Linux.

### Reading Object Dump

`objdump` is a utility which helps in inspecting object files and displaying information from them.

Since it is a very comprehensive tool, it is not in our interest to go through every flag. The most relevant ones include these.

| Command                       | Use Case                                       |
| ----------------------------- | ---------------------------------------------- |
| `objdump -d -M intel hello.o` | Disassemble the object file using Intel syntax |
| `objdump -t hello.o`          | Gives the symbol table                         |
| `objdump -r hello.o`          | Gives the relocation table                     |
| `objdump -h hello.o`          | Gives the section headers table                |

Lets go through them one by one.

#### Inspecting Disassembly

```bash
$ objdump -d -M intel hello.o

main.o:     file format elf64-x86-64
Disassembly of section .text:

0000000000000000 <main>:
   0:   55                      push   rbp
   1:   48 89 e5                mov    rbp,rsp
   4:   48 8d 05 00 00 00 00    lea    rax,[rip+0x0]        # b <main+0xb>
   b:   48 89 c7                mov    rdi,rax
   e:   e8 00 00 00 00          call   13 <main+0x13>
  13:   b8 00 00 00 00          mov    eax,0x0
  18:   5d                      pop    rbp
  19:   c3                      ret
  
 # Address    Hex-Encoded    Disassembly
 # Offset     Machine Code
```

Since we didn't used any optimizations, the result is quite verbose.

Since the program is not loaded into memory, it is quite logical to understand that the address of the `main` function, which is `0000000000000000` here, is just a placeholder value. This is resolved when the program is loaded.&#x20;

These address offsets are relative to where the `main` resides.

Everything here is hex-encoded.

* There are 16 0's in `0000000000000000`.
* A single hex digit is half a byte. Therefore, it is just a 8-byte address or a 64-bit address.

It is quite logical to ask how does 55 translates to `push rbp`. And the answer is opcodes.

* Each operation in assembly has a binary opcode.
* These opcodes are used to translate an assembly instruction into machine code.















