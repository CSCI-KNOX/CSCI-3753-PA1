# Programming Assignment One
## CSCI 3753: Operating Systems, Fall 2018


## Introduction: Welcome to the first programming assignment for CSCI 3753 - Design and Analysis of Operating Systems.
In this assignment we will install and configure tools needed to compile the Linux kernel, build a new kernel, add a new custom system call, and add a new device driver.

This assignment write up is using a Raspberry Pi3 as the linux platform for all development.  It is important to note that you must download the source for the kernel that you are currently running on your development platform.  Otherwise you may have difficulty making the new kernel run on your system. You will need to re-compile the kernel at least twice, and you can expect a full re-compilation to take at least two and a half hours.

## Assignment Components:

 1. **Install necessary tools, download the source code for Linux, and compile the kernel**

 2. **Add a custom system call to the kernel and write a test program that uses the system call**

 3. **Create a new Loadable Kernel Module, dynamically install it into the kernel, and write a test program to test the LKM's functionality.**

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

Now let's install necessary programs if you have not already installed them.
```text
sudo apt install git vim libncurses5-dev make gcc ccache
```

### 1.2 Configuring `git`

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

### 1.3 Download Source code for Linux

Now we will download kernel source code if you have not already downloaded it.

```text
cd ~
git clone --depth 1 https://github.com/raspberrypi/linux.git
```

To use the same configuration as our current kernel that is running, we need to enable configs module.

```text
sudo modprobe configs
```

Make sure to `cd` into top level of the kernel source directory.

```text
cd ~/linux
```

Next we copy the current running kernel config file to our kernel source directory.  We uncompress the virtual file containing the current configuration using `zcat` and place the file into `.config` in the `~/linux` directory.  This configuration file specifies the options for building a kernel. We want to build one exactly like the one currently running, except we will add our own system call.

```text
zcat /proc/config.gz > .config
```
> Note: If you get an error that config.gz was not found, make sure configs module is loaded

To edit .config file using a menu run:
```text
make menuconfig
```
Once the character based menuing is displayed, scroll down to select:
```text
General setup --->
```
In the general setup menu, we want to select the name of our kernel version.  It is currently set to `-v7`, which is appended to the kernel file when we build it.  Scroll down to the local version and select it.
```text
(-v7) Local version - append to kernel release
```
Replace `-v7` with your name. Tab to the top directory and select `<Save>` and save it to the .config file (default).  Use left arrow to select the `Exit` from the general setup menu and left arrow again to select `Exit` from the `menuconfig` application.

### 1.4 Compile the Kernel

```text
make -j4 CC="ccache gcc" modules dtbs zImage
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
sudo cp arch/arm/boot/zImage /boot/<kernel_name>.img
```

Replace `<kernel_name>` with any name you like.  Make sure **NOT** to name it `kenrel7` since this is the default kernel name already installed on your system. If you do so and your platform doesn't boot, you will have to start the assignment from the beginning!  (See 1.7)

### 1.5 Telling the boot process to use your kernel
When the system is booting, it gets the name of the kernel from the `config.txt` file that is located in the `boot` partition.  To specify that you want to load your new kernel, you must edit the `config.txt` file and add a line:
```
kernel=<kernel_name>.img
```
where <kernel_name> is the name you used in the `cp` command above to copy the kernel into the boot partition.

### 1.6 Rebooting to your newly built Kernel
```text
 sudo reboot
```
You are now booting and running your newly built kernel.

### 1.7 What if your new kernel does not run?

If your kernel boots, but does not behave correctly, you can edit the `<device>/boot/.config` file and remove (or comment out) the `kernel=<kernel_name>.img` line, which will then default back to the original kernel.  Save the file and reboot.  You should be back to where you were before step 1.6 was performed.

To recover from a bad kernel (one that will not boot) you need a second system to access the files on the SD card.  Turn OFF the power to your platform and remove the micro SD card.
You will need to mount this device in another computer via an SD card reader or or USB converter. Once you have mounted the device, you can look in its boot partition.
```
ls <device>/boot
```
You can edit the `<device>/boot/.config` file and remove (or comment out) the `kernel=<kernel_name>.img` line, which will then default back to the original kernel.  Save the file back to the SD card, unmount the device, and place it back into platform. Turn the power back ON and your platform will boot the previously working kernel.

---

## 2. Creating a Custom System Call
### 2.1 Create source code file

Now you have to write the actual system call. You will be adding code into the kernel directories and making that source part of the kernel build.  

```text
cd ~/linux/arch/arm/kernel
ls
vim helloworld.c
```
You will be saving a new file in the kernel directory when you save the file.

and copy paste the following code snippet and save it. We have also provided this file so you can copy it from git.

```c
1. #include <linux/kernel.h>
2. #include <linux/linkage.h>
3. asmlinkage long sys_helloworld(void)
4. {
5.   printk(KERN_ALERT "hello world\n");
6.   return 0;
7. }
```

Explanation:

