Installation de ArchLinux
===

#####Schéma de partitionnement : 
`lsblk -f`

    NAME   FSTYPE LABEL              UUID                                
    sda                                                                   
    ├─sda1 ntfs   Réservé au système 1E6EC0966EC067DB                     
    ├─sda2 ntfs   WINDOWS            4EB2C2FFB2C2EA93                    
    └─sda3 exfat  DONNEES            75A8-3CEF                            
    sdb                                                                   
    ├─sdb1 ext4                      788a4d5e-9072-4cd9-9f91-71e6fec70b8a
    ├─sdb2 ext4                      afbebf31-ad90-472b-b4e7-d1433f724370
    └─sdb3 ext4                      0d2635ad-6b53-4042-8627-e8ab51f61afb



###### Changement du clavier

    loadkeys fr-pc

###### on cherche nos partitions

    lsblk -f

###### On les supprime

    mkfs.ext4 /dev/sdb1
    mkfs.ext4 /dev/sdb2
    mkfs.ext4 /dev/sdb3

###### On monte ce qui est nécessaire

    mount /dev/sdb2 /mnt
    mkdir /mnt/{boot,home}
    mount /dev/sdb1 /mnt/boot
    mount /dev/sdb3 /mnt/home

###### Récupération du dernier mirrorlist

    wget -O /etc/pacman.d/mirrorlist https://www.archlinux.org/mirrorlist/all

###### On ne garde que les français (ctrl + K pour éditer)

    nano /etc/pacman.d/mirrorlist

###### On sauvegarde

    cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup

###### On teste les différents mirroirs

    rankmirrors -n 6 /etc/pacman.d/mirrorlist.backup > /etc/pacman.d/mirrorlist

###### On met à jour

    pacman -Syy

###### On installe la base

    pacstrap /mnt/ base base-devel

###### Quelques addons

    pacstrap /mnt/ sudo vim zip unzip p7zip bzip2 tar wireless_tools dialog git wget

###### On sauvegarde notre mirroir actuel

    cp /etc/pacman.d/mirrorlist /mnt/etc/pacman.d/mirrorlist

###### Installation de grub

    pacstrap /mnt/ grub-bios

###### Création du fstab

    genfstab -U -p /mnt >> /mnt/etc/fstab

###### On se chroot

    arch-chroot /mnt

###### Changement des locales

    nano /etc/locale.conf   # LANG=fr_FR.UTF-8
    nano /etc/locale.gen    # décommenter la ligne française
    locale-gen              # réinitialiser les locales

###### Changement de la console
    nano /etc/vconsole.conf # KEYMAP=fr-latin9

###### Changement du nom de la machine
    nano /etc/hostname      # ex : kaiArch

###### Heure locale
    ln -sf /usr/share/zoneinfo/Europe/Paris /etc/localtime

###### Grub
    mkinitcpio -p linux                   # regeneration du noyau
    grub-mkconfig -o /boot/grub/grub.cfg  # création du grub.cfg
    lsblk -f                              # on vérifie qu'on installe au bon endroit
    grub-install /dev/sdc                 # on installe grub

###### Ajout des multilibs
    nano /etc/pacman.conf 

###### Ajout du password admin
    passwd

###### Ajout d'un utilisateur
    useradd -g users -G wheel,network,optical,storage,power,disk -m kai
    passwd kai

###### Changement de ses droits
    visudo

#### On bascule en tant qu'utilisateur

###### Ajout de l'interface graphique
    sudo pacman -Syu xorg-server xorg-xinit xorg-xmessage xorg-utils xf86-video-intel xf86-input-synaptics

###### Ajout de quelques paquets en plus
    sudo pacman -S cinnamon cinnamon-screensaver gucharmap libgnomekbd nemo gnome-terminal gedit gnome-control-center fuse-exfat exfat-utils eog zsh gnome-keyring slim ntfs-3g chromium evince axel gnome-tweak-tool xdg-user-dirs

###### On va récupérer nos confs (voir annexes pour les confs)
    lsblk -f
    sudo mount /dev/sda3 /mnt/
    sudo cp /mnt/10-keyboard-layout.conf /etc/X11/xorg.conf.d/
    sudo cp /mnt/50-synaptics.conf /etc/X11/xorg.conf.d/
    cp /etc/skel/.xinitrc .
    nano .xinitrc

###### Installation de Yaourt
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

###### ajout d'un theme
    yaourt faenza 
    yaourt hope-gtk
    yaourt cinnamon-theme-minty-arch

###### Changement du shell
    wget --no-check-certificate https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh -O - | sh
    cp /mnt/.zshrc . # recuperation du shell
    chsh -s $(which zsh) # changement de shell

###### On demonte la partition où ya nos fichiers
    sudo umount /mnt

