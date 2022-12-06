# CMPE283 - ASSIGNMENT 2

The assignment is to modify the CPUID emulation code in KVM to report back additional information when special CPUID leaf nodes are requested.

* For CPUID leaf node %eax=0x4FFFFFFC:
  * Return the total number of exits (all types) in %eax

* For CPUID leaf node %eax=0x4FFFFFFD:
  * Return the high 32 bits of the total time spent processing all exits in %ebx 
  * Return the low 32 bits of the total time spent processing all exits in %ecx 
    * %ebx and %ecx return values are measured in processor cycles, across all VCPUs

At a high level, you will need to perform the following:
* Start with your assignment 1 environment 
* Modify the kernel code with the assignment(s) functionality:
  * Determine where to place the measurement code (for exit counts and # cycles)
  * Create new CPUID leaf 0x4FFFFFFD, 0x4FFFFFFC
    * Report back information as described above 
* Create a user-mode program that performs various CPUID instructions required to test your
assignment
  * Pro-tip: This can be achieved on ubuntu by installing the ‘cpuid’ package
  * Run this user mode program in the inner VM
    * There is no need to insmod anything like assignment 1 did
* Verify proper output

## Questions



### 1) Team Members

#### Raviteja Kallepalli - 015998853:
* Build Linux kernel and booted nested vm
* Modified the cupid.c and vmx.c modules
* Looked up how to perform testing for kernel code
* Wrote a test program
* Updated documentation

#### Naga Venkata Sri Ram Vemparala - 016001596:
* Compiled the code with modifications
* Made changes to fix the errors occurred while compiling
* Created nested VM to run the test program
* Created documentation.



### 2) Describe in detail the steps you used to complete the assignment.

1) Went to https://github.com/torvalds/linux, forked the repo to my account <br />
2) Clone it to VM  <br />
```
 git clone https://github.com/ravit2839/linux.git
  cd linux
  git status
 git remote -v 

```


3) prepare the linux using make prepare command: <br />
```
make prepare
```
5) Build all the modules that goes into the linux kernel:<br />
```
make -j 8 modules
```
6)build the linux kernel itself:  <br />
```
make -j 8
```
7) After the modules and kernel is built, we can package them into a format that suitable for booting inside our VM <br />
```
sudo make -j 8 INSTALL_MOD_STRIP=1 modules_install
```
8) Install linux kernal 
```
sudo make -j 8 install
```
11) Build and install the modules again, then install the kernel  <br />
```
     make -j 8 modules
     sudo make -j 8 INSTALL_MOD_STRIP=1 modules_install
     sudo make -j 4 install 
```

12) As there are no errors, we can create a inner VM inside our current VM,
but first we need to install some extra tools for KVM 
```
sudo apt-get install cpu-checker
sudo apt update
sudo apt-get install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils
sudo kvm-ok
sudo reboot
```
13) Now we need to authorize a vm users for the inner VM, and verify our VM functionalities:
```
  uname -a
```
![WhatsApp Image 2022-12-05 at 21 29 18](https://user-images.githubusercontent.com/68690234/205825091-9bebcee4-8944-442e-8708-00e324470529.jpeg)


```
  sudo adduser raviteja_kallepalli libvirt
  sudo getent group | grep libvirt 
```
![WhatsApp Image 2022-12-05 at 21 29 38](https://user-images.githubusercontent.com/68690234/205825331-21d41857-6718-41e4-8a63-b6f63de1c518.jpeg)

```
   sudo virsh list --all
```
![WhatsApp Image 2022-12-05 at 21 29 56](https://user-images.githubusercontent.com/68690234/205825392-ecaef2fe-e911-4e2a-8358-66e0aa4df7b5.jpeg)


```
   sudo systemctl status libvirtd 
```   
![main1](https://user-images.githubusercontent.com/68690234/205826257-622b8fb7-e77b-4b3d-bf9c-09d974575d77.png)

16) As everything is functioning properly, the output returns an active (running) status.
Use virt-manager to create  inner VM along with the GUI :
```
cd ~
wget https://releases.ubuntu.com/jammy/ubuntu-22.04.1-desktop-amd64.iso
sudo mv ~/ubuntu-22.04.1-desktop-amd64.iso /var/lib/libvirt/images/
sudo virt-manager
```   
![WhatsApp Image 2022-12-05 at 21 30 56](https://user-images.githubusercontent.com/68690234/205825684-e49e90b7-3374-4952-b15b-827bc7ecde97.jpeg)

17) choose create a new virtual machine  and select the os downloaded ubuntu 22.04.1 
![main2](https://user-images.githubusercontent.com/68690234/205826497-c7743c32-4349-4d07-9488-ac8d312d0a77.png)

18) we can perform our A2 exit handling testing on the newly installed inner-VM:
![main3](https://user-images.githubusercontent.com/68690234/205827291-3f08781b-30fd-4ca0-b6fe-c0f7eaf3fde3.png)
