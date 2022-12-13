# CMPE283 - ASSIGNMENT 3

The assignment is to modify the CPUID emulation code in KVM to report back additional information when special CPUID leaf nodes are requested.

* For CPUID leaf node %eax=0x4FFFFFFE:
  * Return the number of exits for the exit number provided (on input) in %ecx
   * This value should be returned in %eax 
* For CPUID leaf node %eax=0x4FFFFFFF:
  * Return the time spent processing the exit number provided (on input) in %ecx
    * Return the high 32 bits of the total time spent for that exit in %ebx
    * Return the low 32 bits of the total time spent for that exit in %ecx

At a high level, you will need to perform the following:
* Start with your assignment 1 environment 
* Modify the kernel code with the assignment(s) functionality:
  * Determine where to place the measurement code (for exit counts and # cycles)
  * Create new CPUID leaf 0x4FFFFFFE, 0x4FFFFFFF
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
1. Built Linux Kernel.
2. Installed KVM inside the Ubuntu VM.
3. Debugged fixed the errors which occurred while running the code.
4. Made changes to vmx.c

#### Naga Venkata Sri Ram Vemparala - 016001596:
1. Created VM in GCP and configured.  
2. Built Linux Kernel.
3. Install KVM.
4. Wrote the test program and the shell script to execute cpuid command for the exit codes.
5. Made necessary changes to cpuid.c

#### Steps : 
1. We used the existing GCP VM enviornment from last assignment, an Ubuntu 22.04 LTS x86/64. We enabled nested virtualization and Min CPU platform as Intel Cascade Lake. 

```
--enable-nested-virtualization --min-cpu-platform="Intel Cascade Lake"
```

2. We generated private and public SSH keys using 
```
ssh-keygen -t rsa -f ~/.ssh/google_cloud_key -C $USER 
cat /home/$USER/.ssh/google_cloud_key.pub 
```
3. Then, in the GCP VM settings, we copied the generated key to the SSH keys into the Metadata of the VM.  

4. Went to https://github.com/torvalds/linux, forked the repo to my account <br />
5. Clone it to VM  <br />
```
 git clone <Repository Name>
  cd linux
  git status
 git remote -v 
```

![git remote -v]


```
cp /boot/config-5.15.0-1025-gcp ~/linux/.config
```
6. Then we installed gcc, make and flex.
```
sudo apt-get install gcc
sudo apt-get install make
sudo apt-get install flex
```
7. We ran the 
```
cat /proc/cpuinfo
```
8. Installed necessary Tools <br />

```
sudo apt-get install build-essential
sudo apt-get install kernel-package
sudo apt-get install ccache 
sudo apt-get install flex
sudo apt-get install bison
sudo apt-get install libssl-dev
sudo apt-get install libelf-dev
```

to confirm that all the configurations were as needed.  

9. prepared the linux using make prepare command: <br />
```
make prepare
```
10. Build all the modules that goes into the linux kernel:<br />
```
make -j 8 modules
```
11. build the linux kernel itself:  <br />
```
make -j 8
```
12. After the modules and kernel is built, we can package them into a format that suitable for booting inside our VM <br />
```
sudo make -j 8 INSTALL_MOD_STRIP=1 modules_install
```
13. Installed the linux kernel 

```
sudo make -j 8 install
```

14. Based on the requirements, we modified ~/linux/arch/x86/kvm/vmx/vmx.c  : <br />


15. Based on the requirements, we modified ~/linux/arch/x86/kvm/cpuid.c :


16. Build and install the modules again, then install the kernel  <br />

```
     make -j 8 modules
```
![16](https://user-images.githubusercontent.com/68690234/207235603-32d9dac5-759d-425f-890f-a09d12811b9d.png)


```
     sudo make -j 8 INSTALL_MOD_STRIP=1 modules_install
     sudo make -j 4 install 
```

![16-2](https://user-images.githubusercontent.com/68690234/207235638-bfaa8456-838e-473c-9519-cb18036ddb09.png)


17. As there are no errors, we can create a inner VM inside our current VM,
but first we need to install some extra tools for KVM 


```
sudo apt-get install cpu-checker
sudo apt update
sudo apt-get install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils
sudo kvm-ok
sudo reboot
```

18. Now we need to authorize a vm users for the inner VM, and verify our VM functionalities:

```
  uname -a
```

```
  sudo adduser nikhil_konduru libvirt && sudo adduser nikhil_konduru libvirt-qemu
  sudo getent group | grep libvirt 
```


```
   sudo virsh list --all
```



```
   sudo systemctl status libvirtd 
``` 


![18-3](https://user-images.githubusercontent.com/68690234/207236482-c9661a9a-6aaa-4856-b5d2-475b950aaa89.png)


19. As everything is functioning properly, the output returns an active (running) status.
Use virt-manager to create  inner VM along with the GUI :

```
cd ~
wget https://releases.ubuntu.com/jammy/ubuntu-22.04.1-desktop-amd64.iso
sudo mv ~/ubuntu-22.04.1-desktop-amd64.iso /var/lib/libvirt/images/
sudo virt-manager
```   

![19](https://user-images.githubusercontent.com/68690234/207235686-3a4edb5c-6adb-4cd5-b1c7-c67cb66624ae.png)


20. choose create a new virtual machine  and select the os downloaded ubuntu 22.04.1 


21. Performed our exit handling testing on the newly installed inner-VM.


22. Testing the cpuid functionality using the below commands in the two terminals(GCP VM "T1", and nested VM terminal "T2") and 

        Testing the CPUID functionality for '%eax=0x4fffffff'      
        T2: sudo cpuid -l 0x4fffffff
<img width="692" alt="image" src="https://user-images.githubusercontent.com/98585812/206858283-854be588-c7f7-42ff-9e3c-c821402ce401.png">
        T1: sudo dmesg
<img width="693" alt="image" src="https://user-images.githubusercontent.com/98585812/206858301-1ef8d59b-fa94-4791-88e4-53cb7a3cb178.png">

        Testing the CPUID functionality for '%eax=0x4ffffffe'
        T2: sudo cpuid -l 0x4ffffffe
<img width="693" alt="image" src="https://user-images.githubusercontent.com/98585812/206858317-a7931bce-b668-4551-ac2b-403e53c811a4.png">
        T1: sudo dmesg
<img width="668" alt="image" src="https://user-images.githubusercontent.com/98585812/206858323-9025f454-5c92-42de-8be2-cf2ee80729b3.png">