###### Bumblebee
    sudo pacman -S intel-dri bumblebee nvidia lib32-nvidia-utils
    sudo gpasswd -a kai bumblebee



###### On sort
    exit
    exit

###### On demonte tout
    umount /mnt/boot /mnt/home /mnt

###### On reboot
    reboot

###### Activations (ne peuvent pas se faire en chroot)
    sudo systemctl enable slim
    sudo systemctl enable NetworkManager
    sudo systemctl enable bumblebeed.service

###### reboot final
    reboot
    
#### Configuration Annexe


                                #########################################################################
                                #                                                                       #
                                #                                                                       #
                                #       Thème du terminal  : Solarize                                   #
                                #       Thème de cinnamon  : MintyArch                                  #
                                #       Thème du wallpaper : http://wallbase.cc/wallpaper/2226684       #
                                #                                                                       #
                                #########################################################################
    
###### Desactivation de synaptics sur gnome
- Lancer `dconf-editor`
- Éditer `/org/gnome/settings-daemon/plugins/mouse/`
- Unchecker celle qui sert à rien

###### Activation de KMS au boot (carte Intel)
`sudo gedit /etc/mkinitcpio.conf`
Ajouter la valeur `MODULES="i915"`

`sudo mkinitcpio -p linux`

###### Ajout du Thème solarize pour le terminal 
`git clone git://github.com/sigurdga/gnome-terminal-colors-solarized.git`

`cd gnome-terminal-colors-solarized`

`./set_dark.sh`

###### Changement des fonts pour Chrome
`gedit .fonts.conf`

     <match target="font">
        <edit name="autohint" mode="assign">
          <bool>true</bool>
        </edit>
        <edit name="hinting" mode="assign">
          <bool>true</bool>
        </edit>
        <edit mode="assign" name="hintstyle">
          <const>hintslight</const>
        </edit>
      </match>
###### Xorg Touchpad

    Section "InputClass"
           Identifier "touchpad"
           Driver "synaptics"
           MatchIsTouchpad "on"
                  Option "TapButton1" "1"
                  Option "TapButton2" "2"
                  Option "TapButton3" "3"
                  Option "VertEdgeScroll" "on"
                  Option "VertTwoFingerScroll" "on"
                  Option "HorizEdgeScroll" "on"
                  Option "HorizTwoFingerScroll" "on"
                  Option "CircularScrolling" "on"
                  Option "CircScrollTrigger" "2"
                  Option "EmulateTwoFingerMinZ" "40"
                  Option "EmulateTwoFingerMinW" "8"
                  Option "CoastingSpeed" "0"
     EndSection


###### Keyboard Layout
    Section "InputClass"
        Identifier         "Keyboard Layout"
        MatchIsKeyboard    "yes"
        MatchDevicePath    "/dev/input/event*"
        Option             "XkbLayout"  "fr"
        Option             "XkbVariant" "latin9"
    EndSection
    
