archlinux-starter
=================

# Changement du clavier
loadkeys fr-pc

# on cherche nos partitions
lsblk -f

# On les supprime
mkfs.ext4 /dev/sdb1
mkfs.ext4 /dev/sdb2
mkfs.ext4 /dev/sdb3

# On monte ce qui est nécessaire
mount /dev/sdb2 /mnt
mkdir /mnt/{boot,home}
mount /dev/sdb1 /mnt/boot
mount /dev/sdb3 /mnt/home

# Récupération du dernier mirrorlist
wget -O /etc/pacman.d/mirrorlist https://www.archlinux.org/mirrorlist/all

# On ne garde que les français (ctrl + K pour éditer)
nano /etc/pacman.d/mirrorlist

# On sauvegarde
cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup

# On teste les différents mirroirs
rankmirrors -n 6 /etc/pacman.d/mirrorlist.backup > /etc/pacman.d/mirrorlist

# On met à jour
pacman -Syy

# On installe la base
pacstrap /mnt/ base base-devel

# Quelques addons
pacstrap /mnt/ sudo vim zip unzip p7zip bzip2 tar wireless_tools dialog git wget

# On sauvegarde notre mirroir actuel
cp /etc/pacman.d/mirrorlist /mnt/etc/pacman.d/mirrorlist

# Installation de grub
pacstrap /mnt/ grub-bios

# Création du fstab
genfstab -U -p /mnt >> /mnt/etc/fstab

# On se chroot
arch-chroot /mnt

# Changement des locales
nano /etc/locale.conf  # LANG=fr_FR.UTF-8
nano /etc/locale.gen   # décommenter la ligne française
locale-gen             # réinitialiser les locales

# Changement de la console
nano /etc/vconsole.conf # KEYMAP=fr-latin9

# Changement du nom de la machine
nano /etc/hostname  # ex : kaiArch

# Heure locale
ln -sf /usr/share/zoneinfo/Europe/Paris /etc/localtime

#### Grub
mkinitcpio -p linux # regeneration du noyau
grub-mkconfig -o /boot/grub/grub.cfg  # création du grub.cfg
lsblk -f # on vérifie qu'on installe au bon endroit
grub-install /dev/sdc

# Ajout des multilibs
nano /etc/pacman.conf 

# Ajout du password admin
passwd

# Ajout d'un utilisateur
useradd -g users -G wheel,network,optical,storage,power,disk -m kai
passwd kai

# Changement de ses droits
visudo

##########################################
##
## on bascule en tant qu'utilisateur
##
##########################################


# Ajout de l'interface graphique
sudo pacman -Syu xorg-server xorg-xinit xorg-xmessage xorg-utils xf86-video-intel

# Ajout de quelques paquets en plus
sudo pacman -S cinnamon cinnamon-screensaver gucharmap libgnomekbd nemo gnome-terminal gedit gnome-control-center fuse-exfat exfat-utils eog zsh gnome-keyring slim ntfs-3g chromium evince axel gnome-tweak-tool xdg-user-dirs

# On va récupérer nos confs
lsblk -f
sudo mount /dev/sda3 /mnt/
sudo cp /mnt/10-keyboard-layout.conf /etc/X11/xorg.conf.d/
cp /etc/skel/.xinitrc .
nano .xinitrc

# Installation de Yaourt
curl -O https://aur.archlinux.org/packages/pa/package-query/package-query.tar.gz
tar zxvf package-query.tar.gz
cd package-query
makepkg -si
cd ..
curl -O https://aur.archlinux.org/packages/ya/yaourt/yaourt.tar.gz
tar zxvf yaourt.tar.gz
cd yaourt
makepkg -si
cd ..

# ajout d'un theme
yaourt faenza 
yaourt hope-gtk

# Changement du shell
wget --no-check-certificate https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh -O - | sh
cp /mnt/.zshrc . # recuperation du shell
chsh -s $(which zsh) # changement de shell

# On demonte la partition où ya nos fichiers
sudo umount /mnt

# Bumblebee
sudo pacman -S intel-dri bumblebee nvidia lib32-nvidia-utils
sudo gpasswd -a kai bumblebee



# On sort
exit

# On demonte tout
umount /mnt/boot /mnt/home /mnt

# On reboot
reboot

# Activations (ne peuvent pas se faire en chroot)
sudo systemctl enable slim
sudo systemctl enable NetworkManager
sudo systemctl enable bumblebeed.service


#########################################################################
#                                                                       #
#                                                                       #
#       Thème du terminal  : Solarize                                   #
#       Thème de cinnamon  : MintyArch                                  #
#       Thème du wallpaper : http://wallbase.cc/wallpaper/2226684       #
#                                                                       #
#########################################################################
