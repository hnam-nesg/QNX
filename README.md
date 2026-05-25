# QNX OS

**Setup QNX Center SoftWare Center in WSL**
Download the QNX setup file for Linux from the official QNX website, then run the setup file to install QNX Software Center
```
$ cd $HOME
$ cp /mnt/c/Users/<USER_WINDOWS>/Downloads/qnx-setup-*-linux.run .
$ chmod +x qnx-setup-*-linux.run
$ ./qnx-setup-*-linux.run
```

**Run QNX Software Center and set up QNX OS to run on Raspberry Pi 5**
```
$ cd qnx/qnxsoftwarecenter
$ ./qnxsoftwarecenter
```
Once you have opened the QNX Software Center interface, select “Add Installation”, then choose QNX Software Development Platform 8.0 to create the qnx800 folder on WSL.

