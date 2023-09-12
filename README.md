
Follow these simple step and you'll have your playboy PC...

-   First make bootable drive of an Arch Linux (use Rufus) format in GPT style
    
-   Then make sure you have UEFI mode enabled in BIOS
    
-   Boot through your USB
    

> Setup Internet for wireless connection, If you're using ethernet skip this part.
> 
> Or, If you have a problem to connecting with ethernet run this
> 
> ```
> systemctl start dhcpcd@enp0s0
> ```

-   For Wireless Connection
    

```
iwctl
```

Inside the `iwctl` prompt, use the `device list` command to list the available Wi-Fi devices:

```
device list
```

Use the `station <device> scan` command to scan for available Wi-Fi networks:

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

-   Check disks and partition
    

```
lsblk
```

-   Make Partition -
    

```
cfdisk
```

-   Make 3 partition, Let's suppose you have 200GB of Space then, you can make following parititon style
    

1.  1G for /boot/efi parition ( 1G for multiple kernal ) - /dev/sda1
    
2.  190G for root partition - /dev/sda2
    
3.  9G for swap space - /dev/sda3
    

-   After making partition, Format it
    

1.  For boot partition -
    

```
mkfs.fat -F32 /dev/sda1
```

1.  For root partition -
    

```
mkfs.ext4 /dev/sda2
```

1.  For swap Partition -
    

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
mount /dev/sda2 /mnt
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

-   Add user -- here (ayu)
    

```
useradd -m -G wheel -S /bin/bash ayu
```

> Set pass for it, use "passwd" and save it

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
mount /dev/sda1 /boot/efi
```

-   Install GRUB like this --
    

```
grub-install --target=x86_64-efi --bootloader-id=ARCH --efi-directory=/boot/efi
```

> you can replace id "ARCH" with your own custom name

```
grub-mkconfig -o /boot/grub/grub.cfg
```

-   Installing some basic tools ---
    

```
pacman -S git make cmake wget curl
```

-   Exit the fakeroot environment and reboot the sytem
    

```
exit
umount -l /mnt
shutdown now
```
