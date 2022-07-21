# unix_linux_practice

## **UNIX**

### _History_

Unix was the name of an OS that was original created in the early 1970s. Today however Unix refers to _any_ OS that immitates that same OS.

### _POSIX && SUS_

POSIX -- Portable Operating System Interface for Unix
SUS -- Single Unix Specification

The first stands that defined what it meant to be a Unix system. In the Late 80s and early 90s two such standards were created, POSIX and SUS. Today most varients of Unix more or less conform to both of these standards. Mainly these standards _fail_ to solve the problem of protablity because they _do not_ specifiy everything about a "Unix System." They are rather limited in their scope, so there are many features of today's Unix systems which work totaly different system - system or in some cases work only on some systems and not others. Ultimately programs should run on _any_ Unix system so long as they remain with the bounds of the two standards POSIX && SUS.

### _Terminal Character Device File_

In unix system a process might communicate with a terminal via a special file representing that terminal -- _**Terminal Character Device File**_


When a prossess write to a _**Terminal Character Device File**_, that's putting data in the _output buffer_ of that device file, which will then be sent out by the OS to the terminal device assoicated with that device file.

When a user types, that data gets sent from the terminal to the computer and then the OS will take that data and put it in the _input buffer_ of the associated _**Terminal Character Device File**_ and then the process may read from the _**Device File**_ to get that data.

### _Echoing Mode_

With a _**Terminal Character Device File**_ we can turn on a mode called _Echoing_. When a terminal character device operates in echoing mode, then any input it recieves from the terminal it will then immediately echo back out to the terminal so it will appear on screen. In practice when echoing is on, whatever key is typed by the user the key will immediately appear on screen. ( Note :: the terminal _does not_ have an echoing mode, the _**Terminal Character Device File**_ has an echoing mode, so the data is actually being sent from the terminal and then immdiately back to the terminal )

### _Escape Squences_

A sequence of characters beging with the ascii escape charater (ascii code 27 ) 

_**TODO**_ :: how to change text color via escape squences

#### _File Descriptor 0 && 1_

In unix we have this convention where by a process, when they are started expect to **inherit** from their parent two open file descriptor -- 0 && 1.

File Descriptor 0 -- **Standard In** (_abv. stdin_)
File Descriptor 1 -- **Standard Out** (_abv. stdout_)

By Convention, processes expect _**File Descriptor 0**_ (_stdin_) to be a file descriptor open for **reading** a terminal character device file, _**File Descriptor 1**_ (_stdout_) for **writing** that same terminal character device file. In practice this means when a process wishes to read text from a terminal it reads from stdin, and when a process wish to display text to that same terminal it writes to its stdout.


### _System Calls_

System Calls effectively represent the functionality of the OS exposed to programs.

Functions in the OS code that programs can invoke with a special instuction. These system calls are the primary means by which the OS exposes functionality to programs so that these programs can use features of the hardware. Ex: read and write data on a storage device or send and recieve data accross the network

Reason system calls can only be ivoked with a special instuction is that normally when a process executes, it cannot read / execute data thats part of the OS's kernal itself. Each process is to run essentially confined to its own "box" (its own part of memory), thus the special instuction will break out of this "box" and the way it does this is in the instuction one must specifiy the number of the system call you wish to invoke and that causes the CPU to lookup an address that corresponds to that number in a special table.

Note: In most OS, the kernal code for these system calls are placed the address space of each process

```
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
```

Note: When a system call is invoked, it uses the stack of the process to place a stackframe for that system call like any other function, to execute in that context of the process of which invokes them. Which avoids a context switch, ie the memory tables dont have to be switched out from the current process. Thus the table can be left in memory for the duration of the process.

```
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
```

Another advantage is that this arrangment is that it allows for system calls to be interrupted arbitrarily. That is suspended and resumed later. Suspending a process typically the same if you are suspending user code, ie the code to the program itself or if it's running kernal code, ie a syscall.

This includes syscalls for managing process them. Ex one process, starting other process so it can run another program. Then There are many syscall for dealing with...

• Files, creating - deleting files, reading - writing files etc.

