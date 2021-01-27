## **Subscription**
```shell
subscription-manager list --available
subscription-manager attach --pool <some pool id>
```

## **Install Required Packages**
```shell
yum install zlib-devel.i686 zlib-devel.x86_64 redhat-lsb-core.i686 redhat-lsb-core.x86_64 \
libXScrnSaver.x86_64 glibc-devel.i686 glibc.i686 elfutils-libelf-devel.i686 \
elfutils-libelf-devel.x86_64 gcc make kernel-devel libglvnd libglvnd-devel
yum update
```

## **Disable Nouveau Driver**
Edit /etc/modprobe.d/blacklist.conf to include the following lines
```shell
blacklist nouveau
install nouveau /bin/false
```
Run command
```shell
dracut --omit-drivers nouveau -f    
```

Edit /etc/default/grub to include blacklist
```shell
GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=rhel/root rd.lvm.lv=rhel/swap rhgb rd.driver.blacklist=nouveau nouveau.blacklist=1"
```
Rebuild grub.conf
```shell
grub2-mkconfig -o /boot/grub2/grub.cfg   
```
For EFI based system 
```shell
grub2-mkconfig -o /boot/efi/EFI/redhat/grub.cfg
```

Edit /etc/sysconfig/kdump to include blacklist
```shell
KDUMP_COMMANDLINE_APPEND="irqpoll nr_cpus=1 reset_devices cgroup_disable=memory mce=off numa=off udev.children-max=2 panic=10 rootflags=nofail acpi_no_memhotplug transparent_hugepage=never nokaslr novmcoredd hest_disable rd.driver.blacklist=nouveau"
```
Run following commands
```shell
kdumpctl restart   
mkdumprd -f /boot/initramfs-$(uname -r)kdump.img       
```

# **Sign and Load Driver**
openssl req -new -x509 -newkey rsa:2048 -keyout nvidia.key -outform DER -out nvidia.der -nodes -days 36500 -subj "/CN=Graphics Drivers"
mokutil --import nvidia.der
reboot

sh ./NVIDIA-Linux-x86_64-450.57.run --module-signing-secret-key=/root/nvidia.key --module-signing-public-key=/root/nvidia.der

# **Exclude Future Kernel Upgrade**
/etc/yum.conf
exclude=kernel* redhat-release*     
