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
-Display amount of free and used memory in the system

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
  - canonical-certs.pem file under /linux/debian
  - canonical-revoked-certs.pem file under /linux/debian

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
8. Once the reboot is successful, you can check if you are using newly compiled kernel by doing below command in terminal. The Kernel
```
uname -a
```