• Sockets - Unix also has system calls for what it calls "sockets". A socket represents one end of a network connection. So when you want a program to talk to another program in another system, you create a socket in your program and that socket communicates with the socket in the other program, creating the channel of communcation.

• Signals - a "signal" is basically a notification of some event or some condition sent from the OS or process to some other process. Many of the signals sent by the OS indicate some kind of error condition. Ex, when your program causes some kind of memory violation. That is it reads / writes some bytes of memory it doesnt have the permissions to, that will trigger a hardware exception in the CPU, causing the CPU to invoke some predetermined piece of OS code, and that OS code will send a singal to our process indicating the error. And when our process recieves that error it gets interrupted and will invoke a function predetermined to run for when it recieves that signal.

• Inter-process Communication - Basically a kind of mechanism for process to communicate with eachother. Sockets are a type of inter-process communication. IE when a process communicates over a socket, the other program does not have to be on another system, it can actually be another process on the same system. However for process to communicate over the same system there are other mechanisms which have the general advantage of being more efficient

• Threads - When a process runs, by default it has one single "thread" of execution. That is there is one code pointer pointing to what current instuction is and the re is one single stack keep track of the functions being invoked. With multiple threads of execution you can effectively split up a process into separate threads, each thread having its own code pointer and its own stack. A different way to think about threads is that they are like separate processies which run independently and are scheduled independently but they share the same address space. So the data on the heap can be read and written by any thread in the process.

TODO :: TODO :: TODO ::
Multi Threads
TODO :: TODO :: TODO :: 

• I/O devices - in / out deivices (think stdin && stdout). _**Unix alows us to treat I/O devices like files.**_ What it really comes down to is that when it comes to reading / writing data from an I/O device, we can in most cases use the same syscalls that we use to read / write files. So in this sense we can treat I/O devices like files.

The foremost system calls for dealing with files are the open, close, read && write syscalls.

##### _open_
    • When reading and writing to file first that file has to be open, so you invoke to open syscall passing in the file path to that file.
    • open / create a file and return a file descriptor

# File Descriptor
Note :: basically a number which in your process uniquely identifies an open file.

##### _close_
    • release file descriptor
    • invoking read does take a file path as an argument, it takes a file descriptor 

Note :: releasing file descriptors is not strictly critical but good practice.
 
##### _read_
    • copy bytes from file to memory ( this call also _may blocks_ )
    • Under which circumstances these syscalls may block depends on the type of file being read and options when that file was open
    • Default read _blocks_
    • invoking read does take a file path as an argument, it takes a file descriptor

##### _write_
    • copy bytes of memory to a file ( this call also may blocks_ )
    • Under which circumstances these syscalls may block depends on the type of file being write and options when that file was open
    • Default write _does not block_
    • The call to write will copy data from the process to a buffer in memory outside the process controlled by the OS, from there the OS will write the data from the buffer to the actual storage device
    • invoking read does take a file path as an argument, it takes a file descriptor

```
f = open('alice/tim')
write(f, 'blah blah')
write(f, 'bling bling bling blah')
close(f)
```
    
##### mmap - ( 'memory map' pages to the process address space )
    • To allocate memory in a process, the process should invoke the mmap syscall.

##### munmap - ( 'memory unmap' pages from the process address space )
    • Deallocates memory pages from the process address space

```
address = mmap(5000)
# allocated memory

munmap(address)
# deallocated memory
```

Note: Notice we do not specifiy which bytes of memory we want. Generally its left up to the OS to keep track of all the 'chunks' in the address space. So it's left up to the OS to go find a chunk of memory (in the above case) of _**at least**_ 5000 continuous bytes and return to us the address of that first byte.

### _Forking_

The way we create a new process in Unix is for a process to copy itself. When a Process forks in unix, the data associated with that original process get copied form the parent to the child. That is to say it copies its _entire_ address space, user ids, enviroment, file descriptors, etc. This is done with the _**fork**_  syscall.

```

if fork() == 0:
    # new child process
else:
    # original (parent) process

```

When the fork syscall returns, _**both**_ processies pick up precisely at that moment where the fork returned. The only difference between the two process is what value gets returned from invoking fork. In the newly created process, the fork, in the child process, returns 0. In the original process, the parent process, the process that invoked fork in the first place, the fork syscall returns what called the process ID (unique number identifying each process in the system) of that child process.

