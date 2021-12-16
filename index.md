I transfer the ISO file from the flash drive passed around class. Using the ISO file created started the Arch system in VM Workstation. I gave the VM 20 Gb of storage space, 4 Gb of RAM, and 2 processors. I also ran the Arch VM in UEFI mode.

To setup UEFI mode, I went to the library, clicked on settings, went to options, and then under advanced, I changed the firmware type to UEFI.

After setting up the specs, I ran the VM from workstation and did a ping test to see if I was connected to the internet. I wasn’t so I used that following commands to connect. I checked a list of all the network interfaces available to me and then told system to start the DHCP service. After that I ran another ping test and I was all good.

```markdown
ping -c 4 www.google.com
ip address
ip link set up enp0s3
systemctl start dhcpcd.service
ping -c 4 www.google.com
```
Next, I set up the system clock. 
```markdown
timedatectl set-ntp true
```

Then I ran a command to see the current disk layout. The layout showed that sda had 20 Gb of storage and loop0 had a little over 500 Mib.
```markdown
fdisk -1
lsblk
```

Next, I created the partitions for the system. I created three partitions. One of the partitions was given 500 MB and was my EFI partition. The other partition was used 18.5 Gb was used for the Arch OS. The third partition was a swap partition that was given 1Gb. I created a swap partition because I was worried about RAM usage on my laptop. For creating the partitions, I used cfdisk instead of fdisk because I find cfdisk to be easier to use. 
```markdown
cfdisk /dev/sda
```

I created the swap file using the following commands.
```markdown
swap /dev/sda3
swapon /dev/sda3
```

I created the root file system using the following commands.
```markdown
mkfs.ext4 /dev/sda2
```

I created the EFI file system using the following commands. The EFI system partition must contain FAT32 file system.
```markdown
mkfs.fat -F32 /dev/sda1
```

Next, I mounted each partition. The first partition I mounted was the root partition.
```markdown
mount /dev/sda2 /mnt
```

I created a boot directory where I will mount the EFI partition.
```markdown
mkdir /mnt/boot
```

I mounted the EFI partition to that directory.
```markdown
mount /dev/sda1 /mnt/boot
```

I installed the essential packages for my Arch Linux system. This required using pacstrap.
```markdown
pacstrap /mnt base linux linux-firmware
```

I generated an fstab file. This will allow the system to know where to mount the partitions when it boots.
```markdown
genfstab -U /mnt >> /mnt/etc/fstab
```

Since everything should be in order, we can now chroot into it.
```markdown
arch-chroot /mnt
```

Now I set the region that I am currently in. 
```markdown
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
```

Changed root password
```markdown
passwd
```

Installed vim as my text editor.
```markdown
pacman -S vim
```

Set up my language using vim on locale.conf file.
```markdown
vim /etc/locale.conf
```

Next, I started install other needed program in order to complete. I installed OpenSSH for Arch.
```markdown
pacman -Syu openssh
systemctl enable sshd
```

I added in the different user accounts. First, I had to install sudo into my Arch VM.
```markdown
pacman –sync sudo
```

I then started creating the users for my system and setting their passwords.
```markdown
useradd –create-home sal
passwd sal
GraceHopper1906
```

I then added to the users to the wheel group.
```markdown
usermod –append –groups wheel sal
```

I used vim to edit the /etc/sudoers file. I uncommented the line %wheel ALL=(ALL) ALL
```markdown
vim /etc/sudoers
```

I changed the default shell to zsh. To do this, I first installed zsh onto my ArchVM and then changed the path for the users.
```markdown
Pacman -S zsh
chsh -s /usr/bin/zsh
Type in password
```

I allowed the alias command to change L to ‘ls -lah’ and I changed c to ‘clear’. I also needed to save these for future use, so I used vim to edit the .zshrc file and then I added in the following commands. I also added chvim to go to editing the zshrc file.
```markdown
alias l=’ls -lah’
alias c=’clear’
alias chvim=’vim ~/.zshrc’
vim ~/.zshrc
```

I installed a GUI for my Arch VM. I started by installing a graphics driver, then installed the display server Xorg for the desktop environment. After both of those installations, I installed the desktop environment.  For my VM I chose KDE Plasma for my desktop environment. After installing the desktop environment, I also installed the display manager. I used Gnome display manager for this build.
```markdown
lspci | grep -e VGA
pacman -Syyu
pacman -S nividia nividia-utils nvidia-settings
pacman -S xorg xterm xorg-xinit
startx
pacman -S plasma kdeplasma-addons
pacman -S gdm
systemctl enable gdm
systemctl start gdm
```

Next, I installed a sound system, terminal, Firefox, and even LibreOffice because why not.
```markdown
pacman -S pulseaudio pulseaudio-alsa pavucontrol
pacman -S gnome-terminal gnome-system-monitor
pacman -S firefox vlc audacious
pacman -S libreoffice
```

For the video submission I installed yay
```markdown
sudo pacman -S --needed git base-devel
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```
