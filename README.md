cloud-images
============

Scripts to generate clean vm images for kernel/cloud dev

## Debian

Base

```elvish
aria2c "https://cloud.debian.org/images/cloud/bullseye/20220328-962/debian-11-genericcloud-amd64-20220328-962.qcow2"
virt-customize -a debian-11-genericcloud-amd64-20220328-962.qcow2 --root-password password:toor --ssh-inject "root:file:/home/rok/.ssh/id_rsa.pub" --run-command 'ssh-keygen -A'
sudo cp debian-11-genericcloud-amd64-20220328-962.qcow2 /var/lib/libvirt/images/debian-11-genericcloud-amd64-20220328-962.qcow2

fn create-vm { |VM| 
    sudo cp /var/lib/libvirt/images/debian-11-genericcloud-amd64-20220328-962.qcow2 /var/lib/libvirt/images/$VM.qcow2
    sudo virt-customize -a /var/lib/libvirt/images/$VM.qcow2 --hostname $VM
    sudo qemu-img resize /var/lib/libvirt/images/$VM.qcow2 20G
    sudo virt-install --name $VM --virt-type kvm --memory 2048 --vcpus 2 --boot hd,menu=on --disk path=/var/lib/libvirt/images/$VM.qcow2,device=disk --graphics none --os-variant debian11 --network default,model=virtio --console pty,target_type=serial --noautoconsole
}

create-vm debian

# install
# apt install qemu-guest-agent vim tmux neovim fish zsh
```

Clone
```elvish
fn virsh.new { |VM|
    sudo virt-clone --original debian --name $VM --auto-clone
    sudo virt-customize -a /var/lib/libvirt/images/$VM.qcow2 --hostname $VM
}
```

Snapshot
```
fn virsh.snapshot { |domain|
    virsh snapshot-create-as --domain $domain --name (date '+%Y%m%dT%H%M%S')
}
```

Remove
```
fn virsh.rm { |domain| 
    sudo virsh snapshot-list --domain $domain --name | xargs virsh snapshot-delete --domain $domain
    sudo virsh undefine --remove-all-storage $domain
}
```


## ubuntu, cloud-init

virt-customize -a focal-server-cloudimg-amd64.img  --root-password password:toor --ssh-inject "root:file:/home/rok/.ssh/id_rsa.pub" --run-command 'echo -n > /etc/machine-id; rm /var/lib/dbus/machine-id; ln -s /etc/machine-id /var/lib/dbus/machine-id'
sudo mv focal-server-cloudimg-amd64.img /var/lib/libvirt/images/

Create base image
```
aria2c "https://cloud-images.ubuntu.com/focal/current/focal-server-cloudimg-amd64.img"
sudo qemu-img create -b /var/lib/libvirt/images/focal-server-cloudimg-amd64.img -f qcow2 -F qcow2 /var/lib/libvirt/images/ubuntu-focal.qcow2 20G
sudo virt-install --name ubuntu-focal --virt-type kvm --memory 2048 --vcpus 2 --boot hd,menu=on --cloud-init user-data.cfg --disk path=/var/lib/libvirt/images/ubuntu-focal.qcow2,device=disk --graphics none --os-variant ubuntu20.04 --network default,model=virtio --console pty,target_type=serial --noautoconsole
```

cloud-init
```
sudo cloud-localds -v --network-config=network.cfg cloud-init.img user-data.cfg
sudo virt-install --name ubuntu-focal --virt-type kvm --memory 4096 --vcpus 2 --boot hd,menu=on --disk path=cloud-init.img,device=cdrom --disk path=/var/lib/libvirt/images/ubuntu-focal.qcow2,device=disk --graphics none --os-type Linux --os-variant ubuntu20.04 --network default,model=virtio --console pty,target_type=serial
sudo virt-install --name ubuntu-focal --virt-type kvm --memory 4096 --vcpus 2 --boot hd,menu=on --cloud-init --disk path=/var/lib/libvirt/images/ubuntu-focal.qcow2,device=disk --graphics none --os-type Linux --os-variant ubuntu20.04 --network default,model=virtio --console pty,target_type=serial
```


sudo virt-customize -a focal-server-cloudimg-amd64.img --root-password password:toor --ssh-inject "root:file:/home/rok/.ssh/id_rsa.pub" --run-command 'ssh-keygen -A'
sudo cp focal-server-cloudimg-amd64.img /var/lib/libvirt/images/

fn create-vm { |VM| 
    sudo cp /var/lib/libvirt/images/focal-server-cloudimg-amd64.img /var/lib/libvirt/images/ubuntu-focal.qcow2
    sudo virt-customize -a /var/lib/libvirt/images/ubuntu-focal.qcow2 --hostname ubuntu-focal
    sudo qemu-img resize /var/lib/libvirt/images/ubuntu-focal.qcow2 20G
    sudo virt-install --name ubuntu-focal --virt-type kvm --memory 2048 --vcpus 2 --boot hd,menu=on --disk path=/var/lib/libvirt/images/ubuntu-focal.qcow2,device=disk --graphics none --os-variant ubuntu20.04 --network default,model=virtio --console pty,target_type=serial --noautoconsole
}

sudo cp focal-server-cloudimg-amd64.img /var/lib/libvirt/images/ubuntu-focal.img
sudo qemu-img resize /var/lib/libvirt/images/ubuntu-focal.img 20G
sudo virt-install --name ubuntu-focal --virt-type kvm --memory 2048 --vcpus 2 --boot hd,menu=on --cloud-init user-data=./user-data,network-config=./network-config --disk path=/var/lib/libvirt/images/ubuntu-focal.img,device=disk --graphics none --os-variant ubuntu20.04 --network default,model=virtio --console pty,target_type=serial --noautoconsole

sudo virt-clone --original ubuntu-focal --name capi-master-1 --auto-clone
sudo virt-customize -a /var/lib/libvirt/images/capi-master-1.img --hostname capi-master-1

sudo virt-clone --original ubuntu-focal --name capi-master-2 --auto-clone
sudo virt-customize -a /var/lib/libvirt/images/capi-master-2.img --hostname capi-master-2

sudo virt-clone --original ubuntu-focal --name capi-master-3 --auto-clone
sudo virt-customize -a /var/lib/libvirt/images/capi-master-3.img --hostname capi-master-3

sudo virt-clone --original ubuntu-focal --name capi-worker-1 --auto-clone
sudo virt-customize -a /var/lib/libvirt/images/capi-worker-1.img --hostname capi-worker-1
