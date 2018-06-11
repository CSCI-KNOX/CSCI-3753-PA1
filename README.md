# Programming Assignment One
## CSCI 2753: Operating Systems, Summer 2018
Due date and time:
```
     5pm Monday June 18th, 2018
     5pm Friday June 15th, 2018 to receive **bonus** for early completion
```

## Introduction: Welcome to the first programming assignment for CSCI 3753 - Design and Analysis of Operating Systems.
In this assignment we will install and configure tools needed to compile the Linux kernel, build a new kernel, add a new custom system call, and add a new device driver.

This assignment write up is using a Raspberry Pi3 as the linux platform for all development.  It is important to note that you must download the source for the kernel that you are currently running on your development platform.  Otherwise you may have difficulty making the new kernel run on your system. You will need to re-compile the kernel at least twice, and you can expect a full re-compilation to take at least two and a half hours.

## Assignment Components:

1. Install necessary tools, download the source code for Linux, and compile the kernel
2. Add a custom system call to the kernel and write a test program that uses the system call
3. Create a new Loadable Kernel Module, dynamically install it into the kernel, and write a test program to test the LKM's functionality.

---
## 1. Download and Configure Tools

First we will make sure that our platform is up to date.

```text
sudo apt update && sudo apt upgrade -y
```
It's a good idea to reboot here.

```text
sudo reboot
```

### 1.1 Download Tools

Now let's install necessary programs.

```text
sudo apt install git bc vim libncurses5-dev make gcc ccache
```

### 1.2 Configuring git

For `git` to work properly we need to do some minimal configurations. We need to add our name, email, and editor. Replace `"Your Name"` and `youremail@example.com` with your name and your email.

```text
git config --global user.name "Your Name"
git config --global user.email youremail@example.com
```

Editor is optional.

```text
git config --global core.editor vim
```

To see the configuration.

```text
git config --list
```