1. Now kernel.h header file makes us able to use the many constants and functions that are used in kernel hacking, including the `printk` function.
2. `Linkage.h` defines macros that are used to keep the stack safe and ordered.
3. Asmlinkage is a `#define` for some `gcc` magic that tells the compiler that the function should not expect to find any of its arguments in registers (a common optimization), but only on the CPU's stack. It is defined in the linkage.h header files
4. We have named the function sys_helloworld because all system calls’ name start with `sys_prefix`.
5. The function printk is used to print out kernel messages, and here we are using it with the KERN EMERG macro to print out "Hello World!" as if it were a kernel emergency. Note that depending on your system settings, printk message may not print to your terminal. Most of the time they only get printed to the kernel syslog (`/var/log/syslog`) system. If the severity level is high enough, they will also print to the terminal. Look up printk online or in the kernel docs for more information.


### 2.2 Add to kernel makefile

Now we have to tell the build system about our kernel call. Open the file `linux/arch/arm/kernel/Makefile​`. Inside you will see a host of lines that begin with `obj+=`. After the end of the lines adding objects to the list, add the following line (but do not place it inside any special control statements in the file)

```text
obj-y+=helloworld.o
```
This line adds your new system call code to the list of files to be built with the kernel.

### 2.3 Add to kernel jump table

Now you have to add that system call in the system table of the kernel. Go to the directory `linux/arch/arm/tools/`​ and open the file `syscall.tbl`. Look at the file and use the existing entries to add the new system call. Ask google if you find any trouble.  You will add your new system call to the end of the system call header file.
```text
vim ~/linux/arch/arm/tools/syscall.tbl
```
You must also add your call to the list of system calls in the `syscalls.h` include file.

```text
vim ~/linux/include/linux/syscalls.h
```

Add the prototype of your system call at the end of the file before the endif. Check the file structure or google if you have trouble.

### 2.4 Recompile and run

Now recompile the kernel using the instructions given in the previous section (see section 1.5). You only have to move the new kernel to `/boot` directory.

### 2.5 Create test application to use new system call

Now that you have recompiled the kernel and rebooted into the installed kernel, you will be able to use the system call. Write a test C program (check google or type man syscall) to see how to call a system call and what header files to include and what are the arguments. The first argument a system call takes is the system call number we talked about before. If everything succeeds a system call returns 0 otherwise it returns -1. Check `tail /var/log/syslog` to or type `dmesg` to check the `printk` outputs.

### 2.6 Create another system call taking parameters and returning a result value

If everything until now has been perfect, now you have to write a new system call. Name the new system call `cs3753_add`, which has three parameters and returns 0 if no error was detected.
​The call takes the first two integer parameters and adds them together.  The third parameter is an address (in user space) of an integer in which the results will be stored.   You will need to write a test program that calls the new system call and passes the correct type of arguments.  You should test all types of possible errors that could occur in passing values and parameters to the system call.

In your system call implementation, you *must* use ​`printk` to log the numbers to be added, add those two numbers, store the result location.  Again, use `printk` to log the result being calculated in the system call.  Finally, `printf` in the test program (running in userspace) to show the returned value.

### 2.7 **You MUST Submit Your Work for sections 1 and 2**
After you have completed sections 1 and 2, please submit your code for the new system call you have created along with your test program.  Create a zip file (use filename: `<your last name>_PA1_CHECKPOINT.zio`) with all the files you have modified to create your new system call.  Submit that zip file as your submission on Moodle for PA1 Checkpoint.

---

## 3. Creating a new Device Driver
If you want to add code to a Linux kernel, the usual method is to add some source files to the kernel source tree and recompile the kernel. This is what you did in the first part of this assignment.  After each change, the kernel must be recompiled, copied into the boot directory, and the computer must be rebooted.  Again, you did this repeatedly in the first part of this assignment when you added a system call. After you copied the kernel to the boot partition and rebooted, the changes that you made were installed in the kernel.  If more changes are required, you needed to repeat the whole process again.

But you can also add code to the Linux kernel while it is running. A chunk of code that you add in this way is called a loadable kernel module (LKM). These modules can perform any function for the OS, but they have there typical uses:
1. device drivers
2. filesystem drivers
3. system calls

The kernel isolates certain functions, including the modules, especially well and therefore they don't have to be intricately wired into the rest of the kernel.  The part of the kernel that is bound into the image that you boot (i.e. all of the kernel except the LKMs) is called the “base kernel.” LKMs communicate with the base kernel.

 There is a tendency to think of LKMs like user space programs.  Modules do share a lot of user space program properties, but LKMs are definitely not user space programs. LKMs (when loaded) are very much part of the kernel.  As such, they have free run of the system and can easily crash it.

LKMs have several advantages:
1. You don't have to rebuild your kernel
2. LKMs help you diagnose system problems. A bug in a device driver which is bound into the kernel can stop your system from booting at all;
3. LKMs can save you memory, because you have to have them loaded only when you're actually using them
4. LKMs are much faster to maintain and debug.

### Building Loadable Kernel Modules (LKM)
LKMs are object files used to extend a running kernel’s functionality. This is basically a piece of binary code that can be inserted and installed in the kernel on the fly without the need to reboot. This is very handy when you are trying to work with some new device and will be repeatedly be writing and testing your code.  It is very convenient to write system code, install it, test it, and then uninstall it, without ever needing to reboot the system.  

