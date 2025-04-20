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

The implementation of system calls is done here. The ptable can be access in proc.c directly.

### Retreiving arguments and file contents in kernel mode:

To retreive the arguments in the kernel mode which are passing from user space we are using argint() and argstr() for integer and string values respectively.
eg: argint(0,&fd)- will read the first argument and store it in 'fd' variable.

# Implement various system call focused on process in xv6 - Updates

## Part 1 - Getting process statistics and modifying system call in xv6

To display the statistics we created and modified the following files.

- proc.h
- test.c
- proc.c

### 1. proc.h

To display the start time, end time and total time of the process we have extended the PCB by adding the following fields.

```
  uint start_time;
  uint end_time;
  uint total_time;
  struct rtcdate b_timestamp;
  struct rtcdate e_timestamp;
```

The first 3 fields are used to display the time in ticks format and b_timestamp and e_timestamp are used to display the time in UTC time format with Date and Time. To acheive this we are using "date.h" where the struct rtcdate is already declared and we are using the same declaration in proc.h.

### 2. test.c

when a test command is applied followed by some process then we need to print the statistics for the process.

In test.c we are creating child process for the test using fork() to execute the process which we want to print the statistics and parent test process will wait until the child completes. For example the below command we need statistics for head process.

```
test head filename
```

Inside the test.c we are using exec() system call to execute the command that is followed by the test command. here it is "head filename".

exec() will remove the memory image of child test process and replace it with the memory image of "head filename".

### 3. proc.h

#### a. Inside allocproc()

The variables that we declared in the PCB need to be initialized inorder to print. Therefore, we are initializing the start_time of the process in allocproc() where the allocation of the process is done.
As we are displaying additional detail timestamp, we are initializing the b_timestamp also in this allocproc().

```
//initializing the start_time in ticks
p->start_time=ticks;
//Assigning the current timestamp to creation time.
cmostime(&(p->b_timestamp));
```

To initialize the process starttime we are using ticks. "ticks" typically refer to timer interrupts or clock ticks. These are units of time measured by the system timer interrupt. In xv6, the timer interrupt occurs at a constant rate, and the timer keeps track of the number of timer interrupts (or ticks) that have occurred.

To initialize the process timestamp we are using inbuilt function cmostime. cmostime function is used to retrieve the current date and time from the real-time clock (RTC) of the underlying hardware. The RTC is typically part of the computer's hardware and keeps track of the date and time even when the computer is powered off.

#### b. Inside exit()

The exit() system call is used by a process to terminate itself. When a process calls exit(), it effectively signals to the operating system that it has completed its execution and is ready to be cleaned up.

Therefore, we are initializing the end_time and end timestamp of the process in exit() system call. As we are getting endtime of the process it is easy to calculate the total_time executed by the process.

```
//initializing the end_time and tital_time in ticks
curproc->end_time = ticks;
curproc->total_time = curproc->end_time - curproc->start_time;

//assigning the timestamp to e_timestamp.
cmostime(&(curproc->e_timestamp));

// function to print all stats
statdetails();
```

#### c. Inside statdetails()

As soon as the process is exitiing we are printing the statistics of the process inside the statdetails() by calling the function inside exit() when the process is getting terminated.

In statdetails() we are printing 3 variations of time.

- Ticks
- xv6 Timestamp (the time will start when xv6 bootsup)
- UTC timezone timestamp

#### i. Ticks

To print the ticks we are using the fields that we declared in PCB: start_time, end_time and total_time which we initialized in ticks.

```
    cprintf("\n=======================================");
    cprintf("\nTime in Ticks\n");
    cprintf("=======================================\n");
    cprintf("Parameter               ticks\n");
    cprintf("----------------------  ---------------\n");
    cprintf("Start Time              %d\n", curproc->start_time);
    cprintf("End Time                %d\n", curproc->end_time);
    cprintf("Total Time              %d\n", curproc->total_time);
    cprintf("=======================================\n");
```

#### ii. xv6 Timestamp

As soon as the xv6 boots up the ticks will start ticking. To understand better we are displaying the time in HH:MM:SS.SSS format which is human readable. Here, the milliseconds will tell the total_time of the process better compared to ticks.

To do that we are converting the ticks to hrs, min, secs and millisecs by sending ticks to print_HHMMSS()

```
void print_HHMMSS(uint ticks)
{

    uint seconds = ticks / 100; // Convert to seconds (assuming 100 timer ticks per second)
    uint hours = seconds / 3600;
    uint minutes = (seconds % 3600) / 60;
    uint remainingSeconds = seconds % 60;
    uint milliseconds = (ticks % 100) * 10;
    cprintf(" %d:%d:%d.%d\n", hours, minutes, remainingSeconds,milliseconds);
}
```

#### iii. UTC timestamp

This is the time format of real time which we are displaying using cmostime.
The fields that we declared in PCB: b_timestamp and e_timestamp are used to display the timestamp in UTC timezome.

As the rtcdate structure contains year, month, day, hour, minute and second, we are able to print those inside the statdetails() along with ticks and xv6 timestamp formats.