Note: a fork syscall may seem expensive to perform because it includes copying the address space of one process over to a child process, but this is not actually the case. In older Unix's that is actually what happened, but nowadays we have virtual memory. So what happens is that we only need to copy the _memory table_ for the process, not all the actual content. Where the content maybe mgs if not gbs in size, the memory tables themselves are actually quite small, kbs in size. So long as both process only read from their memory rather than write to it, because the memory hasn't change, might as well share the same copy. We want both processies to diverge, such that when in the new fork, when we write to memory that change should only be seen in the new process not the parent process or vice versa. To solve this problem all of the pages on the new process are marked as _**"Copy on Write"**_. As soon either process attemps to write to an address in that page, that triggers an exception in the CPU because it detects it in the memory table that this page has been marked as copy on write. And that then triggers the OS to before allowing the write to occur, to actually copy that page and then update the table of the new fork. Now that the fork has its own copy of that page its okay for writes to go through. This copy on write scheme allows modern unix's to very cheaply fork new process.

### _Exec System Call_

The Exec System Call does not create a new process, that is what fork does. What the exec syscall does is replace the current program in the process with a _new program_. And when it does this essenstially the entire address space of the process gets discarded and effectively a new one is created when loading in the new code from some executable file.

```

if fork() == 0:
    # new child process
    exec('games/pong')
else:
    # original (parent) process

```

When in Unix you wish to start a new program ^^. After the exec most everything about that process remains the same except from the very big difference that its a totally new address space with a new program. The process however retains most of the other resources of that processs such as the enviorment and the file descriptors.

### __exit System Call_

When a process wishes to terminate itself it is done so by invoking _**_exit**_ system call.

```
_exit(0)
```

Note :: the reason for the underscore infront is to distinguish this system call from another function in the c standard libery exit

When we invoke the _exit() syscall we pass in a numer called the "exit code". The exit-code indicated to other programs what happed, why did this program exit. And by _convention_ the exit-code 0 is used to denote that the process terminated normally. Other numbers are used to indicated some kind of error, something went wrong in the program. Again by _convention_ programs should use specific exit-codes to indicate a specific kind of error. ie, exit-code -1 means the program exited because it ran out of memory. Again there are no hard and fast rules about what exit-codes mean, its really up to the indicidual program to decide what they mean. 

When a process terminates the process that can read its exit-code is the parent, the process from which it was forked. And the most common way the parent does this is to invoke the wait() system call

### _wait System Call_

When a process invokes the _**wait()**_ syscall, it goes into a blocked state until the specified other process, in this case the child process terminates. At which point the wait system call returns the exit code from that child.

```
pid = fork()

if fork() == 0:
    # new child process
    exec('games/pong')
else:
    # original (parent) process
    code = wait(pid)

```

### _Process_

TODO TODO TODO TODO TODO TODO TODO TODO TODO TODO TODO TODO
TODO                                                   TODO
TODO     * Terminals - TODO (own section probably)     TODO
TODO                                                   TODO
TODO TODO TODO TODO TODO TODO TODO TODO TODO TODO TODO TODO

### _Processies Enviroment_

What is confusingly called a process enviroment is really just a chunk of data. Its some number of bytes with stuff in it. The one expectation how is that the enviroment is in the form ascii text. where each line starts with a varible name immediately followed by an equals sign (=) immediately followed by a value, where a value is really just any sequence of text characters. Ex... a typical enviroment might look something like this....

```
TERM=xterm
SHELL=/bin/bash
USER=greys
MAIL=/var/mail/ted
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin
PWD=/home/ted
EDITOR=vim
```
## _name=value_

The idea of the enviroment is that it is basically configuration data in the process itself. In fact the enviroment is always stored directly in the process itself somewhere on the heap. And the address of its location is always stored as a global variable in the data section. The idea is that the enviroment is handed down from one process to the next. So When one process is forked from another the child recieves a copy of the process in the state it was in upon the call to fork. This happens naturally because the enviroment lives in the address space of the parent process, and when we fork everything in the address space gets copied over. 

