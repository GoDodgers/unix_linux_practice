# unix_linux_practice

### **UNIX**

#### _Terminal Character Device File_

In unix system a process might communicate with a terminal via a special file representing that terminal -- _**Terminal Character Device File**_


When a prossess write to a _**Terminal Character Device File**_, that's putting data in the _output buffer_ of that device file, which will then be sent out by the OS to the terminal device assoicated with that device file.

When a user types, that data gets sent from the terminal to the computer and then the OS will take that data and put it in the _input buffer_ of the associated _**Terminal Character Device File**_ and then the proccess may read from the _**Device File**_ to get that data.

#### _Echoing Mode_

With a _**Terminal Character Device File**_ we can turn on a mode called _Echoing_. When a terminal character device operates in echoing mode, then any input it recieves from the terminal it will then immediately echo back out to the terminal so it will appear on screen. In practice when echoing is on, whatever key is typed by the user the key will immediately appear on screen. ( Note :: the terminal _does not_ have an echoing mode, the _**Terminal Character Device File**_ has an echoing mode, so the data is actually being sent from the terminal and then immdiately back to the terminal )

