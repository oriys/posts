---
title: "How Linux Works"
date: 2021-04-18T21:53:29+08:00
draft: true
---

# How Linux Works

## THE BIG PICTURE

- Levels and Layers of Abstraction in a Linux System
  - User Processes
    - GUI
    - Servers
    - Shell
  - Linux Kernel
    - System Calls
    - Process Management
    - Memory Management
    - Device Drivers
  - Hardware
    - Processor
    - Main Memory
    - Disks
    - Network Ports
- Hardware: Understanding Main Memory
  - Main memory is just a big storage area for a bunch of 0s and 1s.
  - Each slot for a 0 or 1 is called a bit.
  - running kernel and processes —- they’re just big collections of bits.
  - A CPU is just an operator on memory, it reads its instructions and data from the memory and writes data back out to the memory.
- The Kernel
  - **Processes Management** The kernel is responsible for determining which processes are allowed to use the CPU.
    - Process management describes the starting, pausing, resuming, scheduling, and terminating of processes.
    - The act of one process giving up control of the CPU to another process is called a context switch.
    - context switching
      - The CPU interrupts the current process based on an internal timer, switches into kernel mode, and hands control back to the kernel.
      - The kernel records the current state of the CPU and memory, which will be essential to resuming the process that was just interrupted.
      - The kernel performs any tasks that might have come up during the preceding time slice.
      - The kernel is now ready to let another process run. The kernel analyzes the list of processes that are ready to run and chooses one.
      - The kernel prepares the memory for this new process and then prepares the CPU.
      - The kernel tells the CPU how long the time slice for the new process will last.
      - The kernel switches the CPU into user mode and hands control of the CPU to the process.
  - **Memory Management** The kernel needs to keep track of all memory—what is currently allocated to a particular process, what might be shared between processes, and what is free.
    - Basic Conditions
      - The kernel must have its own private area in memory that user processes can’t access.
      - Each user process needs its own section of memory.
      - One user process may not access the private memory of another process.
      - User processes can share memory.
      - Some memory in user processes can be read-only.
      - The system can use more memory than is physically present by using disk space as auxiliary.
    - MMU
      - enables a memory access scheme called virtual memory
      - When using virtual memory, a process does not directly access the memory by its physical location in the hardware. Instead, the kernel sets up each process to act as if it had an entire machine to itself
    - The implementation of a memory address map is called a page table.
  - **Device Drivers and Management** The kernel acts as an interface between hardware and processes. It’s usually the kernel’s job to operate the hardware.
    - A device is typically accessible only in kernel mode because improper access could crash the machine
    - different devices rarely have the same programming interface, even if the devices perform the same task
  - **System calls and support** Processes normally use system calls to communicate with the kernel.
    - fork() When a process calls fork(), the kernel creates a nearly identical copy of the process.
    - exec() When a process calls exec(program), the kernel loads and starts program, replacing the current process.
    - pseudodevices
      - Pseudodevices look like devices to user processes, but they’re implemented purely in software. This means they don’t technically need to be in the kernel, but they are usually there for practical reasons.
      - Technically, a user process that accesses a pseudodevice must use a system call to open the device, so processes can’t entirely avoid system calls.
      - /dev/random
- User Space
  - the main memory that the kernel allocates for user processes is called user space
  - user space also refers to the memory for the entire collection of running processes
- Users
  - A user is an entity that can run processes and own files.
  - OS identifies users by simple numeric identifiers called user IDs.
  - Every user-space process has a user owner
  - root may terminate and alter another user’s processes and access any file on the local system
  - Operating as root can be dangerous
  - Groups are sets of users. The primary purpose of groups is to allow a user to share file access to other members of a group.

## BASIC COMMANDS AND DIRECTORY HIERARCHY

- The Bourne Shell: /bin/sh
  - A shell is a program that runs commands
  - Alternative
    - bash
    - zsh
    - csh
