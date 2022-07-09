# unix_linux_practice

## **UNIX**

#### _History_

Unix was the name of an OS that was original created in the early 1970s. Today however Unix refers to _any_ OS that immitates that same OS.

#### _POSIX && SUS_

POSIX -- Portable Operating System Interface for Unix
SUS -- Single Unix Specification

The first stands that defined what it meant to be a Unix system. In the Late 80s and early 90s two such standards were created, POSIX and SUS. Today most varients of Unix more or less conform to both of these standards. Mainly these standards _fail_ to solve the problem of protablity because they _do not_ specifiy everything about a "Unix System." They are rather limited in their scope, so there are many features of today's Unix systems which work totaly different system - system or in some cases work only on some systems and not others. Ultimately programs should run on _any_ Unix system so long as they remain with the bounds of the two standards POSIX && SUS.

#### _Terminal Character Device File_

In unix system a process might communicate with a terminal via a special file representing that terminal -- _**Terminal Character Device File**_


When a prossess write to a _**Terminal Character Device File**_, that's putting data in the _output buffer_ of that device file, which will then be sent out by the OS to the terminal device assoicated with that device file.

When a user types, that data gets sent from the terminal to the computer and then the OS will take that data and put it in the _input buffer_ of the associated _**Terminal Character Device File**_ and then the proccess may read from the _**Device File**_ to get that data.

#### _Echoing Mode_

With a _**Terminal Character Device File**_ we can turn on a mode called _Echoing_. When a terminal character device operates in echoing mode, then any input it recieves from the terminal it will then immediately echo back out to the terminal so it will appear on screen. In practice when echoing is on, whatever key is typed by the user the key will immediately appear on screen. ( Note :: the terminal _does not_ have an echoing mode, the _**Terminal Character Device File**_ has an echoing mode, so the data is actually being sent from the terminal and then immdiately back to the terminal )

#### _Escape Squences_

A sequence of characters beging with the ascii escape charater (ascii code 27 ) 

_**TODO**_ :: how to change text color via escape squences

#### _File Descriptor 0 && 1_

In unix we have this convention where by a proccess, when they are started expect to **inherit** from their parent two open file descriptor -- 0 && 1.

File Descriptor 0 -- **Standard In** (_abv. stdin_)
File Descriptor 1 -- **Standard Out** (_abv. stdout_)

By Convention, Proccesses expect _**File Descriptor 0**_ (_stdin_) to be a file descriptor open for **reading** a terminal character device file, _**File Descriptor 1**_ (_stdout_) for **writing** that same terminal character device file. In practice this means when a proccess wishes to read text from a terminal it reads from stdin, and when a proccess wish to display text to that same terminal it writes to its stdout.

#### _Forking_

When a Process forks in unix, the file descriptors get copied form the parent to child.

#### _System Calls_ 

Functions in the OS code that programs can invoke with a special instuction. These system calls are the primary means by which the OS exposes functionality to programs so that these programs can use features of the hardware. Ex: read and write data on a storage device or send and recieve data accross the network

Reason system calls can only be ivoked with a special instuction is that normally when a proccess executes, it cannot read / execute data thats part of the OS's kernal itself. Each proccess is to run essentially confined to its own "box" (its own part of memory), thus the special instuction will break out of this "box" and the way it does this is in the instuction one must specifiy the number of the system call you wish to invoke and that causes the CPU to lookup an address that corresponds to that number in a special table.

Note: In most OS, the kernal code for these system calls are placed the address space of each process

-----------------------------------------------
|   _________________________________________  |
|  |                                         | |
|  |     Kerernal Code                       | |  <-------- only accessible in system calls
|  |_________________________________________| |
|   _________________________________________  |
|  |                                         | |
|  |     Stack                               | |
|  |_________________________________________| |
|                                              |
|                                              |
|                                              |
|   _________________________________________  |
|  |                                         | |
|  |     HEAP                                | |
|  |_________________________________________| |
|   _________________________________________  |
|  |                                         | |
|  |                                         | |
|  |                                         | |
|  |     HEAP                                | |
|  |                                         | |
|  |                                         | |
|  |_________________________________________| |
|                                              |
|                                              |
|                                              |
|   _________________________________________  |
|  |     HEAP                                | |
|  |_________________________________________| |
|   _________________________________________  |
|  |                                         | |
|  |     Code                                | |  ^^^ JUMPS to Kernal Code via special instuction ^^^
|  |_________________________________________| |
-----------------------------------------------

Note: When a system call is invoked, it uses the stack of the proccess to place a stackframe for that system call like any other function, to execute in that context of the proccess of which invokes them. Which avoids a context switch, ie the memory tables dont have to be switched out from the current proccess. Thus the table can be left in memory for the duration of the proccess.

-----------------------------------------------
|                   STACK                      |
|                                              |
|   _________________________________________  |
|  |                                         | |
|  |     Frame of Syscall                    | |
|  |_________________________________________| |
|   _________________________________________  |
|  |                                         | |
|  |     Frame of Baz                        | |
|  |_________________________________________| |
|   _________________________________________  |
|  |                                         | |
|  |     Frame of Bar                        | |
|  |_________________________________________| |
|   _________________________________________  |
|  |                                         | |
|  |     Frame of Foo                        | |
|  |_________________________________________| |
|   _________________________________________  |
|  |                                         | |
|  |     Frame of Main                       | |
|  |_________________________________________| |
-----------------------------------------------

Another advantage is that this arrangment is that it allows for system calls to be interrupted arbitrarily. That is suspended and resumed later. Suspending a proccess typically the same if you are suspending user code, ie the code to the program itself or if it's running kernal code, ie a syscall.


#### _Proccess_

How a process transitions between a few different states

 _________________________________________             _________________________________________ 
|                                         |           |                                         |
|       NEW Proccess Created              |           |         Proccess TERMINATED             |
|_________________________________________|           |_________________________________________|
                    |                                                      ^
                    |                                                      |
                    |                                                      |
                    V                                                      |
  _______________________________________                 _____________________________________ 
 |                                       |               |                                     |
 |                Waiting                |  <--------->  |               Running               |
 |_______________________________________|               |_____________________________________|
                       ^                                                   |
                        \                                                  |
                         \                                                 V
                          \                               _____________________________________ 
                           \                             |                                     |
                            \__________________________  |              BLOCKED                |
                                                         |_____________________________________|