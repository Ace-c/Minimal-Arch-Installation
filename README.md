
Follow these simple steps and you'll have your right to say `I use Arch, btw` 

**1. Create a bootable Arch Linux USB drive**
> [!NOTE]
> You can use ventoy, etcher, rufus. If you're using rufus, use GPT style

**2. Make sure you have UEFI mode enabled in BIOS**

**3. Boot through your USB**


&nbsp;

## Setup Internet -
> [!NOTE]
> If you're using Ethernet, skip this part & go to partionting step
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

Make 3 partition, Let's suppose you have 100GB of Space then, you can make following parititon style

- 1G for `/boot` parition (1G for multiple kernel) `/dev/sda2`
- 18G for `swap` space `/dev/sda3` (For 16GB RAM)
 
> [!NOTE]
> To calculate how much swap is needed, just whatever you RAM size is +2GB, Swap is essential for hibernation feature.

- Rest size for root `/` partition `/dev/sda1`


**3. After making partition, Format it**

- For root partition -
```
mkfs.ext4 /dev/sda1
```
-  For boot partition -
```
mkfs.fat -F32 /dev/sda2
```
-  For swap Partition -
```
mkswap /dev/sda3
```

-   Now, update your pacman repository
    
```
pacman -Syy
```

-   Mounting Root and Swap partition System -

1.  Mount Root
    
```
mount /dev/sda1 /mnt
```

-   Install Base pacakges
    
```
pacstrap -K /mnt base base-devel linux-lts linux-firmware amd-ucode sudo nano vi
```

Note - You can use "intel-ucode" for Intel baed system



-   Configure the File system
    

```
genfstab -U /mnt >> /mnt/etc/fstab
```

```
arch-chroot /mnt
```

-   Setting Timezone
    
```
ln -sf /usr/share/zoneinfo/Asia/Kolkata /etc/localtime
```
> [!NOTE]
> You have to set timezone according to your location

```
hwclock --systohc
```

-   Localization
    
```
nano /etc/locale.gen
```

1.  Uncomment the below line -
    
> #en_IN.UTF-8

Note - Set this according to your region

1.  save and exit
    

-   Generate Locale -
    

```
locale-gen
```

-   Add Language to locale.conf
    

```
echo "LANG=en_IN.UTF-8" > /etc/locale.conf
```

-   Set Hostname
    

```
echo arch > /etc/hostname
```

> set hosts -

```
nano /etc/hosts
```

> Add these line & save it, use "TAB" for spacing

```
127.0.0.1    localhost
::1          localhost
127.0.1.1    arch.localdomain  arch
```

-   Install and Enable Network Manager
    

```
pacman -S networkmanager
systemctl enable NetworkManager
```

-   Set Root Password
    

> use cmnd "passwd" and set password for your hostname

-   Add user -- Your user name(eg. ayu)
    

```
useradd -m -G wheel ayu
```

-   Set pass for user

```
passwd ayu(put your user name)
```

-   Give sudo (superuser) permission for user
    

```
EDITOR=nano visudo
```

> Uncomment the following line -

```
#  %wheel ALL=(ALL:ALL)  ALL 
```

> save and exit

-   Install GRUB Bootloader ---
    

```
pacman -S grub efibootmgr
```

> Create directory where EFI partition will be mounted

```
mkdir /boot/efi
```

-   Now mount EFI partition you had created before (/dev/sda1)
    

```
mount /dev/sda2 /boot/efi
```

-   Install GRUB like this --
    

```
grub-install --target=x86_64-efi --bootloader-id=ARCH --efi-directory=/boot/efi
```

> you can replace id "ARCH" with your own custom name

```
grub-mkconfig -o /boot/grub/grub.cfg
```


-   Exit the fakeroot environment and reboot the sytem
    

```
exit
umount -l /mnt
shutdown now
```
