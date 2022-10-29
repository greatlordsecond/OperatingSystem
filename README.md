# Operating-System-Configuratuion

## Arch Linux SBG for ASUS-ROG-Zephyrus-G15-GA503QR_GA503QR
**0. System Specifications**

* CPU: AMD Ryzen 9 5900HS with Radeon Graphics (16) @ 4.680GHz
* Memory: 39584MiB
* Storage: 1.5TB
* Integrated GPU: AMD ATI 07:00.0 Cezanne 
* Dedicated GPU: NVIDIA GeForce RTX 3070 Mobile / Max-Q
* Resolution: 2560x1440 

**1. Download Latest Arch x86_64.iso**

https://archlinux.org/download/


**2. Create a bootable usb**
* In windows make use of https://rufus.ie/en/ 
* In linux use a dd command

```
dd if=archlinux-version-x86_64.iso of=/dev/sdb bs=4M
```
**Note**
Only issue which might appear is nouveau crashing the installation, this can be solved by adding a boot parameter **modprobe.blacklist=nouveau** to the kernel cmdline before booting the installation media

**3. Connect to Internet**

```
iwctl
```
```
device list
```
```
station wlan0 connect Zenos_5g
```

<details><summary>Troubleshoot</summary>

```
rfkill unblock all
```
</details>

**4. Ping some site on the Internet to verify connection**
```
ping -c 3 archlinux.org
```

**5. Update system clock**
```
timedatectl set-ntp true
```


<details><summary>Check status</summary>

```
timedatectl status
```
</details>

**6. Update your mirrorlist**
```
reflector --verbose --latest 200 --sort rate --save /etc/pacman.d/mirrorlist
```

**7. Install terminus-font and archlinux-keyring**
```
pacman -Syy terminus-font archlinux-keyring
```

<details><summary>setfont</summary>

```
setfont ter-v24b
```
</details>

**8. Create the encrypted filesystems**

* boot EFI Drive Setup
```
gdisk /dev/nvme1n1
```
- Type 'o' to create a partition table
- Type 'n' for a new partition
- Enter
- Enter
- +1G
- EF00 This will create a 1 gb partition with the EFI format that we need for booting

* swap Drive setup
- Type 'n' to create a new partition
- Enter
- Enter
- +40G
- 8200

* root Drive setup
- Type 'n' to create a new partition
- Enter
- Enter
- +128G
- 8300

* Check if everything looks right by pressing ‘p’. It should look like this:
```
   Number    Start (sector)  End (sector)   Size          Code    Name
   1         2048            2099199        1024.0 MiB    EF00    EFI system partition
   2         2099200         85985279       40.0 GiB      8200    Linux swap
   3         85985280        2000408575     912.9 GiB     8300    Linux filesystem
```

* Looks good?
Press 'w' to write changes to disk.

* Create filesystems
```
mkfs.fat -F32 /dev/nvme1n1p1
mkswap /dev/nvme1n1p2
mkfs.ext4 /dev/nvme1n1p3
```

* Mount partitions
```
mount /dev/nvme1n1p3 /mnt
```
```
mkdir /mnt/boot
```
```
mount /dev/nvme1n1p1 /mnt/boot
```
```
swapon /dev/nvme1n1p2
```

**9) Install Arch linux base and vim packages**
```
pacstrap -i /mnt base vim
```

**10) Generate the /etc/fstab file**
```
genfstab -U -p /mnt >> /mnt/etc/fstab
```

**11) Copy systemd network configuration files**
```
cp /etc/systemd/network/* /mnt/etc/systemd/network
```

**12) Change root to new system**
```
arch-chroot /mnt
```

**13) get the repo add to your /etc/pacman.conf at the end**
```
vim /etc/pacman.conf
```
```
[g14]
SigLevel = DatabaseNever Optional TrustAll
Server = https://arch.asus-linux.org
```

**14) Installing all the required packages**
```
pacman -Sy linux-g14 linux-firmware linux-g14-headers base-devel efibootmgr mtools \
dosfstools openssh iwd zsh ntfs-3g amd-ucode xf86-video-amdgpu git ark unrar sshfs \
networkmanager asusctl nvidia-dkms supergfxctl gdm gnome gnome-tweaks firefox \
deepin-icon-theme deepin-gtk-theme virtualgl vlc pulseaudio-bluetooth gimp kdenlive\
qbittorrent obs-studio yt-dlp gparted
```

**15) Set the timezone**
```
ln -sf /usr/share/zoneinfo/Asia/Kolkata /etc/localtime
```

**16) Set locale**
```
sed -i 's/#en_US.UTF-8/en_US.UTF-8/g' /etc/locale.gen (uncomment en_US.UTF-8)
```
```
locale-gen
```

**17) Create locale.conf**
```
vim /etc/locale.conf
```
Add the below line with your locale info
```
LANG=en_US.UTF-8
```

**18) Set your hostname**
```
vim /etc/hostname
```
Add something like below line
```
ai
```

**19) Set your hosts**
```
vim /etc/hosts
```
Add the below lines by making required changes
```
127.0.0.1   localhost
::1         localhost
127.0.1.1   ai.localdomain   ai
```

**20) Configure mkinitcpio**
```
vim /etc/mkinitcpio.conf
```
Update "HOOKS=(base udev autodetect modconf block filesystems keyboard fsck)" to
```
HOOKS=(base udev autodetect keyboard keymap consolefont modconf block filesystems resume fsck)
```