- Using the Shell
  - The Shell Window
    - The easiest way to open a shell window from a GUI like Gnome or KDE is to open a terminal application, which starts a shell inside a new window.
  - cat
    - concatenation
    - outputs the contents of one or more files or another source of input.
  - Standard I/O, Standard error
    - Standard Input
      - Processes read data from input streams and write data to output streams. Streams are very flexible. For example, the source of an input stream can be a file, a device, a terminal window, or even the output stream from another process.
    - Standard Output
      - The kernel gives each process a standard output stream where it can write its output.
    - Standard error
      - an additional output stream for diagnostics and debugging.
    - CTRL-D
      - Pressing CTRL-D on an empty line stops the current standard input entry from the terminal with an EOF (end-of-file) message (and often terminates a program).
    - CTRL-C
      - terminates a program regardless of its input or output
- Basic Commands
  - ls
    - The ls command lists the contents of a directory.
  - cp
    - copy files
  - mv
    - move files to other directories
    - rename file
  - touch
    - create a file
  - rm
    - delete a file
  - echo
    - prints its arguments to the standard output
- Navigating Directories
  - /
    - root directory
  - pahtname
    - ..
      - parent of a directory
    - .
      - current directory
    - absulute path
      - a path starts with /
    - relative path
      - a path not starts with /
  - cd
    - shell built-in
    - if you omit dir, the shell returns to your home directory
  - mkdir
    - createa a new directory
  - rmdir
    - removes the directory
  - Shell Globbing
    - \*
      - The shell matches arguments containing globs to filenames, substitutes the filenames for those arguments, and then runs the revised command line.
      - at\* expands to all filenames that start with at.
      - \*at expands to all filenames that end with at.
      - \*at\* expands to all filenames that contain at.”
    - ?
      - match exactly one arbitrary character
    - single quotes
      - If you don’t want the shell to expand a glob in a command, enclose the glob in single quotes
  - Intermediate Commands
    - grep
      - prints the lines from a file or input stream that match an expression.
    - less
      - The less command comes in handy when a file is really big or when a command’s output is long and scrolls off the top of the screen.
      - an enhanced version for more
    - pwd
      - print working directory
    - diff
      - To see the differences between two text files
    - file
      - to see a file and ensure its format
    - find
      - find a file in a directory tree
    - locate
      - locate searches an index that the system builds periodically
    - head
      - show the first n lines of a file
    - tail
      - show the last n lines of a file
    - sort
      - quickly puts the lines of a text file in alphanumeric order
  - Changing Your password and Shell
    - passwd
      - change your password
    - chsh
      - change your shell
  - Dot Files
    - files and directories whoes names begin with a dot
    - ls -a to display dot files
  - Environment and Shell Variables
    - To assign a value to a shell variable, use the equal sign (=)
    - Don’t put any spaces around the = when assigning a variable.
    - use export to export the shell variables to the env
  - The Command Path
    - PATH is a special environment variable that contains the command path (or path for short), a list of system directories that the shell searches when trying to locate a command.