Note :: See fork / exec syscall for process creation

• Process ID Number

If processies are only created from other processies, that leaves that question where did the first process come from? When a Unix system starts, there is always a first process called the _**init process**_. And from there all other processies decend. Effectively you end up with this higherarchy of processies starting with init and in-turn their children and their children and so forth. Each process that is created is known by a unique id number, ie a _**Process ID Number**_ or _**pid**_. _**init**_ always has the pid of **1** and any subsequent process from created from there have basically whatever is the next avaible pid number numerically. Do understand that these pid numbers can be reused.

Processies not only have associated pid but they also have an associated user id. That is a number that indicates the user that owns this process. The idea of a user account is that a user account has an associated set of privileges, and so when a process is owned by an user, the process has the privileges of that user and only that user. What that effectively means, is when you execute certain system calls, those system calls may fail because the user that owns the process does not have the privilege to perform that action. For example if the user which owns the process does not have the privilege to write to a certain file, then all attempts to write to that file will fail.

## _Root / Super User_

In most Unix systems the accounts on the system are all listed in a file called _**"/etc/passwd"**_. etc is a standard dir mainly for config files. The user account given the id number **0** is a special account called the root user or the super user. This user account is special because its allowed to do anything it wants. So when a process is running with the privileges of the super user account, system calls will _**never**_ fail for privileges reasons.

Consider what happens when a Unix system starts. When a Unix system starts, its just the one process, the init process that is owned by the super user that runs with super user privileges. The init process then spawns the login process, basically a process that prompts a human to enter a user account, name and password to login ( runs with super user privileges ). Once you successful login, the login process spawns a shell process and that shell is owned by the user account that just logged in and runs with the privileges of that user account.


```
#pid 1 (init) user 0
        |            \
        |              fork, exec
        V            /
#pid 2 (login) user 0
        |            \
        |              fork, setuid, exec
        V            /
#pid 3 (shell) user 1780
```

## _Real / Effective / Saved ID_

Some what confusingly, associated with each process is not just one user id, that is the user id of the owner of the process, but actually three different ids.

• "real" ID: The owning User
• "effective" ID: Determines privileges
• "saved" ID: Set by exec to match effective ID ( saved id effectively keeps a record of whatever the effective id was at the time of the last exec call )

Note :: each file and directory is owned by a single user

## _User Groups_
```
# etc/group
```

A user group as the name implies is just an assoication of user accounts, and these groups ar non exclusive.a

• User may belong to multiple groups, but have one primary group
• Each file and directory is owned by one group.
• Each process has a real, effective, saved group id
• binary files have _setgid_ bit
• _setegid_ and _setgid_

Note :: In truth groups are almost a legecy feature of Unix systems, they are traditionally part of unix because in the early days, unix was primarly used as a multi user system, where you have multiple people logging into the same system. So it was thought unix needed some in-built mechanism to group users together but for various reasons that model of computing is out of date. You typically do not have a unix system where multiple people login to the same system, everyone has their own system now changing the paradigm.

## _Permissions_

Associated with each _file_ and each _directory_ are 9 different permissions

```
 ______     _____________________     _____________________     _____________________ 
|      |   |                    |   |                     |   |                     |
|   D  |   |       RWX          |   |         RWX         |   |         RWX         |
|______|   |____________________|   |_____________________|   |_____________________|
    |               |                          |                          |
    |               |                          |                          |
   dir            user                       group                      other

File permissions

* Read :: can read bytes of a file
* Write :: can modify bytes of a file
* Exexcute :: can _exec_ a file

Directory permissions

* Read :: you can see all the names of the files and directories which that directory contains
* Write :: means you can modify this listing, you can remove or rename or add any of the files or directories in that director
* Exexcute :: most confusing, doesnt really have anything to do with execution. When you dont have exectute permission on a directory, that effectively means you cannot use that directory in any file path. Any system call with that directory in the filepath with fail. If the directroy without execute permission is the last component of a file path, that is if the file path points to that directory itself then thats okay, its only when we try to use the directory in a file path to get a stuff _inside_ that directory that it will fail.

( * assume we do not have execute permission for "taft" dir )

/adams/taft/garfield/eisenhower  <not okay>
/adams/taft/  <okay>

```

