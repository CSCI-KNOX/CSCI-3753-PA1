# Programming Assignment One

##### CSCI 2753: Operating Systems, String 2018

Programming Assignment One  
Due Date and Time: 6 PM, Sunday, February 4, 2018  
(Bonus for completed assignments turned in by 6pm Friday February 2)  

## Introduction:
Welcome to the first programming assignment for CSCI 3753 - Operating Systems. In this assignment we will go over how to compile and install a modern Linux kernel, as well as how to add a custom system call. Most importantly you will gain skills and set up an environment that you will use in future assignments.

You will need the following software:

  1. CSCI 3753 Fall 2017 Virtual Machine

All other software and files should be installed on the virtual machine image. It is important that you do not update the virtual machine image as all the assignments are designed and written using the software versions that the VM is distributed with. Also make note that this assignment will require you to re-compile the kernel at least twice, and you can expect each compilation to take at least half an hour and up to 5 hours on older machines.

After installing virtualbox, run ​`sudo apt-get install cu-cs-csci-3753​` to install all the necessary packages required for the course.

## Assignment Components:

  1. Configure the Grub
  2. Downloading source code for Linux
  3. Compile the kernel
  4. Add a system call
  5. Write a test program that uses the system call

## Configuring the Grub

Grub is the boot loader installed with Ubuntu 14.04. It provides configuration options to boot from a list of different kernels available on the machine. By default Ubuntu 14.04 suppresses much of the boot process from users; as a result, we will need to update our Grub configuration to allow booting from multiple kernel versions and to recover from a corrupt installation. Perform the following:

### Step 1:

From the command line, load the grub configuration file:
```bash
sudo vim /etc/default/grub
```
(feel free to replace emacs with the editor of your choice).

### Step 2:

Make the following changes to the configuration file:

  1. Comment out:  
  `GRUB_HIDDEN_TIMEOUT=0`
  2. Comment out:
  `GRUB_HIDDEN_TIMEOUT_QUIET=true`
  3. Comment out:  
  `GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"`  
and add the following new line directly below it:  
`GRUB_CMDLINE_LINUX_DEFAULT=""`
4.  Save updates

### Step 3:

From the command line, update Grub: ​`sudo update-grub​`.

### Step 4:

Reboot your virtual machine and verify you see a boot menu as shown in Figure 1.

...

## 2. Downloading Linux Source Code

First, you have to download the the tools and linux source code that you will be using for this assignment where you will be adding your new system call, compiling the source and then running the kernel with the added system call. For this execute the following commands in the terminal.

  1. cd
  2. sudo

### 2.1 Installing Tools and Downlading Kernel

```bash
sudo apt-get install git bc
```
## Title

aadfd

```bash
sudo 
```