### 3.1 Create source code for new device driver
The kernel uses jump tables to call the correct device drivers and functions of those drivers.  Each LKM must define a standard jump table to support the kernels dynamic use of the module.  The easiest way to understand the functionality that must be implemented, is to create a simple module. We will create a new module `helloworld` that will log the functions being called.
In the project directory you should find the `hellomodule.c` and `Makefile` files. Open the `hellomodule.c` file in your editor.  
This simple source file has all the code needed to install and uninstall an LKM in the kernel.  There are two macros listed at the bottom of the source file that setup the jump table for this LKM.  Whenever the module is installed, the kernel will call the routine specified in the `module_init` macro, and the routine specified in the `module_exit` will be called when the module is uninstalled.

### 3.2 Create a Makefile

Now you have to compile your module.  There are a couple of ways to add our module to the list of modules to be built for a kernel.  One is to modify the makefile used by the kernel build.  The other is to write our own local makefile and attach it to the build when you want to make the modules.  Create your own make file,  create named `Makefile` and type the following lines in the file:

```Makefile
obj-m:= hellomodule.o
all:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules
```

Here `m` in `obj-m` means module and you are telling the compiler to create a module object named hellomodule.o as the result.  To build the module you will build modules for the kernel run:

```
make
```


You will see there is now a file named `hellomodule.ko`. This is the kernel module (.ko) object you will be
inserting in the basic kernel image.

### 3.3 Install Module
To insert the module, type the following command:

```
sudo insmod hellomodule.ko
```

The kernel has tried to insert your module.  If it is successful, you will see the log message that has been inserted into `/val/logs/system.log`.  If you type `lsmod` you will see your module is now inserted in the kernel.   

### 3.4 Uninstall Module
To remove the kernel use the following command:

```
sudo rmmod hellomodule
```

To verify that the module was uninstalled, check the system log and you should see our module exit message.
You can also use the `lsmod` command to verify the module is no longer in the system.

### 3.5 Create device file
The device drivers can be dynamically installed into the kernel.  How does the kernel know which device driver to use with which device?  Each device will have corresponding device file that is located in the `/dev` directory.  If you list the file in that directory you will see all the devices currently known by the kernel.  These are not regular files.  They are virtual files that only supply data from or give data to the device.  

To add a new device you need to create a new entry in the `/dev` directory.  Using the `mknod` command, you can create a new entry.

```
sudo mknod -m <permission> <location> <type of driver> <major number> <minor number>
```

For our example we will create a device called `simple_character_device` with permissions (using standard file permissions [r,w,e]), is a character device (`c`) with major number of `240`. The major number should be unique and you can look at current devices already installed, but usually user modules start at 240.
```
        sudo mknod –m 777 /dev/simple_character_device c 240 0
```

### 3.6 Modify device driver to support open, close, read, write, seek
Using your `hellomodule.c` as template, create a new device driver that will be modified to support the following functions:  open, read, write, seek, close.   You will need to create a buffer to store the data for this device.  It will exist as long as the module is installed.  Once it is uninstalled, all data will be lost.


###     -------NEED TO EXPLAIN INTERNAL JUMP TABLE AND FILE STRUCT-------

### 3.7 Write test application for testing new device driver
Using the basic interactive testing code we have provided, test your code for `open/close`.
Then test your `write` code.  Then your `read` code.  Now you have a working device that can read and write.  But you also need to be able to seek to a location and perform functions from that location within the data.  You must implement `seek` and also add code to the testing application to call the seek system call.


### 3.8 Utilities for Loadable Kernel Modules
* _insmod_:   Insert an LKM into the kernel.
* _rmmod_:    Remove an LKM from the kernel.
* _depmod_:   Determine interdependencies between LKMs.
* _kerneld_:  Kerneld daemon program
* _ksyms_:    Display symbols that are exported by the kernel for use by new LKMs.
* _lsmod_:    List currently loaded LKMs.
* _modinfo_:  Display contents of .modinfo section in an LKM object file.
* _modprobe_: Insert/remove an LKM or set of LKMs intelligently (e.g., if module A must be loaded before loading module B, modprobe will automatically load A when module B is requested to be loaded)

## 3.9 **You MUST Submit Your Work for section 3**
After you have completed section 3, please submit your code for the new device driver you have created.  Create a zip file (use filename: `<your last name>_PA1.zio`) with all the files you have modified to create your new device driver.  Submit that zip file as your submission on Moodle for PA1.

---

## References:
1. You can use the Linux manual pages to check the functions and their functionalities.
2. http://www.fsl.cs.sunysb.edu/kernel-api/re941.html
3. http://lxr.free-electrons.com/ident?i=unregister_chrdev
4. http://www.fsl.cs.sunysb.edu/kernel-api/re256.html
5. http://www.fsl.cs.sunysb.edu/kernel-api/re257.html

## Grading
* 20% working code
* 80% interview
