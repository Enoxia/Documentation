# Inspecting .ova files
An `OVA` file contains a compressed version of a virtual machine. When you open an OVA file, the virtual machine is extracted and imported into the virtualization software installed on your computer.
We have 2 choices ; we can either mount the `.ova` in `Virtual box` or `VMWare` to start it and access the VM, or we can extract the `.ova` to inspect the VM's file system (a `.vmdk` file) directly on our machine. 

We can `extract` a `.ova` file using tar :
```bash
tar -xf file.ova
```

It will give us several files, including the `.vmdk`, which is the `virtual disk` of the virtual machine.

We can then inspect this virtual disk by mounting it on our machine in a created folder : 
```bash
mkdir /mnt/VM
sudo guestmount -a nom_du_disque.vmdk -i --ro /mnt/VM
```

When finished, don't forget to unmount the VM's disk 
```bash
sudo guestunmount /mnt/VM
```