###### zshrc
    # Path to your oh-my-zsh configuration.
    ZSH=$HOME/.oh-my-zsh
    
    # Set name of the theme to load.
    # Look in ~/.oh-my-zsh/themes/
    # Optionally, if you set this to "random", it'll load a random theme each
    # time that oh-my-zsh is loaded.
    ZSH_THEME="gnzh"
    
    # Example aliases
    # alias zshconfig="mate ~/.zshrc"
    # alias ohmyzsh="mate ~/.oh-my-zsh"
    
    # Set to this to use case-sensitive completion
    # CASE_SENSITIVE="true"
    
    # Comment this out to disable bi-weekly auto-update checks
    # DISABLE_AUTO_UPDATE="true"
    
    # Uncomment to change how many often would you like to wait before auto-updates occur? (in days)
    # export UPDATE_ZSH_DAYS=13
    
    # Uncomment following line if you want to disable colors in ls
    # DISABLE_LS_COLORS="true"
    
    # Uncomment following line if you want to disable autosetting terminal title.
    # DISABLE_AUTO_TITLE="true"
    
    # Uncomment following line if you want red dots to be displayed while waiting for completion
    # COMPLETION_WAITING_DOTS="true"
    
    # Which plugins would you like to load? (plugins can be found in ~/.oh-my-zsh/plugins/*)
    # Custom plugins may be added to ~/.oh-my-zsh/custom/plugins/
    # Example format: plugins=(rails git textmate ruby lighthouse)
    plugins=(git)
    
    source $ZSH/oh-my-zsh.sh
    
    # Customize to your needs...
    export PATH=/usr/local/bin:/usr/bin:/bin:/usr/local/sbin:/usr/sbin:/sbin:/usr/bin/core_perl
    
    
    # modified commands
    alias diff='colordiff'              # requires colordiff package
    alias grep='grep --color=auto'
    alias more='less'
    alias df='df -h'
    alias du='du -c -h'
    alias mkdir='mkdir -p -v'
    alias nano='nano -w'
    alias ..='cd ..'
    
    # new commands
    alias da='date "+%A, %B %d, %Y [%T]"'
    alias du1='du --max-depth=1'
    alias hist='history | grep'      # requires an argument
    alias openports='netstat --all --numeric --programs --inet --inet6'
    alias pg='ps -Af | grep $1'         # requires an argument (note: /usr/bin/pg is installed by the util-linux package; maybe a different alias name should be used)
    
    # privileged access
    if [ $UID -ne 0 ]; then
        alias sudo='sudo '
        alias scat='sudo cat'
        alias svim='sudo vim'
        alias root='sudo su'
        alias reboot='sudo reboot'
        alias halt='sudo halt'
        alias update='sudo pacman -Su'
        alias netcfg='sudo netcfg2'
    fi
    
    # ls
    alias ls='ls -hF --color=auto'
    alias lr='ls -R'                    # recursive ls
    alias ll='ls -l'
    alias la='ll -A'
    alias lx='ll -BX'                   # sort by extension
    alias lz='ll -rS'                   # sort by size
    alias lt='ll -rt'                   # sort by date
    alias lm='la | more'
    
    # safety features
    alias cp='cp -i'
    alias mv='mv -i'
    alias ln='ln -i'
    alias chown='chown --preserve-root'
    alias chmod='chmod --preserve-root'
    alias chgrp='chgrp --preserve-root'
    alias ping-google='ping -c 3 www.google.fr'
    
    # pacman aliases (if necessary, replace 'pacman' with your favorite AUR helper and adapt the commands accordingly)
    alias pac="sudo /usr/bin/pacman -S"      # default action    - install one or more packages
    
    function extract()
    {
         if [ -f $1 ] ; then
             case $1 in
                *.tar.bz2)   
                    tar xvjf $1     
                    ;;
                *.tar.gz)    
                    tar xvzf $1     
                    ;;
                *.bz2)       
                    bunzip2 $1      
                    ;;
                *.rar)
                    unrar x $1      
                    ;;
                *.gz)
                    gunzip $1       
                    ;;
                *.tar)
                    tar xvf $1      
                    ;;
                *.tbz2)
                    tar xvjf $1     
                    ;;
                *.tgz)
                    tar xvzf $1     
                    ;;
                *.zip)
                    unzip $1        
                    ;;
                *.Z)
                    uncompress $1   
                    ;;
                *.7z)
                    7z x $1         
                    ;;
                *)  
                    echo "'$1' cannot be extracted via extract" 
                    ;;
            esac
        else
            echo "'$1' is not a valid file"
        fi
    }
    
    
    ################################################################
    
    alias yanoc="yaourt --noconfirm"
    alias yanocup="yaourt --noconfirm -Syua"
    
    # Pacman alias examples
     alias pacupg='sudo pacman -Syu'        # Synchronize with repositories before upgrading packages that are out of date on the local system.
     alias pacin='sudo pacman -S'           # Install specific package(s) from the repositories
     alias pacins='sudo pacman -U'          # Install specific package not from the repositories but from a file 
     alias pacre='sudo pacman -R'           # Remove the specified package(s), retaining its configuration(s) and required dependencies
     alias pacrem='sudo pacman -Rns'        # Remove the specified package(s), its configuration(s) and unneeded dependencies
     alias pacrep='pacman -Si'              # Display information about a given package in the repositories
     alias pacreps='pacman -Ss'             # Search for package(s) in the repositories
     alias pacloc='pacman -Qi'              # Display information about a given package in the local database
     alias paclocs='pacman -Qs'             # Search for package(s) in the local database
    
     # Additional pacman alias examples
     alias pacupd='sudo pacman -Sy && sudo abs'     # Update and refresh the local package and ABS databases against repositories
     alias pacinsd='sudo pacman -S --asdeps'        # Install given package(s) as dependencies of another package
     alias pacmir='sudo pacman -Syy'                # Force refresh of all package lists after updating /etc/pacman.d/mirrorlist
    
    
    alias axel64='axel -a -n 64'
    alias webserver-start="sudo httpd && sudo systemctl start mysqld"
    alias webserver-restart="sudo systemctl restart httpd && sudo systemctl restart mysqld"
    alias webserver-stop="sudo systemctlstop httpd && sudo systemctl stop mysqld"
    alias nm-switch="sudo systemctl stop wicd.service && sudo systemctl start NetworkManager"
    alias connect-wifi="sudo wpa_supplicant -B -Dwext -i wlp13s0 -c /etc/wpa_supplicant/wpa_supplicant-wlp13s0.conf"

