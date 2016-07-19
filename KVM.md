## KVM  
_________________________________________________________________________________________
## List VM's
[virt-tools.org](http://virt-tools.org/learning/start-list-with-command-line/)  
`list --all`  

## Create disk img  
[raymii.org](https://raymii.org/s/tutorials/KVM_add_disk_image_or_swap_image_to_virtual_machine_with_virsh.html#Create_and_attach_the_disk_image)  
`qemu-img create -f raw example-vm-swap.img 1G`  

## Disk Resize  
[serverfault](http://serverfault.com/questions/324281/how-do-you-increase-a-kvm-guests-disk-space)  
`qemu-img resize vmdisk.img +10G`

## Mount CD media
[ndchost.com](https://www.ndchost.com/wiki/libvirt/change-media)  
`virsh # change-media vm1 hdc /var/lib/libvirt/images/cd2.iso --insert`

## Ejecting the existing media
[ndchost.com](https://www.ndchost.com/wiki/libvirt/change-media)  
```
virsh # domblklist vm1
Target     Source
------------------------------------------------
hda        /dev/storage1/vm1_disk1
hdc        /var/lib/libvirt/images/cd1.iso
```
`virsh # change-media vm1 hdc --eject`
