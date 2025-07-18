# Understanding Hello, World!

Everyone starts their coding journey with a `Hello, World!\n` program.

For example, it takes only 4 lines to do this in C:

{% code title="hello.c" %}
```c
#include<stdio.h>

int main(void){
  printf("Hello, World!\n");
}
```
{% endcode %}

But how does it really works? This is what we are going to understand here.

It is really complex, but equally magnificent. Creating one single article would make it boring as it is very long. Therefore, it would come in a series of articles, which are in proper order.

1. [A high level overview of the build process](https://ankuragrawal.gitbook.io/home/understanding-hello-world/a-high-level-overview-of-build-process-in-c)
2. [Introduction to ELF format](https://ankuragrawal.gitbook.io/home/understanding-hello-world/introduction-to-elf)
3. [Introduction to processes in Linux](https://ankuragrawal.gitbook.io/home/understanding-hello-world/introduction-to-processes-in-linux)
4. [Why `void main();` is technically incorrect?](https://ankuragrawal.gitbook.io/home/understanding-hello-world/why-main-function-shouldnt-be-of-type-void)
5. [A high level overview of the movement](macro-level-roadmap.md)
6. [Moving from C source to assembly (x64 Intel, of course)](c-greater-than-assembly.md)
7. [Inspecting the object code](object-code-analysis/)
8. [Inspecting the linked ELF Binary](linked-elf-analysis/)