If you have never used git, we recommend skimming thought this free [book](https://git-scm.com/book/en/v2).

### 1.3 Additional Tools (optional)

If your going to access your Raspberry Pi remotely, we highly recommend that you use `tmux` to allow you to have multiple text windows available both on the platform, but also can be shared with SSH connections.

```text
sudo apt install tmux
```

Tmux is a terminal multiplexer allowing you to disconnect from remote hosts while processes are still running. If, for any reason, you are disconnect form your Pi, the compile process --  if started in `tmux` --  will still be running. It also provides terminal tabs.

To start using tmux, create a new session.

```text
tmux new -s pa1
```
You will notice on the bottom of your screen a green bar with `[pa1] 0:bash*`. Also note that `pa1` can be anything. This is just a name. You can have multiple sessions at a time with different names.

Now lets detach from our session. To send a command to tmux we use `Ctrl+b`. To detach, first `Ctrl+b` followed by `d`. Now the green bar will no longer be displayed. Something to note here is that our pa1 session is still running. We can check this with:

```text
tmux ls
pa1: 1 windows (created Thu Jun  7 22:37:36 2018) [80x23]
```
to reattach to `pa1` session we use:

```text
tmux a -t pa1
```
> Note: to exit tmux type `exit` in the terminal

If you're remotely connected to the Pi and want to keep running a program but want to disconnect. You first connect to the Pi using `ssh`. Start a tmux session and run the program inside tmux. Then detach from session and disconnect for the ssh connection. The program will keep running inside tmux.

### 1.3.1 Terminal Tabs

One great thing about tmux is it allows you to have multiple tabs. To create a new tab type `Ctrl+d` followed by `c`. The bottom green bar should look something like this `[pa1] 0:bash- 1:bash*`. Notice how there is an asterisk on `1:bash*`. To move back and forth between the tabs type `Ctrl+d` then `n` for next and `p` for previous.

Tmux is a very powerful program and extremely customizable. I'll leave it up to you to explore it.

### 1.4 Download Source code for Linux

Now we will download kernel source code.

```text
cd ~
git clone --depth 1 https://github.com/raspberrypi/linux.git
```

To use the same configuration as our current kernel we need to enable configs module.

```text
sudo modprobe configs
```

Make sure to `cd` into top level of the kernel source directory.

```text
cd ~/linux
```

Next we copy the current running kernel config file to our kernel source directory.

```text
zcat /proc/config.gz > .config
```
> Note: If you get an error that config.gz was not found, make sure configs module is loaded

To edit .config file using a menu run:

```text
sudo make menuconfig
```

Select

```text
General setup --->
```

Select

```text
(-v7) Local version - append to kernel release
```
Replace `-v7` with your name. Tab to the top directory and select `<Save>`. Save it to the .config file (default).

### 1.5 Compile the Kernel

```text
sudo make -j4 CC="ccache gcc" modules dtbs zImage
```

This will take about two and a half hours the first time.  We are using the ccache utility to make subsequent compiles much faster.

```text
sudo make modules_install
```
Make sure to replace `<kernel_name>` with any name except the default's kernel name (i.e. `kernel7`).
```text
sudo cp arch/arm/boot/dts/*.dtb /boot/
sudo cp arch/arm/boot/dts/overlays/*.dtb* /boot/overlays/
sudo cp arch/arm/boot/dts/overlays/README /boot/overlays/
sudo cp arch/arm/boot/zImage /boot/$<kernel_name>.img
```

Replace `<kernel_name>` with any name you like. This name will be use in the `config.txt` file to boot this specific kernel. Make sure **NOT** to name it `kenrel7` since this is the default kernel name already installed on your system. If you do so and your platform doesn't boot, you will have to start the assignment from the beginning!

### 1.6 Rebooting to your newly built Kernel
```text
 sudo reboot
```

### 1.7 What if your new kernel does not run?

To recover from a bad kernel you need a second system to access the files on the SD card.

#### On Windows

#### On Mac

#### On Linux

Insert the card into a SD card reader. This will mount the two partitions on the SD card. Open file `/boot/config.txt` and comment out `kernel=<kenrel_name>.img`. Save the file. Now `unmount` the SD card and place it back into Pi3. Plug the Pi back in.

---

## 2. Creating a Custom System Call
### 2.1 Create source code file

Now you have to write the actual system call. First you will see that the linux source code is downloaded in the kernel folder. Go to that folder linux from the terminal. Then follow these steps.

```text
cd ~
vim linux/arch/arm/kernel/helloworld.c
```

and copy paste the following code snippet and save it. We have also provided this file so you can copy it from git.

```c
#include <linux/kernel.h>
#include <linux/linkage.h>
asmlinkage long sys_helloworld(void)
{
  printk(KERN_ALERT "hello world\n");
  return 0;
}
```

Explanation:

1. Now kernel.h header file makes us able to use the many constants and functions that are used in kernel hacking, including the `printk` function.
2. `Linkage.h` defines macros that are used to keep the stack safe and ordered.
3. Asmlinkage is a `#define` for some `gcc` magic that tells the compiler that the function should not expect to find any of its arguments in registers (a common optimization), but only on the CPU's stack. It is defined in the linkage.h header files
4. We have named the function sys_helloworld because all system calls’ name start with `sys_prefix`.
5. The function printk is used to print out kernel messages, and here we are using it with the KERN EMERG macro to print out "Hello World!" as if it were a kernel emergency. Note that depending on your system settings, printk message may not print to your terminal. Most of the time they only get printed to the kernel syslog (`/var/log/syslog`) system. If the severity level is high enough, they will also print to the terminal. Look up printk online or in the kernel docs for more information.


### 2.2 Add to kernel makefile

Now we have to tell the build system about our kernel call. Open the file `arch/arm/kernel/Makefile​`. Inside you will see a host of lines that begin with `obj+=`. After the end of the definition list, add the following line (do not place it inside any special control statements in the file)

```text
obj-y+=helloworld.o
```

### 2.3 Add to kernel jump table

Now you have to add that system call in the system table of the kernel. Go to the directory `arch/arm/tools/`​ and open the file `syscall.tbl`. Look at the file and use the existing entries to add the new system call. Ask google if you find any trouble.

Now you will add the new system call in the system call header file. Go to the location

```text
cd ~/linux/include/linux/
```

and open the file ​ `syscalls.h`​ and add the prototype of your system call at the end of the file before the endif. Check the file structure or google if you have trouble.

### 2.4 Recompile and run

Now recompile the kernel using the instructions given in the previous section. You only have to move the new kernel to `/boot/`.

### 2.5 Create test application to use new system call

Now that you have recompiled the kernel and rebooted into the installed kernel, you will be able to use the system call. Write a test C program (check google or type man syscall) to see how to call a system call and what header files to include and what are the arguments. The first argument a system call takes is the system call number we talked about before. If everything succeeds a system call returns 0 otherwise it returns -1. Check `sudo tail /var/log/syslog` to or type `dmesg` to check the `printk` outputs.

### 2.6 Create another system call taking parameters and returning a result value

If everything until now has been perfect, now you have to write a new system call. This system call will be given two numbers and an address of where to store the results.

Name the new system call ​ `cs3753_add` or `simple_add` ​ and pass the three arguments to the routine, number 1, number2, and result pointer. You will need to write a test program that calls the new system call and passes the correct type of argument.

In your system call implementation, you must use ​`printk` to log the numbers to be added, add those two numbers, store the result location, use `printk` to log the result and then print the result again in the test program that you will be running in the userspace. Do all the changes necessary to test this new system call.

---

## 3. Creating a new Device Driver
### 3.1 Create source code for new device driver
### 3.2 Modify makefile
### 3.3 Install Module
### 3.4 Uninstall Module
### 3.5 Create device file
### 3.6 Modify device driver to support open,close, read, write, seek
### 3.7 Write test application for testing new device driver

## References:

## Grading