- Special Characters
  | Character | Name(s) | Uses |
  | --------- | ------------------------ | -------------------------------------------------------------------------------------------------------------------- |
  | \* | star, asterisk | Regular expression, glob character |
  | . | dot | Current directory, file/hostname delimiter |
  | ! | bang | Negation, command history |
  | \| | pipe | Command pipes |
  | / | (forward) slash | Directory delimiter, search command |
  | \ | backslash | Literals, macros (never directories) |
  | $ | dollar | Variables, end of line |
  | ' | tick, (single) quote | Literal strings |
  | ` | backtick, backquote | Command substitution |
  | " | double quote | Semi-literal strings |
  | ^ | caret | Negation, beginning of line |
  | ~ | tilde, squiggle | Negation, directory shortcut |
  | # | hash, sharp, pound | Comments, preprocessor, substitutions |
  | \[ \] | (square) brackets | Ranges |
  | \{ \} | braces, (curly) brackets | Statement blocks, ranges |
  | \_ | underscore, under | Cheap substitute for a space used when spaces aren’t wanted or allowed, or when autocomplete algorithms get confused |
- Command Line Editing
  | Keystroke | Action |
  | --------- | ------------------------------------------------- |
  | CTRL-B | Move the cursor left |
  | CTRL-F | Move the cursor right |
  | CTRL-P | View the previous command (or move the cursor up) |
  | CTRL-N | View the next command (or move the cursor down) |
  | CTRL-A | Move the cursor to the beginning of the line |
  | CTRL-E | Move the cursor to the end of the line |
  | CTRL-W | Erase the preceding word |
  | CTRL-U | Erase from cursor to beginning of line |
  | CTRL-K | Erase from cursor to end of line |
  | CTRL-Y | Paste erased text (for example, from CTRL-U) |
- Text Editors
  - Emacs
  - Vim
- Getting Online Help
  | Section | Description |
  | ------- | --------------------------------------------------------------------- |
  | 1 | User commands |
  | 2 | Kernel system calls |
  | 3 | Higher-level Unix programming library documentation |
  | 4 | Device interface and driver information |
  | 5 | File descriptions (system configuration files) |
  | 6 | Games |
  | 7 | File formats, conventions, and encodings (ASCII, suffixes, and so on) |
  | 8 | System commands and servers |
- Shell Input and Output
  - \>
    - The shell creates file if it does not already exist.
  - \>\>
    - append the output to the file instead of overwriting it
  - <
    - channel a file to a program’s standard input, use the < operator:
  - Understanding Error Messages
    - No such file or directory
    - File exists
    - Not a directory
    - Is a directory
    - No space left on device
    - Permission denied
    - Operation not permiitted
    - Segmentation fault, Bus error
- Listing and manipulating Processes
  - ps
    - PID
    - TTY
    - STAT
    - TIME
      - cpu time
    - COMMAND
    - x show all of your running processes
    - ax show all processes on the system
    - u include more detailed information on proceses
    - w show full command names
  - Process Termination
    - kill
  - Job Control
    - kill STOP
    - kill CONT
    - CTRL-Z
    - jobs
    - tmux
  - Background Processes
    - detach a process from the shell and put it in the “background” with the ampersand
    - nohup
- File Modes and Permissions
  - permissions
    - r means that the file is readable.
    - w means that the file is writable.
    - x means that the file is executable.
    - \- means “nothing”.
  - User types
    - User
    - Group
    - Other
  - Modify Permissions
    - chmod
  - Working with Symbolic Links
    - ln -s target linkname
- Archiving and Compressing Files
  - gzip
  - tar
  - zcat
  - xz
  - bzip2
  - zip
  - unzip
- Linux Directory Hierarchy Essentials
  - /bin
    - Contains ready-to-run programs
  - /dev
    - Contains device files
  - /etc
    - Core system configuration directory
  - /home
    - Holds home directories for regular users
  - /lib
    - holds library files containing code that executables can use
  - /proc
    - Provides system statistics through a browsable directory-and-file interface
  - /run
    - Contains runtime data specific to the system, including certain process IDs, socket files, status records, and, in many cases, system logging.
  - /sys
    - This directory is similar to /proc in that it provides a device and system interface
  - /sbin
    - The place for system executables
  - /tmp
    - A storage area for smaller, temporary files that you don’t care much about.
  - /usr
    - most of the user-space programs and data reside.
  - /var
    - The variable subdirectory, where programs record information that can change over the course of time
  - /boot
    - Contains kernel boot loader files.
  - /media
    - A base attachment point for removable media such as flash drives that is found in many distributions.
  - /opt
    - This may contain additional third-party software.
  - /include
    - Holds header files used by the C compiler.
  - /local
    - Is where administrators can install their own software.
  - /man
    - Contains manual pages.
  - /share
    - Contains files that should work on other kinds of Unix machines with no loss of functionality.
- kernel Location
  - /mvlinuz or /boot/vmlinuz
  - normally a binary file
- Running Commands as the Superuser
  - sudo
    - allow administrators to run commands as root when they are logged in as themselves
  - /etc/sudoers
    - configure the privileged users in your /etc/sudoers file
    - visudo checks for file syntax
  - sudo logs
    - `journalctl SYSLOG_IDENTIFIER=sudo`

## DEVICES

- Device Files
  - Block device
    - Programs access data from a block device in fixed chunks.
  - Character device
    - Character devices work with data streams.
    - You can only read characters from or write characters to character devices
    - During character device interaction, the kernel cannot back up and reexamine the data stream after it has passed data to a device or process.
  - Pipe device
    - Named pipes are like character devices, with another process at the other end of the I/O stream instead of a kernel driver.
  - Socket device
    - Sockets are special-purpose interfaces that are frequently used for interprocess communication.
    - Socket files represent Unix domain sockets
- The sysfs Device Path
  - /sys/devices
  - To provide a uniform view for attached devices based on their actual hardware attributes, the Linux kernel offers the sysfs interface through a system of files and directories.
- dd and Devices
  - Its sole function is to read from an input file or stream and write to an output file or stream, possibly doing some encoding conver- sion on the way.
  - `dd if=/dev/zero of=new_file bs=1024 count=1`
  - important dd opts
  - if=file
    - The input file. The default is the standard input.
  - of=file
    - The output file. The default is the standard output.
  - bs=size
    The block size.
  - ibs=size, obs=size
    - The input and output block sizes.
  - count=num
    - The total number of blocks to copy.
  - skip=num
    - Skip past the first num blocks in the input file or stream, and do not copy them to the output.
- Device Name Summary
  - Hard Disk: /dev/sd\*
  - Virtual Disks: /dev/xvd\*, /dev/vd\*
  - Non-Volatile Memory Devices: /dev/nvme\*
  - Device Mapper: /dev/dm-\*, /dev/mapper/\*
  - CD and DVD Drives: /dev/sr\*
  - PATA Hard Disks: /dev/hd\*
  - Terminals: /dev/tty*, /dev/pts/*, and /dev/tty
  - Serial Ports: /dev/ttyS*, /dev/ttyUSB*, /dev/ttyACM\*
  - Parallel Ports: /dev/lp0 and /dev/lp1
  - Audio Devices: /dev/snd/\*, /dev/dsp, /dev/audio, and More
  - Device File Creation
    - mknod
- udev
  - The Linux kernel can send notifica- tions to a user-space process called udevd upon detecting a new device on the system.
  - This udevd process could examine the new device’s characteristics, create a device file, and then perform any device initialization.
  - devtmpfs
    - The devtmpfs filesystem was developed in response to the problem of device availability during boot.
  - udevd Operation and Configuration
    - The kernel sends udevd a notification event, called a uevent, through an internal network link.
    - udevd loads all of the attributes in the uevent.
    - udevd parses its rules, filters and updates the uevent based on those rules, and takes actions or sets more attributes accordingly.
  - udevadm
    - The udevadm program is an administration tool for udevd.
  - Device Monitoring
    - `udevadm monitor`
    - `udevadm monitor --kernel --subsystem-match=scsi`
- In-Depth: SCSI and the Linux Kernel
  - SCSI subsystem and its three layers of drivers:
    - [Linux SCSI subsystem schematic](https://apprize.best/linux/superuser/superuser.files/image010.jpg)
    - The top layer handles operations for a class of device. For example, the sd (SCSI disk) driver is at this layer; it knows how to translate requests from the kernel block device interface into disk-specific commands in the SCSI protocol, and vice versa.
    - The middle layer moderates and routes the SCSI messages between the top and bottom layers, and keeps track of all of the SCSI buses and devices attached to the system.
    - The bottom layer handles hardware-specific actions. The drivers here send outgoing SCSI protocol messages to specific host adapters or hardware, and they extract incoming messages from the hardware. The reason for this separation from the top layer is that although SCSI messages are uniform for a device class (such as the disk class), different kinds of host adapters have varying procedures for sending the same messages.
