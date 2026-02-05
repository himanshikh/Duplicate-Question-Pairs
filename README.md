# Understanding Virtual Memory on Linux

This tutorial will guide you through the process of probing the virtual memory infrastructure on a Linux system. We will use a simple C program to allocate memory and then use command-line tools to inspect the state of that memory.

## 1. What is Virtual Memory?

In a nutshell, virtual memory is a memory management capability of an operating system (OS) that uses hardware and software to allow a computer to compensate for physical memory shortages by temporarily transferring data from random access memory (RAM) to disk storage.

Each process on a Linux system gets its own virtual address space. This address space is a set of memory addresses that the process can use. The kernel and the Memory Management Unit (MMU) on the CPU work together to translate these virtual addresses into physical addresses in RAM.

## 2. The Tools

We will use the following tools to inspect the virtual memory of a process:

*   `/proc` filesystem: A virtual filesystem that provides a wealth of information about processes.
*   `/proc/[pid]/maps`: Shows the memory mappings of a process.
*   `/proc/[pid]/pagemap`: Provides information about the mapping of each virtual page to a physical page frame.

## 3. The `prober` Program

Let's start with a simple C program that allocates some memory and then waits for input. This will give us a running process to inspect.

Create a file named `prober.c` with the following content:
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h> 

int main() {
    char *p = malloc(1024);
    if (p == NULL) {
        perror("malloc");
        return 1;
    }

    printf("Allocated 1KB of memory at address: %p\n", p);
    printf("Process ID: %d\n", getpid());
    printf("Press Enter to exit...\n");

    // Touch the memory to make sure it's mapped
    p[0] = 'a';

    getchar();

    free(p);
    return 0;
}
```

## 4. Compiling and Running the `prober`

To compile the `prober.c` file, we can use `gcc`. We will also create a `Makefile` to make this easier.

Create a file named `Makefile` with the following content:
```makefile
all: prober pagemap_reader

prober: prober.c
	gcc -o prober prober.c

pagemap_reader: pagemap_reader.c
	gcc -o pagemap_reader pagemap_reader.c

clean:
	rm -f prober pagemap_reader
```

Now, you can compile the program by running `make`.

## 5. Probing the Virtual Memory

Now that we have a running process, we can start probing its virtual memory.

**Step 1: Run the `prober` program.**

```bash
./prober
```

You will see an output similar to this:
```
Allocated 1KB of memory at address: 0x55a6d0f5a2a0
Process ID: 12345
Press Enter to exit...
```

**Step 2: Inspect the memory mappings.**

Open a new terminal and use the `cat` command to view the memory mappings of the `prober` process. Replace `12345` with the process ID of your `prober` program.

```bash
cat /proc/12345/maps
```

You will see a list of memory mappings. Look for the one that corresponds to the heap. It will look something like this:

```
55a6d0f5a000-55a6d0f7b000 rw-p 00000000 00:00 0                                  [heap]
```

The address `0x55a6d0f5a2a0` from the `prober` output should fall within this range.

**Step 3: Inspect the page table.**

Now, let's find out which physical page this virtual address is mapped to. We will use the `/proc/[pid]/pagemap` file for this. This is a binary file, so we can't just `cat` it. We need a program to parse it. We have created a program called `pagemap_reader` for this purpose.

**Step 4: Run `pagemap_reader`**

In the same terminal where you inspected the maps, run the `pagemap_reader` program. You need to provide the PID of the `prober` process and the virtual address you got from its output.

```bash
./pagemap_reader 12345 0x55a6d0f5a2a0
```

Remember to replace `12345` and `0x55a6d0f5a2a0` with the actual values from your `prober`'s output.

You will see an output like this:

```
Virtual Address: 0x55a6d0f5a2a0
Page Frame Number (PFN): 0x1a2b3c
Physical Address: 0x1a2b3c2a0
```

This shows you the Page Frame Number (PFN) and the calculated physical address for the given virtual address.

## Conclusion

This tutorial has shown you the basic steps to probe the virtual memory of a process on Linux. You have learned how to:

*   Use `/proc/[pid]/maps` to view memory mappings.
*   Use `/proc/[pid]/pagemap` to get the Page Frame Number (PFN) for a virtual address.
*   Calculate the physical address from the PFN.

This is just the tip of the iceberg. The `/proc` filesystem contains a vast amount of information about the kernel and processes. I encourage you to explore it further.

