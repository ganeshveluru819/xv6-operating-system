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

### For implementing the system calls we need to modify some files:

#### 1. syscall.h

By default there will be 21 system calls and we can add additional system calls here.

#### 2. syscall.c

Here we are adding function pointer to the system call and declaring the function prototype with extern scope as we are not defining here.

#### 3. uys.S

it will acts as a interface for the system calls and we need to add SYSCALL(name) here, additional to existing system calls.

#### 4. user.h

Here we are declaring the function with arguments and return type as we need to pass the arguments to system call regarding file information and options etc.

#### 5. proc.c

The implementation of system calls is done here. The ptable can be accessed in proc.c directly.

### Retreiving arguments and file contents in kernel mode:

To retreive the arguments in the kernel mode which are passing from user space we are using argint() and argstr() for integer and string values respectively.
eg: argint(0,&fd)- will read the first argument and store it in 'fd' variable.

# Scheduler Implementation

## proc.h

In this we have added two more variables inside the process structure(PCB). cpu_start is used to store the time when the process is first assigned to cpu and prio is used in case of priority scheduling.

```
unit cpu_start;
unit prio;
```

## proc.c

### Global variables

Global variables to keep track of stats, times, scheduler, num of processes are declared in the beginning of proc.c.

```
// Global variables to store times, scheduler
int which_scheduler=0;
int avg_wait_time=0;
int avg_turnaround_time=0;
int num_process=0;
int priority_arrival=0;

//Global arrays to store all process details and times to print at the end
int st[ar_size];
int ct[ar_size];
int at[ar_size];
int bt[ar_size];
int pid_list[ar_size];
int turn[ar_size];
int wait_time[ar_size];
int prior_list[ar_size];
char process_list[20][20];
```

### sys_set_scheduler() & sys_reset_scheduler()

Inside the sys_set_scheduler() we are setting the algorithm to either FCFS or Priority based on the argument provided to the system call.

Inside the sys_reset_scheduler() we are printing the all time details like Arrival time, Completion time, Burst time, Turnaround time, Waiting Time, Response time and Priorities and resetting the global variables.

### sys_assign_prior()

This system call is particularly used in assigning the processes a priority value in case of priority scheduling based on the argument sent.

# FCFS

The user program for FCFS scheduler is written in FCFS.c file.

The input format of the command is as follows

```
FCFS cmd1 # cmd2 # cmd3 ...

eg: FCFS head osfile.txt # uniq osfile.txt # userhead osfile.txt # useruniq osfile.txt
```

In the user file we first setting the scheduler to "FCFS" using the system call sys_set_scheduler() mentioned above and setting all variables to 0.

Next based on the command-line arguments, it forks child processes to execute commands separated by the "#" symbol, and then waits for all child processes to complete before resetting a scheduler. It iterates through the command-line arguments, identifying commands separated by "#" as child processes, and forking and executing each command in a child process. If the execution of a command fails, it prints an error message. Once all child processes have executed, the parent process waits for them to complete, and then a scheduler is reset.

### FCFS scheduler in proc.c

Inside the scheduler function based on the scheduler we are writing condtions to execute FCFS scheduler.

if the scheduler ==1 : for FCFS then we are iterating in the process table to select a process which has least arrival time that is the first process that enters the system and we are assigning cpu to that system by initializing the cpu_start variable to ticks.

```
 // ignoring init and sh processes from FCFS
        if(p->pid > 1)
        {
          if (fcf_proc != 0)
          {
            // here we need  find the process with the lowest arrival time.
            if(p->start_time < fcf_proc->start_time)
              fcf_proc = p;
          }
          else
              fcf_proc = p;
        }
```

# Priority

The user program for Priority scheduler is written in PRIORITY.c file.

There are two variations implemented for Priority Scheduling.

1. user provided command based on priority (highest priority process placed first followed by lowest process.)
2. user explicitly mentioned priority values for different processes.

The input format for both the versions are as follows:

```
version 1:

PRIORITY cmd1 # cmd2 # cmd3 ...

eg:  PRIORITY head osfile.txt # uniq osfile.txt # userhead osfile.txt # useruniq osfile.txt

```

In this head and uniq has higher priority than userhead and useruniq process.

```
version 2:

PRIORITY -p cmd1 pval # cmd2 pval # cmd3 pval.....

eg: PRIORITY -p userhead -n 3 osfile.txt 1 # head -n 2 osfile.txt 2 # uniq osfile.txt 3 # ps 7

```

In this we are given a flag '-p' to tell the os that user is giving explicit priority values and priority values is given for each command. In the above example the priority for uniq is 3 whereas for ps it is 7.

Inside the PRIORITY.c based on the version we have 2 conditions.
if it is 2nd version then we are placing all commands in an array and based on the priority values mentioned by the user we are assigning priority values using assign_prior() system call to the process after creating a child to the process PRIORITY. The child itself will execute the command using exec().

