# Programming Assignment One
## CSCI 2753: Operating Systems, Summer 2018
Due date and time:
```
     5pm Monday June ???, 2018
     5pm Friday June ???, 2018 to receive **bonus** for early completion
```

## Introduction: Welcome to the first programming assignment for CSCI 3753 - Design and Analysis of Operating Systems. 
In this assignment we will install and configure tools needed to compile the Linux kernel, build a new kernel, add a new custom system call, and add a new device driver. 

This assignment write up is using a Raspberry Pi3 as the linux platform for all development.  It is important to note that you must download the source for the kernel that you are currently running on your development platform.  Otherwise you may have difficulty making the new kernel run on your system. You will need to re-compile the kernel at least twice, and you can expect a full re-compilation to take at least two and a half hours.

## Assignment Components:

1. Install necessary tools, download the source code for Linux, and compile the kernel
2. Add a custom system call to the kernel and write a test program that uses the system call
3. Create a new Loadable Kernel Module, dynamically install it into the kernel, and write a test program to test the LKM's functionality.

## 1 Download and Configure Tools

First we will make sure that our Pi is up to date.

```text sudo apt update && sudo apt upgrade -y ``` It's a good idea to reboot here.

```text sudo reboot ```

### 1.1 Download Tools

Now let's install necessary programs.

```text sudo apt install git bc vim libncurses5-dev make gcc ccache ```

### 1.2 Configuring git

For `git` to work properly we need to do some minimal configurations. We need to add our name, email, and editor.

```text $ git config --global user.name "John Doe" $ git config --global user.email johndoe@example.com ```

Editor is optional.

```text $ git config --global core.editor vim ```

To see the configuration.

```text $ git config --list ```

If you have never used git, we recommend skimming thought this free [book](https://git-scm.com/book/en/v2).

### 1.2 Additional Tools (optional, but recommended)

If your going to access your Pi remotely, we highly recommend that you use `tmux`.

```text sudo apt install tmux ```

Tmux is a terminal multiplexer allowing you to disconnect from remote hosts while processes are still running. If, for any reason, you are disconnect form your Pi, the compile process --  if started in `tmux` --  will still be running. It also provides terminal tabs.

To start using tmux, create a new session.

```text tmux new -s pa1 ``` You will notice on the bottom of your screen a green bar with `[pa1] 0:bash*`. Also note that `pa1` can be anything. This is just a name. You can have multiple sessions at a time with different names.

Now lets detach from our session. To send a command to tmux we use `Ctrl+b`. To detach, first `Ctrl+b` followed by `d`. Now the green bar will no longer be displayed. Something to note here is that our pa1 session is still running. We can check this with:

```text tmux ls pa1: 1 windows (created Thu Jun  7 22:37:36 2018) [80x23] ``` to reattach to `pa1` session we use:

```text tmux a -t pa1 ``` > Note: to exit tmux type `exit` in the terminal

If you're remotely connected to the Pi and want to keep running a program but want to disconnect. You first connect to the Pi using `ssh`. Start a tmux session and run the program inside tmux. Then detach from session and disconnect for the ssh connection. The program will keep running inside tmux.

### 1.2.1 Terminal Tabs

One great thing about tmux is it allows you to have multiple tabs. To create a new tab type `Ctrl+d` followed by `c`. The bottom green bar should look something like this `[pa1] 0:bash- 1:bash*`. Notice how there is an asterisk on `1:bash*`. To move back and forth between the tabs type `Ctrl+d` then `n` for next and `p` for previous.

Tmux is a very powerful program and extremely customizable. I'll leave it up to you to explore it.

## 2 Download Source code for Linux

Now we will download kernel source code.

```text cd ~ git clone --depth 1 https://github.com/raspberrypi/linux.git ```

To use the same configuration as our current kernel we need to enable configs module.

```text sudo modprobe configs ```

Make sure to `cd` into top level of the kernel source directory.

```text cd ~/linux ```

Next we copy the current kernel config file to our source directory.

```text zcat /proc/config.gz > .config ``` > Note: If you get an error that config.gz was not found, make sure configs module is loaded

To edit .config file using a menu run:

```text sudo make menuconfig ```

Select

```text General setup ---> ``` Select

```text (-v7) Local version - append to kernel release ``` Replace `-v7` with your name. Tab to the top directory and select `<Save>`. Save it to the .config file (default).

Next, compile the kernel.

```text sudo make -j4 CC="ccache gcc" ```

This will take about two and a half hours.
