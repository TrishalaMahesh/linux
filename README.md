# CMPE-283(Spring'22) Assignments:
## Team Members
- Trishala M (SJSU ID:015219646)
- Suhas Anand Balagar (SJSU ID:015243540)
## Assignment 1: To Discover VMX features
### Collaborative efforts:

1. Installed VMware Workstation and created a Ubuntu 20.04 VM 
2. Installed all the neccesary packages required for building linux kernel
3. Cloned torvalds original linux github repo into VM
4. Built latest version of Linux module using various make commands and verified it's installation after rebooting
5. Created 283-1 folder and adder required files to generate Kernel Object file and verfied VMX features
6. Added functionality to determine if secondary procbased controls are available and check the ability to set “Activate Secondary 
Controls” control in the primary procbased controls.

  
  
  ### Trishala Contributions:
  Added code in CMPE283-1.C to support reading following MSRs to detect VMX features
  - Primary procbased controls
  - Secondary procbased controls
 
  ###  Suhas Contributions:
  Added code in CMPE283-1.C to support reading following MSRs to detect VMX features
  - Entry based procbased controls
  - Exit based procbased controls
 
## Assignment 1 Steps:
1. Create a Ubuntu 20.04 VM on VMware Workstation with 200GB Disk space,8 GB Memory and  8 processor cores.
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
14. Below are the screenshots of the output:

