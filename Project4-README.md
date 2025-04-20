## System Specifications:

### Device Specifications:

1. Device name DESKTOP-LQQON4N
2. Processor Intel(R) Core(TM) i5-10200H CPU @ 2.40GHz 2.40 GHz
3. Installed RAM 16.0 GB (15.8 GB usable)
4. Device ID B8BC3905-999D-466B-B7A1-6063FD2E3420
5. Product ID 00327-35930-55360-AAOEM
6. System type 64-bit operating system, x64-based processor
7. Pen and touch No pen or touch input is available for this display

### Windows Specifications:

1. Edition : Windows 10 Home Single Language
2. Version : 22H2
3. Installed on ‎25-‎02-‎2022
4. OS build : 19045.3448
5. Experience : Windows Feature Experience Pack 1000.19044.1000.0

===========================================================================

## Steps to Run the Code:

I have executed the commands in the visual studio code as well as in the ubuntu.

### steps for the ubuntu terminal:

1. open the ubuntu.

2. navigate to xv6-public directory using cd command.
   ![Alt text](Project_1_Screenshots/ubuntu-execution-steps.PNG)
3. run make clean and make qemu-nox commands to invoke the shell.
4. run the commands for uniq and head.
   ![Alt text](Project_1_Screenshots/ubuntu-execution-steps2.PNG)

### steps for vscode:

1. open vscode.
2. open xv6-public folder using open folder option.
3. On the top select "terminal" and open new terminal.
   ![Alt text](Project_1_Screenshots/vscode-1.PNG)
4. select the Ubuntu(WSL) terminal from the options.
5. run make clean and make qemu-nox commands to invoke the shell.
   ![Alt text](Project_1_Screenshots/vscode-2.PNG)
6. run the commands for uniq and head.

==============================================================================

### The text files we are considering:

1. osfile.txt

```
> cat osfile.txt

I understand os
I understand os
I understand os
I UNDERSTAND os
I UNDERSTAND os
I UNDERSTAND os
I love os
I love os
I love os
I love os
I love os
Hello xv6
Hello xv6
Thanks xv6
Bye
```

2. nextfile.txt

```
> cat nextfile.txt

I love music.
I love music.
I love music.
I love music.
I love music.
I love music.
I love music.
I love music of Ganesh.
I love music of Ganesh.
I love music of Ganesh.
I love music of Ganesh.
Thanks.
Bye.
Bye.
Take Care.
Take Care.
```

for each user c program file we are creating, we are including in the EXTRA and UPROGS in the Makefile.

## Virtual Memory Management is xv6

- xv6 operates with 32-bit virtual addresses, resulting in a 4GB virtual address space. The management of memory allocations in xv6 is handled through paging, utilizing a 4KB page size and employing a two-level page table structure.

- The CR3 CPU register maintains a reference to the page table associated with the currently running process. The translation of virtual addresses to physical addresses by the Memory Management Unit (MMU) involves indexing: the initial 10 bits of a 32-bit virtual address index into a page table directory, leading to an inner page table.

- The subsequent 10 bits index into this inner page table to locate the Page Table Entry (PTE), housing a 20-bit physical frame number along with flags.

- Within each process's virtual address space, the kernel's code and data commence from KERNBASE (2GB in the code) and extend up to PHYSTOP, which has a maximum value of 2GB. This range of virtual addresses, [KERNBASE, KERNBASE+PHYSTOP], maps to the physical memory range [0, PHYSTOP].

- The kernel is uniformly mapped into the address space of every process, and it encompasses mappings for all usable physical memory. As a consequence, xv6 is limited to utilizing a maximum of 2GB of physical memory. Sheets 02 and 18 provide comprehensive details regarding the memory layout within xv6.

![Alt text](Project_4_Screenshots/xv6_memory.png)

### Functions

### `PGROUNDUP` and `PGROUNDDOWN`

- **Purpose**: `PGROUNDUP` rounds up an address to the nearest page boundary, performing a ceil-like operation. Conversely, `PGROUNDDOWN` rounds down an address to the previous page boundary.
- **Example**:
  - `PGROUNDUP(620)` will yield `1024`.
  - `PGROUNDDOWN(2400)` will result in `2048`.

### `switchuvm`

- **Purpose**: This function loads a user process's information to execute it by setting the process's Page Table to `%cr3`.
- **Usage**: Primarily employed to switch between the kernel and user space.

### `kalloc`

- **Purpose**: Allocates an unused page (4096 bytes) in RAM and returns the virtual address of the new page.
- **Usage**: Utilized when a free page is required for allocation.

### `walkpgdir`

