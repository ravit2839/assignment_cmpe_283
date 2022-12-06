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

#### Naga Venkata Sri Ram Vemparala - 016001596:
* Compiled the code with modifications
* Made changes to fix the errors occurred while compiling
* Created nested VM to run the test program
* Created documentation.

#### Raviteja Kallepalli - 015998853:
* Build Linux kernel and booted nested vm
* Modified the cupid.c and vmx.c modules
* Looked up how to perform testing for kernel code
* Wrote a test program
* Updated documentation

### 2) Describe in detail the steps you used to complete the assignment.

1) Went to https://github.com/torvalds/linux, forked the repo to my account <br />
2) Clone it to VM  <br />
```
 git clone https://github.com/ravit2839/linux.git
  cd linux
  git status
 git remote -v 
```
![git remote -v](https://github.com/kondurunikhil/virtualisation_Ass_2/blob/main/images/git_remote_v.png)
```
cp /boot/config-5.15.0-1025-gcp ~/linux/.config
```

```
4) prepare the linux using make prepare command: <br />
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
9)for Assignment 2 let's modify ~/linux/arch/x86/kvm/vmx/vmx.c  : <br />
```
  extern atomic_t total_exits_counter;
    extern atomic64_t total_cup_cycles_counter;

    static int vmx_handle_exit(struct kvm_vcpu *vcpu, fastpath_t exit_fastpath)
    {
        uint64_t start_time_counter, end_time_counter;
        int ret;
        arch_atomic_inc(&total_exits_counter);
        start_time_counter = rdtsc();
        ret = __vmx_handle_exit(vcpu, exit_fastpath);
        end_time_stamp_counter = rdtsc();
        arch_atomic64_add((end_time_counter - start_time_counter), &total_cup_cycles_counter);
    }
```
10) for A2 requirement modify ~/linux/arch/x86/kvm/cpuid.c :
```
atomic_t total_exits_counter = ATOMIC_INIT(0);
    EXPORT_SYMBOL(total_exits_counter);

    atomic64_t total_cup_cycles_counter = ATOMIC64_INIT(0);
    EXPORT_SYMBOL(total_cup_cycles_counter);

    int kvm_emulate_cpuid(struct kvm_vcpu *vcpu)
    {
        u32 eax, ebx, ecx, edx;

        if (cpuid_fault_enabled(vcpu) && !kvm_require_cpl(vcpu, 0))
            return 1;

        eax = kvm_rax_read(vcpu);
        ecx = kvm_rcx_read(vcpu);

        switch(eax) {
            case 0x4FFFFFFC:
                eax = arch_atomic_read(&total_exits_counter);
                break;
            case 0x4FFFFFFD:
                ebx = (atomic64_read(&total_cup_cycles_counter) >> 32);;
                ecx = (atomic64_read(&total_cup_cycles_counter) & 0xFFFFFFFF);
                //printk(KERN_INFO "### Total CPU Exit Cycle Time(hi) in EBX = %u", ebx);
                //printk(KERN_INFO "### Total CPU Exit Cycle Time(lo) in ECX = %u", ecx);
                break;
```
11) Build and install the modules again, then install the kernel  <br />
```
     make -j 8 modules
     sudo make -j 8 INSTALL_MOD_STRIP=1 modules_install
     sudo make -j 4 install 
```
![install](!!!)
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
![install](!!!)

```
  sudo adduser raviteja_kallepalli libvirt
  sudo getent group | grep libvirt 
```
![getent](!!!!)
```
   sudo virsh list --all
```
![getent](!!!!)

```
   sudo systemctl status libvirtd 
```   
![getent](!!!!)

16) As everything is functioning properly, the output returns an active (running) status.
Use virt-manager to create  inner VM along with the GUI :
```
cd ~
wget https://releases.ubuntu.com/jammy/ubuntu-22.04.1-desktop-amd64.iso
sudo mv ~/ubuntu-22.04.1-desktop-amd64.iso /var/lib/libvirt/images/
sudo virt-manager
```   
![getent](https://github.com/kondurunikhil/virtualisation_Ass_2/blob/main/images/virt-manager.png)
17) choose create a new virtual machine  and select the os downloaded ubuntu 22.04.1 
![innervm](https://github.com/kondurunikhil/virtualisation_Ass_2/blob/main/images/inner-vm.png)

18) we can perform our A2 exit handling testing on the newly installed inner-VM:
![output](https://github.com/kondurunikhil/virtualisation_Ass_2/blob/main/images/out_put.png)
