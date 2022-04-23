## Steps for Assignment 1
1. Create a Ubuntu 20.04 VM on VMware Workstation with atleast 200GB Disk space,8 GB Memory and  8 processor cores.
2. Install git and other neccessary required packages using the below command.


```
sudo apt install htop git build-essential  manpages-dev flex bison libncurses-dev pkg-config libssl-dev dwarves libelf-dev kernel-package ncurses-dev fakeroot wget bzip2 liblz4-tool zstd  
```

3.Run below command inside the Virtual Machine to check for vmx flags to make sure Hardware Virtualisation in enabled and working(Nested Virtualisation)

```
cat /proc/cpuinfo
```

4.Clone the forked github repo of linux source code
```
git clone https://github.com/suhasAB/linux
``` 
and get inside linux directory

```
cd linux
```
5. Check memory and disk space using following commands

- Report file system disk space usage

  ```
  df -h 
  ```
- Display amount of free and used memory in the system

  ```
  free
  ```
- Interactive process viewer

  ```
  htop
  ```
  
6. Make sure Git Repo is upto date
```
cd linux/
git status 
git remote -v
```

7. Make Commands to build the Linux Kernel
- Run the below command in /linux directory to view the config menu in a UI form.

```
make menuconfig
```

- Run below command to find arch name and it's version
```
uname -a
```

- Copy the config file from boot with the name matching with uname's version
```
cp /boot/config-$(uname -r) .config
```

- Create the following certificate files manually
  - x509.genkey file under /linux/certs
  ```
   [ req ]
  default_bits = 4096
  distinguished_name = req_distinguished_name
  prompt = no
  string_mask = utf8only
  x509_extensions = myexts

  [ req_distinguished_name ]
  O = cmpe283
  CN = cmpe283 signing key
  emailAddress = XXXobscuredXXX

  [ myexts ]
  basicConstraints=critical,CA:FALSE
  keyUsage=digitalSignature
  subjectKeyIdentifier=hash
  authorityKeyIdentifier=keyid
  ```
  
  - canonical-certs.pem file under /linux/debian
  - canonical-revoked-certs.pem file under /linux/debian
  - [Reference for debian certs](https://salsa.debian.org/kernel-team/linux/-/blob/master/debian/certs/debian-uefi-certs.pem)

- Run below make commands to build linux kernel
```
make oldconfig
make prepare
make clean
make -j 8 modules
make -j 8
sudo make INSTALL_MOD_STRIP=1 modules_install
sudo make install
sudo reboot
```
8. Once the reboot is successful, you can check if kernel version has been updated to the latest version from the cloned source repo along with timestamp
```
uname -a
```
9. Create a directory under linux folder called 'CMPE283-1' and add following files in it.
  - Makefile with contents fetched from canvas
  - cmpe283-1.c and add code that checks for VMX Controls
  
10.Run ```make ``` command inside 'linux/CMPE283-1' path to generate Kernel Object file for cmpe283-1.c
- Add the below line if you face any license related errors
```
MODULE_LICENSE("GPL v2");
```

11. Verify Kernel Object is created using command
```
ls |grep *.ko
```

12. Insert  cmpe283-1.ko into kernel by running INSMOD command
```
sudo insmod cmpe283-1.ko
```
- Verify If module is inserted by running below command. Output should be cmpe283_1
```
lsmod | grep cmpe283
```

13. Run ```dmesg``` command to display VMX Features

