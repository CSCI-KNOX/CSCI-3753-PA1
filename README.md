# Programming Assignment One

##### CSCI 2753: Operating Systems, String 2018

Programming Assignment One  
Due Date and Time: 6 PM, Sunday, February 4, 2018  
(Bonus for completed assignments turned in by 6pm Friday February 2)  

## Introduction:
Welcome to the first programming assignment for CSCI 3753 - Operating Systems. In this assignment we will go over how to compile and install a modern Linux kernel, as well as how to add a custom system call. Most importantly you will gain skills and set up an environment that you will use in future assignments.

You will need the access to a Pi3.

 Make note that this assignment will require you to re-compile the kernel at least twice, and you can expect each compilation to take at least two and a half hours.

## Assignment Components:

  1. Install and configure necessary tools
  2. Downloading source code for Linux
  3. Compile the kernel
  4. Add a system call
  5. Write a test program that uses the system call


## 1 Download and Configure tools

First, you have to download the the tools and linux source code that you will be using for this assignment where you will be adding your new system call, compiling the source and then running the kernel with the added system call. For this execute the following commands in the terminal.

```bash
sudo apt-get install git bc
```

### 2 Download Source code for Linux


```bash
git clone --depth 1 https://github.com/raspberrypi/linux.git
```
