Most of the content in this note comes from **Andrew S. Tanenbaum - Modern Operating System.** This is the main source, if I have used something else, it will be mentioned accordingly.

# Introduction

_Source: Page number: 32_

A modern computer consists of one or more processors, some main memory, disks, printers, a keyboard, a mouse, a display, network interfaces and various IO devices. The manage all these components optimally, there is a layer of software called the **operating system**, whose job is to provide user programs with a better, simpler, cleaner, model of the computer and to handle managing all the resources mentioned above. 

Most computers have two modes of operation: kernel mode and user mode. OS (operating system) runs in **kernel mode** (also called **supervisor mode**). In this mode it has complete access to all the hardware and can execute any instruction the machine is capable of executing. The rest of the software runs in **user mode**, in which only a subset of the machine instructions is available. In particular, those instructions that affect control of the machine or do IO, are forbidden to user-mode programs. 

![](../assets/Pasted%20image%2020260722222758.png)

The user interface program, shell or GUI, is the lowest level of user-mode software, and allows the user to start other programs, such as a Web browser, email reader etc. which also make heavy usage of OS.