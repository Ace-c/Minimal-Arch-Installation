
Follow these simple steps and you'll have your right to say `I use Arch, btw` 

**1. Create a bootable Arch Linux USB drive**
> [!NOTE]
> You can use ventoy, etcher, rufus. If you're using rufus, use GPT style

**2. Make sure you have UEFI mode enabled in BIOS**

**3. Boot through your USB**


&nbsp;

## Setup Internet -
> [!NOTE]
> If you're using Ethernet, skip this part & go to [Disk & Partionting](https://github.com/Ace-c/Minimal-Arch-Installation/blob/README.md/README.md#disks-and-partitioning) step
> 
> Or, If you have a problem, connecting with ethernet run this cmd. Interface name could be diff, to check interface, run `ip link`
> ```
> systemctl start dhcpcd@enp0s0
> ```
> 

-   **Setup Wireless Connection**
    

```
iwctl
```

Inside the `iwctl` prompt, use the `device list` command to list the available Wi-Fi devices
```
device list
```

Use the `station <device> scan` command to scan for available Wi-Fi networks
```
iwctl station wlan0 scan
```

After the scan is complete, use the `station <device> get-networks` command to list the available Wi-Fi networks:
```
iwctl station wlan0 get-networks
```

Connect to your network.
```
iwctl -P "PASSPHRASE" station wlan0 connect "NETWORKNAME"
```

Now, check your internet connection using
```
ping -c 3 google.com
```

## Disks and Partitioning

**1. List Devices**
```
lsblk
```
**2. Make partition using cfdisk**
```
cfdisk
```

Make 3 partition, Follow this partition style

- 1G for `/boot` partition (1G for multiple kernel)
- 18G for `swap` partition. To calculate how much swap is needed, Your RAM size(eg, 16G) +2GB
- Rest size for root `/` partition

&nbsp;

**3. After making partition, Format it**
> run `lsblk` check your partition, I'll asume **/** at `/dev/sda1`, **/boot** at `/dev/sda2`, **swap** at `/dev/sda3`

- For root `/` Partition -
```
mkfs.ext4 /dev/sda1
```
-  For boot Partition -
```
mkfs.fat -F32 /dev/sda2
```
-  For swap Partition -
```
mkswap /dev/sda3
```


## Mounting & Installing Base Packages -

**1.  Mount root parition**
```
mount /dev/sda1 /mnt
```
**2. Refresh all package databases**
```
pacman -Syy
```

**3. Install essential packages**
- Install `intel-ucode` for Intel based system
```
pacstrap -K /mnt base base-devel linux linux-firmware amd-ucode sudo nano vi
```



## Configure The System - 

**1. Configure the file system**
```
genfstab -U /mnt >> /mnt/etc/fstab
```

**2. Enter chroot**
```
arch-chroot /mnt
```

**3. Setting Timezone**
```
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
```
> Example: Region- Asia, city- kolkata

```
hwclock --systohc
```

**4. Localization**
```
nano /etc/locale.gen
```
> Uncomment the line `#en_IN.UTF-8`, Set this according to your region. save and exit


**5. Generate Locale**   
```
locale-gen
```

- Add Language to locale.conf(set acc. to your region).
```
echo "LANG=en_IN.UTF-8" > /etc/locale.conf
```

**6. Set Hostname**
```
echo arch > /etc/hostname
```

```
nano /etc/hosts
```

-  Add following contents & save it, use "TAB" for spacing

```
127.0.0.1    localhost
::1          localhost
127.0.1.1    arch.localdomain  arch
```

- Install and Enable Network Manager
```
pacman -S networkmanager
systemctl enable NetworkManager
```

**7. Set User & Password**
    
- set root password 
```
passwd
```

- Add user (eg, ayu) 
```
useradd -m -G wheel ayu
```

-  Set password for the user
```
passwd ayu
```

-  Enable sudo permission to the user
```
EDITOR=nano visudo
```
> Uncomment the following line `#%wheel ALL=(ALL:ALL)  ALL`



## Installing Bootloader & Configuring
    
```
pacman -S grub efibootmgr
```

- Create `efi` directory 
```
mkdir /boot/efi
```

- Now mount `/boot/efi` to `/dev/sda2`
```
mount /dev/sda2 /boot/efi
```

- Install GRUB 
```
grub-install --target=x86_64-efi --bootloader-id=ArchLinux --efi-directory=/boot/efi
```

- Update grub configurtion
```
grub-mkconfig -o /boot/grub/grub.cfg
```


## Exit and reboot the sytem 
    
```
exit
```

```
umount -R /mnt
```

```
reboot
```