Each one of these permissions is either on or off, set or unset. Which Class of permissions applies to a process is determined by the following rule.

```
if file_user_id == effective_user_id:
    user class
else if file_group_id == effective_group_id:
    group class
else:
    other
```


How the user ids of a process can get changed.

• The _exec_ system call may change the effective and saved ids when the binary file, the executable file being loaded has a special flag get on it called the _**setuid**_ bit. When this flag is on the exec'd executable file the effective and saved id get set to the user id of the owner of that executable file. The most common use for the _**setuid**_ mechanism is that we want a program with non-super-user privileges to be able to run a program with super-user privileges.

• The processies user ids can also be set more directly with the _**seteuid**_ && _**setuid**_ system calls. the _**seteuid**_ only sets the "effective" user id where as the _**setuid**_ sets the "real" "effective" "saved" user ids. The main limitation with this mechanism is that non-super-users can only directly set the "effective" id to match the real or saved id and only the root user can use these system calls to set processies ids.

For every process currently in the system, the OS keeps a data stucture that keeps track of everything associated with that process. And those things include ...

• Address Space - ( the memory table that is loaded when that process is running )
• User IDs
• File Discriptors
• Environment
• Current Directory
• Parent Directory
• Root Directory

```
-----------------------------------------------
|                 PROCESS                      |
|   _________________________________________  |
|  |                                         | |
|  |     Kernal Code                         | | // Fixed in size
|  |_________________________________________| |
|   _________________________________________  |
|  |                                         | |
|  |     Stack                               | | // Starts empty, grows automatically with the start of a program 
|  |_________________________________________| |
|   _________________________________________  |
|  |                                         | |
|  |     HEAP                                | | // Explicitly allocated during execution
|  |                                         | |
|  |_________________________________________| |
|                                              |
|   _________________________________________  |
|  |                                         | |
|  |     HEAP                                | | // Explicitly allocated during execution
|  |_________________________________________| |
|                                              |
|   _________________________________________  |
|  |                                         | |
|  |     Uninitialized Data                  | | // Globals **Without** initial Value
|  |_________________________________________| |
|   _________________________________________  |
|  |                                         | |
|  |     Initialized Data                    | | // Globals **With** initial Value
|  |_________________________________________| |
|   _________________________________________  |
|  |                                         | |
|  |     CODE aka "Text"                     | | // Program "text" so to speak, fixed in size 
|  |_________________________________________| |
-----------------------------------------------
```

Looking closer at address spaces, most processies include a section for whats called _**Uninitialized Data**_ && _**Initialized Data**_. These are sections of memory set aside at the start of each program for storing global variables. The difference being that in the _**Uninitialized Data**_ those global variables _do not_ have any initial value where _**Initialized Data**_ _they do_. Reason we thing of initialized globals different from uninitialized globals is that for every initialized global that value needs to be stored in the executable,the initital value, where as uninitialized globals there is no value, the executable just needs to make note of how much space it will need for addtional global variables.

How a process transitions between a few different states

```
 _________________________________________             _________________________________________ 
|                                         |           |                                         |
|       NEW process Created               |           |         process TERMINATED              |
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
```

Most obviouly a process can be _"running,"_ that is actually being currently executed by a CPU or it can be _"waiting."_ Meaning it can be waiting for the scheduler to put it in a _"running"_ state. While Running a process can also be transitioned in to the stateof being "blocked." While blocked a process will not be scheduled, so it wont ever run again until it is again placed into the "waiting" state. There are many way for which a process might get blocked, the most commen of which is from invoking certain syscalls, for example the syscall for reading from a file may block the process. General Pattern :: a process gets blocked when it has to wait for something, then either the OS or some other process will then single that process to unblock.

Note: _"Blocking"_ effectively means _"waiting,"_ but what we call _"waiting"_ refers to waiting to be scheduled or ready to be scheduled. So a blocked process waiting from some trigger to unblock, then waits _**again**_ in the _"waiting"_ state to be scheduled.

Note :: Process can get terminated for a verity of reasons, but when a process wishes to terminate itself it is done so with the _**_exit**_ system call.