![Assignment 1a](https://github.com/TrishalaMahesh/linux/blob/master/screenshots/1a.png)
![Assignment 1b](https://github.com/TrishalaMahesh/linux/blob/master/screenshots/1b.png)
![Assignment 1c](https://github.com/TrishalaMahesh/linux/blob/master/screenshots/1c.png)


## Assignment 2 and 3:Instrumentation via hypercall
### Collaborative efforts:

1. After making changes to the vmx.c and cpuid.c files, we followed the steps as mentioned below to compile and make the modules. 
2. Built and loaded the kerenl again for outer VM.
3. Created an Inner VM with Ubuntuu 20.04 image using virt-manager.
4. Captured the output for the individual cpuid leaf nodes inside the inner VM.


  ### Trishala Contributions:
  - Added feature to determine the total number of exits for CPUID - 0X4FFFFFFF
  - Added feature to determine the number of exits for the exit number provided (on input) in %ecx for CPUID=0x4FFFFFFD
 
  ###  Suhas Contributions
  - Added feature to detect the time spent inside the VM for processing all exits for CPUID - 0X4FFFFFFE
  - Added feature to detect the time spent processing the exit number provided (on input) in %ecx for CPUID - 0X4FFFFFFC

## Assignment 2 and 3 steps:

1. Pre-requisite: Working model of assignment 1.
2. Modify the cpuid.c & vmx.c files to support exits for CPUID leaf nodes (0x4fffffff, 0x4ffffffe,0x4ffffffc,0x4ffffffd)
3. The following steps were perfomed as shown below to build the KVM Module and to install kernel:
```
sudo make -j 8
sudo make INSTALL_MOD_STRIP=1 modules_install
sudo make install
sudo rmmod kvm_intel
sudo rmmod kvm 
lsmod|grep kvm 
```
4. Then inner VM (ubuntuu) was created inside existing VM by installing Virtual manager using below commands:
```
sudo apt update
sudo apt install cpu-checker
kvm-ok
sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virtinst virt-manager
sudo systemctl is-active libvirtd
sudo usermod -aG kvm $USER
sudo usermod -aG libvirt $USER
newgrp libvirt
```
5. Then started the virtual manager using the below command:
```
virt-manager
```
6. In the inner VM install following packages:
```
sudo apt-get update -y
sudo apt-get install -y cpuid"
```
7. The following commands were run in nested VM to check for cpuid leaf nodes exits
```
    cpuid -l 0x4fffffff
    cpuid -l 0x4ffffffe
    cpuid -l 0x4ffffffc -s {exit_type}
    cpuid -l 0x4ffffffd -s {exit_type}
    
 ```    
 The output screenshots for assignment 2 are as below:
![Assignment 2](https://github.com/TrishalaMahesh/linux/blob/master/screenshots/2a.png)
![Assignment 2](https://github.com/TrishalaMahesh/linux/blob/master/screenshots/2b.png)

 The output screenshots for assignment 3 are as below:
![Assignment 3](https://github.com/TrishalaMahesh/linux/blob/master/screenshots/3a.png)
![Assignment 3](https://github.com/TrishalaMahesh/linux/blob/master/screenshots/3b.png)
![Assignment 3](https://github.com/TrishalaMahesh/linux/blob/master/screenshots/3c.png)
![Assignment 3](https://github.com/TrishalaMahesh/linux/blob/master/screenshots/3d.png)
![Assignment 3](https://github.com/TrishalaMahesh/linux/blob/master/screenshots/3e.png)
![Assignment 3](https://github.com/TrishalaMahesh/linux/blob/master/screenshots/3f.png)
 
 
 Below are the list of the most frequent exits:
 1. Exit type 1 External Interruot
 2. Exit type 10 CPUID
 3. Exit type 30 I/O instruction
 4. Exit type 48 = EPT violation
 
 Below are the lists of the least frequent exits:
  1. Exit type 0- Exception or non-maskable interrupt
  2. Exit type 7-Interrupt window
  3. Exit type 28-Control-register access
  4. Exit type 49-EPT misconfiguration

## Assignment 4: Nested Paging vs. Shadow Paging
### Trishala Contributions:
 - Ran  Assignment 3 code and booted a inner test VM using that code
 - Recorded total exit count information for each type of exit handled by KVM
 - Above step was completed sequence of queries of CPUID leaf function 0x4FFFFFFD
### Suhas Contributions:
 - Shutdown the inner test VM and removed kvm-intel module from running kernel
 - Reloaded kvm-intel module from the latest kernel version's lib path with ept=0 flag on 
 - Booted the same inner test vm and recorded total exit count information for each type of exit handled by KVM



## Assignment 4 steps:
1.Run Assignment 3 code and boot inner test vm.
2.Once the inner VM boots,run the following command in inner vm terminal to get the total count for each type of exit handled by KVM) 
using a series of queries of CPUID leaf function 0x4FFFFFFD.
```
cpuid -l 0x4ffffffd -s{exit_type}
```
3. Run ```dmesg``` on outer VM to read the counts and verify.
4. Turn off inner VM
5. Run below command to remove  ‘kvm-intel’ module from your running kernel
```
sudo rmmod kvm-intel
```

6.Relaod the kvm_intel module from the lib path of your current kernel version 
```
nsmod  /lib/modules/5.18.0-rc3+/kernel/arch/x86/kvm/kvm-intel.ko ept=0
```
7.Boot the same inner test vm again, and record the total exit count information (total count for each type of exit handled by KVM)
using a series of queries of CPUID leaf function 0x4FFFFFFD.
```
cpuid -l 0x4ffffffd -s{exit_type}
```
8. Run ```dmesg``` on outer VM to read the counts and verify.

Sample output screenshots

- Without EPT
![Assignment 4](https://github.com/TrishalaMahesh/linux/blob/master/screenshots/4a.png)
![Assignment 4](https://github.com/TrishalaMahesh/linux/blob/master/screenshots/4b.png)
![Assignment 4](https://github.com/TrishalaMahesh/linux/blob/master/screenshots/4c.png)
![Assignment 4](https://github.com/TrishalaMahesh/linux/blob/master/screenshots/4d.png)
![Assignment 4](https://github.com/TrishalaMahesh/linux/blob/master/screenshots/4e.png)
![Assignment 4](https://github.com/TrishalaMahesh/linux/blob/master/screenshots/4f.png)

- With EPT
![Assignment 4](https://github.com/TrishalaMahesh/linux/blob/master/screenshots/e1.png)
![Assignment 4](https://github.com/TrishalaMahesh/linux/blob/master/screenshots/e2.png)
![Assignment 4](https://github.com/TrishalaMahesh/linux/blob/master/screenshots/e3.png)
![Assignment 4](https://github.com/TrishalaMahesh/linux/blob/master/screenshots/e4.png)
![Assignment 4](https://github.com/TrishalaMahesh/linux/blob/master/screenshots/e5.png)
![Assignment 4](https://github.com/TrishalaMahesh/linux/blob/master/screenshots/e6.png)

- Exclusive Exits only with EPT
![Assignment 4](https://github.com/TrishalaMahesh/linux/blob/master/screenshots/e11.png)
![Assignment 4](https://github.com/TrishalaMahesh/linux/blob/master/screenshots/e12.png)


### Answers about Assignment 4:
1. What did you learn from the count of exits? Was the count what you expected? If not, why not?
- Exit count is more for shadow paging compared to nested paging since the VMM performs more work in case of shadow paging.
- Few of the Exit types that occur around 6 times more than that of nested paging are as follows:
    - Exit 0 : Exception or NMI
    - Exit 1 : External interrupt
    - Exit 7 : Interrupt window.
    - Exit 12 : HLT
    - Exit 28 : Control-register accesses.
    - Exit 32 : WRMSR
- Few type of exits that occured exclusively in shadow paging
 - Exit 33 : VM-entry failure due to invalid guest state
 - Exit 14 : INVLPG
 
3. What changed between the two runs (ept vs no-ept)?
- During Shadow paging i.e.ept=0 , VM performs more TLB flushes, page faults etc. and so their are more exits comapred to Nested paging ept=1.
-  These exits include exits on %cr3 read and write, exits on page faults occuring in shadow page table and guest page table, exits on TLB flushes to remove stale entries when there is a free. This is the reason for increase in the number of exits.