**21) Generate initramfs & install bootctl**
```
mkinitcpio -p linux-g14
```
```
bootctl install
```

**22) Create linux boot entry**
```
vim /boot/loader/entries/arch.conf
```
Add the below lines
```
title arch
linux /vmlinuz-linux-g14
initrd /amd-ucode.img
initrd /initramfs-linux-g14.img
options resume=/dev/nvme1n1p2 root=/dev/nvme1n1p3 rw pci=noaer quiet splash
```

**23) Enable boot menu item for windows**
```
lsblk
```
You will see something like 
```
    NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
    nvme1n1     259:0    0 953.9G  0 disk 
    ...
    nvme0n1     259:4    0 465.8G  0 disk 
    -> nvme0n1p1 259:5    0   100M  0 part 
       nvme0n1p2 259:6    0    16M  0 part 
    ...
```
```
mount /dev/nvme0n1p1 /mnt
```
```
cp -R /mnt/EFI/Boot /boot/EFI/
```
```
cp -R /mnt/EFI/Microsoft /boot/EFI/
```
```
umount /mnt
```

**24) configure bootloader, Set root password and create a user****
```
vim /boot/loader/loader.conf
```
Add below mentioned lines, save & close file
```
timeout 10
default arch 
```

```
passwd
```
```
useradd -m -g wheel rama
```
```
passwd rama
```
```
EDITOR=vim visudo
```
uncomment below line
```
%wheel ALL=(ALL) ALL
```

**25) Enable required services**
```
systemctl enable systemd-networkd
systemctl enable systemd-resolved
systemctl enable systemd-timesyncd
systemctl enable NetworkManager
systemctl enable bluetooth
systemctl enable power-profiles-daemon
systemctl enable supergfxd
systemctl enable gdm
```
**26) Finish installation**
```
exit
umount -a
reboot
```
login as rama and change SHELL to zsh and configure as required
```
chsh -s /bin/zsh
```
Add below lines to ~/.zshrc
```
PROMPT="%F{cyan}%n%f %F{white}%1~%f %F{yellow}$%f "
RPROMPT="%F{241}%T%f"
```

**27) set-ntp & update the Hardware clock**
```
sudo timedatectl set-ntp true
sudo hwclock --systohc
```

**28) Adjust system clock from real time clock**
```
sudo timedatectl set-local-rtc 1 --adjust-system-clock
```

**29) Setup yay (AUR Helper)**
```
cd /opt
```
```
sudo git clone https://aur.archlinux.org/yay-git.git
```
```
sudo chown -R $USER ./yay-git
```
```
cd yay-git
```
```
makepkg -si
```

**30) Setup gnome-browser-connector**
```
cd /opt
```
```
sudo git clone https://aur.archlinux.org/gnome-browser-connector.git
```
```
sudo chown -R $USER ./gnome-browser-connector
```
```
cd gnome-browser-connector/
```
```
makepkg -si
```

**31) Install required fonts**
```
yay -S ttf-comic-mono-git freetype2 ttf-ms-fonts
```
```
sudo pacman -S freetype2 fontconfig cairo ttf-ubuntu-font-family \
noto-fonts noto-fonts-cjk ttf-dejavu \
ttf-liberation ttf-opensans ttf-cascadia-code
```
```
sudo ln -s /etc/fonts/conf.avail/70-no-bitmaps.conf /etc/fonts/conf.d  
```
```
sudo vim /etc/profile.d/freetype2.sh
```
Add/uncomment the following line to/in it, save and close 
```
export FREETYPE_PROPERTIES="truetype:interpreter-version=40"
```
```
mkdir -p ~/.config/fontconfig/conf.d/
```
```
vim ~/.config/fontconfig/conf.d/20-no-embedded.conf
```
Add the following lines to it, save and close 
```
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
<match target="font">
<edit name="embeddedbitmap" mode="assign">
<bool>false</bool>
</edit>
</match>
</fontconfig>
```
```
reboot

```

**32) GPU Status tools**
* nvidia-smi
* glxspheres64

**33) Installing required packages**
```
yay -S visual-studio-code-bin android-studio intellij-idea-community-edition pycharm-community-edition \
onlyoffice-bin libvirt virtualbox virtualbox-ext-vnc virtualbox-guest-iso virtualbox-guest-utils \
virtualbox-host-dkms virtualbox-ext-oracle teamviewer google-chrome brave-bin
```

**34) Supergfxctl - graphics switching**
* integrated, uses the iGPU only and force-disables the dGPU
```
sudo supergfxctl --mode integrated
```
* dedicated, uses the Nvidia gpu only
```
sudo supergfxctl --mode dedicated
```
* hybrid, enables Nvidia prime-offload mode
```
sudo supergfxctl --mode hybrid
```
* compute, enables Nvidia without Xorg. Useful for ML/Cuda
```
sudo supergfxctl --mode compute
```
vfio, binds the Nvidia gpu to vfio for VM pass-through
```
sudo supergfxctl --mode vfio
```
need to relogin after switching

* get current mode
```
sudo supergfxctl -g 
```

**35) gnome extensions**
* ArcMenu
* Caffeine
* Dash to Panel
* Vitals

**36) Machine Learning Setup**
```
sudo pacman -Syu cuda cudnn jupyter-notebook python-tensorflow-cuda
```
To test gpu availability
```
import tensorflow as tf
tf.test.is_gpu_available()
```