- **Purpose**: Retrieves the content of the 2nd level Page Table Entry (PTE). Determines the 2nd level Page Table by using the 1st level Page Directory Entry (PDE).
- **Operation**:
  - Checks for PTE presence and allocates second-level page tables if necessary (`alloc = 1`).
  - Maps virtual addresses to physical addresses in the Page Table Entries.

### `mappages`

- **Purpose**: Maps a range of virtual addresses to physical addresses in page-sized chunks. Handles mappings of page table entries.
- **Operation**:
  - Checks for pre-existing mappings and performs remapping if necessary.
  - Establishes mappings by setting the page table entries with relevant permissions.

### `allocuvm`

- **Purpose**: Increases a process's user virtual memory from `oldsz` to `newsz`. Allocates and maps new pages as needed, expanding the heap.
- **Operation**:
  - Allocates new pages for the process.
  - Maps newly allocated pages to the specified virtual address range.

### `deallocuvm`

- **Purpose**: Frees logical and physical pages between old and new sizes of a process. Resets the associated Page Table Entries.
- **Operation**:
  - Frees logical and physical memory pages between the old and new sizes.
  - Clears the corresponding Page Table Entries.

### `copyuvm`

- **Purpose**: Sets up a child process's memory as a complete copy of the parent's memory, ensuring an identical memory image for the child.
- **Operation**:
  - Copies the parent's memory content to the child's memory space.
  - Creates an identical memory layout for the child process.

### `setupkvm`

- **Purpose**: Configures kernel virtual memory for a child process based on the parent's address space.
- **Operation**:
  - Allocates and maps physical pages for the child process.
  - Copies the parent's memory content and sets up entries in the child's Page Table.

## Process Creation:

#### userinit:

- The initial user process, known as `init`, is launched by the `userinit` function within the `main` function in the `main.c` file.

- In the `allocproc` function, the process ID (pid) is assigned, and the process structure is initialized for each process. Specifically, for the `init` process, a pid of 1 is returned during the allocation process.

- During this sequence, `allocproc` invokes `kalloc` to set up the necessary data on the kernel stack. Additionally, the `setupkvm` function is called to establish the kernel page table for the `init` process.

- Subsequently, the `inituvm` function is executed, responsible for allocating a single page of physical memory. It proceeds to copy the `init` executable into this allocated memory and sets up a page table entry (PTE) for the first page within the user virtual address space.

- now the init executable that has been loaded runs and it calls exec.

![Alt text](Project_4_Screenshots/init_code.PNG)

#### exec

- The process begins with the `setupkvm` function, where a separate kernel page table is established for the `exec` function. `allocuvm` follows, allocating a single page for the executable, which suffices for the `init` process. This allocation count, marked as 1, is significant for tracking memory allocations.

- Subsequently, `loaduvm` is employed to load the executable from the disk into the newly allocated page. Moving forward, another call to `allocuvm` extends the new memory image to encompass not only executable code and data but also stack space.

- Rather than allocating a single page for the stack, two pages are allocated, with the second serving as the actual stack and the first functioning as a guard page. This guard page, present but not user-accessible, serves as a protective measure against the stack growing off the stack page.

- The user's memory now comprises strings containing command-line arguments and an array of pointers at the top of the stack. Notably, the `exec` system call doesn't alter the kernel stack; instead, it modifies the user part of the memory image.

- To facilitate the execution of the newly loaded executable, the `exec` code updates the return address in the trap frame to point to the entry address of the binary and sets the stack pointer to the top of the newly created user stack.

- Upon successful execution, `switchuvm` transitions to the new memory image by updating the CR3 register, enabling access to the new memory image in userspace. Subsequently, the `freevm` and `deallocuvm` functions release the old memory image utilized by the process, specifically deallocating the page allocated during `inituvm`.

#### sh and fork():

- In the context of initializing the shell (sh) process, the `fork` system call takes center stage. Upon the execution of the `fork` call within the `init` process, a new process (pid = 2) for the shell is created via the `allocproc` function.

- The process of duplicating the parent's memory for the child process is facilitated by the `copyuvm` function. This ensures that the child process's memory image is an exact replica of its parent's memory. Specifically, within `copyuvm`, a new kernel page table is generated, and memory allocation for the parent process is replicated. Notably, the three pages previously created during the `init` execution via `exec`—which loaded and set up the initial executable—are replicated at this stage.

- This memory duplication operation results in the allocation of three pages for the shell process, establishing its initial memory image.

## Part A: Pointer Dereference

In this we are first dereferencing a null pointer in the xv6 using the code which is nptr.c below:

```
#include "syscall.h"
#include "types.h"
#include "user.h"
int main()
{

    int *my_addr = (int *)0x0000;
    printf(1, "printing the virtual address and value %p: %d\n", my_addr, *my_addr);

    exit();
}
```

In this C code snippet, the program attempts to dereference a null pointer (0x0000) by assigning it to an integer pointer my_addr and then tries to print the value at this address.
The output for the corresponding code is shown in below Fig:

![Alt text](<Project_4_Screenshots/part A/nptr.PNG>)

"trap 6 error" typically refers to a specific kind of error known as an invalid opcode exception.

Similary the dereferencing in the linux is done and the output came as segmentation fault.

![Alt text](<Project_4_Screenshots/part A/Linux_null.PNG>)

### unmapping 3 pages:

The task is to make the first 3 pages unmapped in process virtual address space which means we have to place the code from 4th page.
To acheive this we need to modify some files:

#### 1. exec :

In this the sz is starting from 0 and we should change this to 3\*PGSIZE to load the image from 4th page.

```
// Load program into memory.
  sz = 3*PGSIZE;//p4
// starting mapping from 4th page
```

#### 2. copyuvm:

It exists in vm.c which is used to copy all pages from parent to child which usually called by fork() and the starting page should be changed from i=0 to i= PGSIZE.

```
if((d = setupkvm()) == 0)
    return 0;
  for(i = 3*PGSIZE; i < sz; i += PGSIZE){
    if((pte = walkpgdir(pgdir, (void *) i, 0)) == 0)
      panic("copyuvm: pte should exist");
    if(!(*pte & PTE_P))
      panic("copyuvm: page not present");
    pa = PTE_ADDR(*pte);
    flags = PTE_FLAGS(*pte);
    if((mem = kalloc()) == 0)
      goto bad;
    memmove(mem, (char*)P2V(pa), PGSIZE);
    if(mappages(d, (void*)i, PGSIZE, V2P(mem), flags) < 0) {
      kfree(mem);
      goto bad;
    }
  }
```

#### 3. Makefle:

The execution genally starts from 0 where the instruction pointer points to and now this showuld be changed to 0x3000 as entry point for our program execution.

```
%: %.o $(ULIB)
	$(LD) $(LDFLAGS) -N -e main -Ttext 0x3000 -o $@ $^
	$(OBJDUMP) -S $@ > $*.asm
	$(OBJDUMP) -t $@ | sed '1,/SYMBOL TABLE/d; s/ .* / /; /^$$/d' > $*.sym

_forktest: forktest.o $(ULIB)
	# forktest has less library code linked in - needs to be small
	# in order to be able to max out the proc table.
	$(LD) $(LDFLAGS) -N -e main -Ttext 0x3000 -o _forktest forktest.o ulib.o usys.o
	$(OBJDUMP) -S _forktest > forktest.asm

```

#### 4. syscall.c - arptr:

The condition in the argptr is written as the address is zero then return -1.

```
if(size < 0 || (uint)i >= curproc->sz || (uint)i+size > curproc->sz || i==0)
return -1;
```

Now after making the changes we need to run the nptr.c which contains code to dereference a null pointer. The output for the code is shown in below screenshot.

![Alt text](<Project_4_Screenshots/part A/nptr_v2.PNG>)

A "trap 14" error, specifically "err 4", refers to a page fault exception with an error code 4 on x86 architecture. This exception occurs when the processor detects an attempt to access a page that is not present in physical memory, causing a page fault.

### Running older codes:

#### 1. head

![Alt text](<Project_4_Screenshots/part A/head -1.PNG>)

#### 2. uniq

![Alt text](<Project_4_Screenshots/part A/uniq -1.PNG>)

#### 3. test

![Alt text](<Project_4_Screenshots/part A/test -1.PNG>)

#### 4. ps

![Alt text](<Project_4_Screenshots/part A/ps -1.PNG>)

#### 5. scheduler

![Alt text](<Project_4_Screenshots/part A/fcfs -1.PNG>)

![Alt text](<Project_4_Screenshots/part A/prio - 1.PNG>)

The older codes head, uniq, ps, test, and schedulers are working normally like previous and giving correct output.

## Part B: Stack Rearrangement

In this we have to change the place of the stack and first page should lies between 636KB and 640KB. The stack should grow backwards and should keep at end of address space. To do this we need to modify some files:

#### 1.proc.h

In this we need to add a field in PCB for maintaining the stack pointer.

```
 uint top_stack;
```

#### 2. exec

In this we need to change the allocation code of the stack by allocating the first page at 636-640KB and there should be a gap between heap and stack. There should be one guard page.

