# Programming Assignment One

##### CSCI 2753: Operating Systems, String 2018

Programming Assignment One  
Due Date and Time: 6 PM, Sunday, February 4, 2018  
(Bonus for completed assignments turned in by 6pm Friday February 2)  

## Introduction:
Welcome to the first programming assignment for CSCI 3753 - Operating Systems. In this assignment we will go over how to install and configure tools needed to compile the Linux kernel. We will download, compile and install a modern Linux kernel, as well as how to add a custom system call.

You will need access to a Pi3.

 Note that this assignment will require you to re-compile the kernel at least twice, and you can expect each compilation to take at least two and a half hours.

## Assignment Components:

  1. Install and configure necessary tools
  2. Downloading source code for Linux
  3. Compile the kernel
  4. Add a system call
  5. Write a test program that uses the system call


## 1 Download and Configure Tools

First we will make sure that our Pi is up to date.

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

For `git` to work properly we need to do some minimal configurations. We need to add our name, email, and editor.

```text
$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com
```

Editor is optional.

```text
$ git config --global core.editor vim
```

To see the configuration.

```text
$ git config --list
```

If you have never used git, I recommend skimming thought this free [book](https://git-scm.com/book/en/v2).

### 1.2 Additional Tools (Optional)

If your going to access your pi remotely we highly recommend that you use `tmux`.

```text
sudo apt install tmux
```

Tmux is a terminal multiplexer allowing you to disconnect while processes are running. For any reason you are disconnect form your Pi the compile process will still be running once you reconnect. It also provides terminal tabs, which improves workflow.

## 2 Download Source code for Linux

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

Next we copy the current kernel config file to our source directory.

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

Next, compile the kernel.

```text
sudo make -j4 CC="ccache gcc"
```

This will take about two and a half hours.
