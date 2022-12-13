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