For the First version, we are splitting the commands based on delimiter '#' and declared a variable to assign the priorities. The first command is given priority '1' as user not mentioned priority explicitly and for the sake of the scheduler and increasing the value as we are moving towards the end of the commands. Here also for each command we are forking a child to execute the command.

### Priority scheduler in proc.c

Inside the scheduler function based on the scheduler we are writing condtions to execute Priority scheduler.

if the scheduler ==2 : for Priority then we are iterating in the process table to select a process which has priority value less that is the process that has highest priority in the system and we are assigning cpu to that system by initializing the cpu_start variable to ticks.

```
// ignore init and sh processes from Priority
        if(p->pid > 1)
        {
          if (prio_proc != 0)
          {
            // here I find the process with the highest priority(less value)
            if(p->prio < prio_proc->prio)
              prio_proc = p;
          }
          else
              prio_proc = p;
        }
```

### Calculating times and Storing.

For the Priority scheduler we are assigning the arrival time of child processes with parent arrival time as we are assuming that all processes present in the initial stage whereas not with the FCFS(executing based on arrival times). we are searching for the parent process when scheduling algorithm is priority and storing it's value and assigning to all child process.

```
  if(which_scheduler==2 && priority_arrival==0)
  {
    for(p = ptable.proc; p < &ptable.proc[NPROC]; p++)
    {
      if(strcmp_p(p->name,"PRIORITY")==0)
        priority_arrival=p->start_time;

    }
  }

  if(which_scheduler==2)
  {
    curproc->start_time=priority_arrival;

  }

```

Different times:

- ArrivalT: Arrival time is the point of time at which a process enters the ready queue.
- TAT: Turn Around time is the total amount of time spent by a process in the system.
  When present in the system, a process is either waiting in the ready queue for getting the CPU or it is executing on the CPU.

```
Turn Around time = Completion time – Arrival time
```

- BT : Burst time is the amount of time required by a process for executing on CPU.

- CT : Completion time is the point of time at which a process completes its execution on the CPU and takes exit from the system.

- WT : Waiting time is the amount of time spent by a process waiting in the ready queue for getting the CPU.

```
Waiting time = Turn Around time – Burst time
```

- RT : Response time is the amount of time after which a process gets the CPU for the first time after entering the ready queue.

```
Response Time = Time at which process first gets the CPU – Arrival time
```

These all times are calculating in the exit() in proc.c and storing required details in global array to print after executing all the processes in sys_reset_scheduler() system call.

```
int tat=curproc->end_time - curproc->start_time;
int burst_time=curproc->end_time - curproc->cpu_start;
int wt=tat-burst_time;

// stroing all details in global array
st[num_process]=curproc->cpu_start;
ct[num_process]=curproc->end_time;
at[num_process]=curproc->start_time;
bt[num_process]=burst_time;
prior_list[num_process]=curproc->prio;
pid_list[num_process]=curproc->pid;
turn[num_process]=tat;
wait_time[num_process]=wt;


//incrementing turnaround time and waiting time with new process's
avg_turnaround_time+=tat;
avg_wait_time+=wt;
num_process+=1;

```

Finally the avg_turnaround_time and avg_wait_time is divided by num_process to get the actual average.

## Outputs:

### FCFS

```
1. FCFS head -n 3 osfile.txt # uniq -c osfile.txt -d osfile.txt # userhead -n 2 osfile.txt
```

![Alt text](Project_3_Screenshots/FCFS/fcfs-kernel-user.PNG)

```
2. FCFS useruniq -d osfile.txt # userhead -n 2 osfile.txt # head -n 3 osfile.txt # uniq -c osfile.txt
```

![Alt text](Project_3_Screenshots/FCFS/fcfs-user-kernel.PNG)

### Priority

#### version-1

![Alt text](Project_3_Screenshots/PRIORITY/p-kernel-user.PNG)

![Alt text](Project_3_Screenshots/PRIORITY/p-user-kernel.PNG)

#### version-2

![Alt text](<Project_3_Screenshots/PRIORITY/option kernel - user.PNG>)

![Alt text](<Project_3_Screenshots/PRIORITY/option kernel user.PNG>)

![Alt text](<Project_3_Screenshots/PRIORITY/option mixed.PNG>)

some of the commands:

```
1. PRIORITY -p head osfile.txt 4 # uniq osfile.txt 7 # userhead -n 3 osfile.txt 5 # ps 8
2. PRIORITY head -n 2 osfile.txt # uniq osfile.txt # useruniq osfile.txt # userhead -n 3 osfile.txt
3. FCFS useruniq -d osfile.txt # userhead -n 2 osfile.txt # head -n 3 osfile.txt # uniq -c osfile.txt
4. PRIORITY -p userhead -n 3 osfile.txt 1 # head -n 2 osfile.txt 2

```

======================================================================================

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

Professor Office hours: Asked doubt regarding how the command should be and expected output. Told Priority can be implemented in either ways- user giving explicit priority or priority based on order.