```
    cprintf("\n==========================================================");
    cprintf("\nTime based on system timestamp (UTC timezone)\n");
    cprintf("==========================================================\n");
    cprintf("Start Date and Time:  %d-%d-%d %d:%d:%d\n",
    st.year, st.month, st.day, st.hour, st.minute, st.second);
    cprintf("End Date and Time:    %d-%d-%d %d:%d:%d\n",
    et.year, et.month, et.day, et.hour, et.minute, et.second);
    cprintf("==========================================================\n");
```

Observed that start time and end time showing same timestamp as the process executing in milliseconds.

## Part 2 - Implementing ps on xv6

The ps command that we are going to implement will work in 3 variations.

```
>ps
>ps pid
>ps process_name
```

To display the ps table we created and modified the following files.

- ps.c
- proc.c
- The files that are mentioned in "For implementing the system calls we need to modify some files:" section to implement system call.

### 1. ps.c

In ps.c we are parsing the command to know whether the given command has pid as argument or process name as argument or no argumnets.

if the value of argc=1 then we need to display all process details in ptable.

if there are 2 arguments then we are checking whether the given argument is integer(pid) or string(process name) and passing the respective arguments to system call psdetails() which is implemented in proc.c.

we are using the flg variable to know whether given argument is integer or string. if flg=1 then it is string and if flg=2 then it is integer.

### 2. proc.c

In this we are implementing sys_psdetails() system call and retreiving arguments that are passed in ps.c.

The code snippet implemented iterates through the process table in the xv6 operating system, collects information about running processes, and prints a formatted table of process details. It includes the process ID (PID), process name, start time, elapsed time, process state, and the name of the parent process for each active process.
if there are no processes with the given details it will display there are no process.

```
for(p = ptable.proc; p < &ptable.proc[NPROC]; p++)
    {
        if(p->state!=UNUSED)
        {

          cprintf("|%d\t|%s\t\t|%d\t\t|%d\t\t|%s\t\t|%s\t\t\n",
          p->pid, p->name,p->start_time,ticks-p->start_time,states[p->state],p->parent->name);
        }

    }
```

====================================================================================

### Commands:

#### head system call stat commands

```
test head filename
test head file1 file2
test head -n N filename
test head -n N file1 file2
cat filename | test head
```

#### uniq system call stat commands

```
test uniq filename
test uniq -c filename
test uniq -d filename
test uniq -i filename
cat filename | test uniq
test cat filename | test uniq ( prints stats for both cat and uniq)
```

#### ps commands

```
ps
ps init
ps sh
ps <process name>
ps 1
ps 2
ps N
```

#### head and uniq comabination with stats

we can display stats together for both uniq and head using pipe operator in between.

```
test head -n N filename | test uniq filename
test head filename | test uniq -c filename
```

#### ps system call with test.

As mentioned for new system calls also our test.c should work.

```
test ps
test ps <pid>
test ps <pname>
```

### Sample Outputs

#### 1. test head filename

![Alt text](<Project_2_Screenshots/test_head/test head filename.PNG>)

#### 2. test head -n N filename

![Alt text](<Project_2_Screenshots/test_head/test head -n N filename.PNG>)

#### 3. test head -n N file1 file2

![Alt text](<Project_2_Screenshots/test_head/test head -n N file1 file2 and so on.PNG>)

#### 4. cat filename | test head

![Alt text](<Project_2_Screenshots/test_head/cat filename  test head.PNG>)

#### 5. test uniq filename

![Alt text](<Project_2_Screenshots/test_uniq/test uniq filename.PNG>)

#### 6. test uniq -c filename

![Alt text](<Project_2_Screenshots/test_uniq/test uniq -c filename.PNG>)

#### 7. test uniq -i filename

![Alt text](<Project_2_Screenshots/test_uniq/test uniq -i filename.PNG>)

#### 8. test uniq -d filename

![Alt text](<Project_2_Screenshots/test_uniq/test uniq -d filename.PNG>)

#### 9. cat filename | test uniq

![Alt text](<Project_2_Screenshots/test_uniq/cat filename test uniq.PNG>)

#### 10. test head -n N filename | ps

![Alt text](<Project_2_Screenshots/test_combination/test head ps.PNG>)

#### 11. test head -n N filename | test uniq [flag] filename

![Alt text](<Project_2_Screenshots/test_combination/test head uniq -1.PNG>)
![Alt text](<Project_2_Screenshots/test_combination/test head uniq -2.PNG>)

#### 12. test uniq [flag] filename| ps

![Alt text](<Project_2_Screenshots/test_combination/test uniq ps.PNG>)

#### 13. ps [pid|process name]

![Alt text](Project_2_Screenshots/ps/ps.PNG)

![Alt text](<Project_2_Screenshots/ps/ps process name.PNG>)

![Alt text](<Project_2_Screenshots/ps/ps pid -2.PNG>)

#### 14. test ps [pid|process name]

![Alt text](<Project_2_Screenshots/ps/test ps.PNG>)

![Alt text](<Project_2_Screenshots/ps/test ps pid.PNG>)

![Alt text](<Project_2_Screenshots/ps/test ps pname.PNG>)

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

Teaching Assistant: Clarified doubt regarding the implementation.

Professor Online hours: Asked doubt regarding how the command should be and expected output.
Told that we can implement time variations / timestamp.