```
if ((sz = allocuvm(pgdir, sz, sz + 156 * PGSIZE)) == 0)
    goto bad;

// to create a gap between stack and heap
for(int temp=156;temp>=152;temp--)
{
  clearpteu(pgdir, (char *)(sz - temp * PGSIZE));
}

// for guard page
  clearpteu(pgdir, (char*)(sz - 2*PGSIZE));
  sp = sz;

// cprintf("stack top sp: 0x0000%x\n", sz);
myproc()->top_stack = sz;
```

#### 3. fork()

In this the stack pointer of the parent should be copied to child as we declared that in proc.h.

```
np->sz = curproc->sz;
np->parent = curproc;
*np->tf = *curproc->tf;
np->top_stack=curproc->top_stack;
```

#### 3. trap.c

The stack grow on demand when page fault occurs. Therefore to handle the page fault condition we are adding code in trap.c.

```
case T_PGFLT:
    grow_stack();
    break;
```

when this page fault occurs it calls grow_stack() which is written in vm.c.

#### 4. defs.h

The declaration of the function is grow_stack is written in defs.h.

#### 4. vm.c

The function definition of grow_stack is written in vm.c and this function increases the stack on demand. First it will check whether the page fault is between stack and guard and allocates one page if it is true otherwise it will show invalid access.

```
void grow_stack(void)
{
  int stack = myproc()->top_stack;
  int addr = rcr2();
  int gpage = myproc()->top_stack - 151 * PGSIZE;


  cprintf("gpage ending: 0x0000%x stack: 0x0000%x addr: 0x0000%x\n", gpage, stack, addr);

  if ( addr < stack && addr >= gpage)
  {

    int new_stack_sz = stack - PGSIZE;
    new_stack_sz = allocuvm(myproc()->pgdir, new_stack_sz, new_stack_sz + PGSIZE);
    if (new_stack_sz == 0)
    {
      cprintf("Not a valid access\n");
      myproc()->killed = 1;
    }
    else
    {
      lcr3(V2P(myproc()->pgdir));
      cprintf("Allocating a new stack page\n");
    }
  }
  else
  {
    cprintf("Invalid access\n");
    myproc()->killed = 1;
  }
}

```

Now if we print the dereference program after making the changes it will show the invalid access as the page that is getting faulted is not between the stack and guard.

The output for all the previous commands after updating the stack allocation is given below.

#### 1. head

![Alt text](<Project_4_Screenshots/part B/head -2.PNG>)

#### 2. uniq

![Alt text](<Project_4_Screenshots/part B/uniq -2.PNG>)

#### 3. test

![Alt text](<Project_4_Screenshots/part B/test -2.PNG>)

#### 4. ps

![Alt text](<Project_4_Screenshots/part B/ps -2.PNG>)

#### 5. fcfs

![Alt text](<Project_4_Screenshots/part B/fcfs - 2.PNG>)

#### 6. priority

![Alt text](<Project_4_Screenshots/part B/prio -2.PNG>)

## Resources:

online resources:
Lecture Videos of XV6: https://www.cse.iitb.ac.in/~mythili/os/
(helpful in understanding the xv6 concepts and existing code of xv6. mainly Processes in xv6, Process management system calls in xv6, Trap handling in xv6, Scheduling and context switching in xv6, User process creation in xv6)

Xv6 Operating System -adding a new system call: https://www.geeksforgeeks.org/xv6-operating-system-adding-a-new-system-call/
(helped in understanding about how to implement a system call in xv6 operating system)

Xv6 Operating System -add a user program : https://www.geeksforgeeks.org/xv6-operating-system-add-a-user-program/
(helped in understanding about how to add a user program in xv6 OS)

Uniq Functionality: https://www.redhat.com/sysadmin/uniq-command-lists
Uniq Functionality: https://www.javatpoint.com/linux-uniq
Uniq Lecture : https://youtu.be/k2k1LaKsAWI
(helped to understand the functionality of uniq command in linux)

Head Command : https://www.baeldung.com/linux/head-tail-commands
Head Command: https://linuxhint.com/linux-head-command-with-examples/
Head Lecture : https://youtu.be/ZZACz-filWU
(helped to understand the functionality of head command in linux)

ps command: https://www.geeksforgeeks.org/ps-command-in-linux-with-examples/

Different times: https://www.gatevidyalay.com/turn-around-time-response-time-waiting-time/

FCFS and Priority: https://www.geeksforgeeks.org/difference-between-fcfs-and-priority-cpu-scheduling/

Memory Management : https://github.com/zarif98sjs/xv6-memory-management-walkthrough/tree/main

Professor Office hours: Asked doubt regarding how to check whether the implementation is correct or not. Told that will check the changes no need to write the testing code.
