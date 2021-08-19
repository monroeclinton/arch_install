# Arch Setup Guide

- Uses Suckless
- Somewhat minimal

## Setup Guide

1.  Enable networking
    If using `iwd`:
    ```
    sudo systemctl enable iwd
    sudo systemctl start iwd
    ```
    Create `/etc/iwd/main.conf`
    ```
    [General]
    EnableNetworkConfiguration=true

    [Network]
    NameResolvingService=systemd
    ```
2.  Modify `/etc/systemd/resolved.conf` and add DNS settings. Then enable DNS
    ```
    sudo systemctl daemon-reload
    sudo systemctl enable systemd-networkd
    sudo systemctl start systemd-networkd
    sudo systemctl enable systemd-resolved
    sudo systemctl start systemd-resolved
    ```
3.  Install X
    ```
    sudo pacman -Syu xorg xorg-xinit
    ```
4.  Install desktop enviroment

    Install dwm:
    ```
    mkdir ~/bin
    cd ~/bin
    git clone https://github.com/monroeclinton/dwm
    cd dwm
    make
    make install
    ```
    Install st
    ```
    sudo pacman -Syu xclip # Needed for copying to clipboard
    cd ~/bin
    git clone https://github.com/monroeclinton/st
    cd st
    make
    make install
    ```
    Install dmenu
    ```
    cd ~/bin
    git clone https://github.com/monroeclinton/dmenu
    cd dmenu
    make
    make install
    ```
    Install dwmblocks
    ```
    cd ~/bin
    git clone https://github.com/monroeclinton/dwmblocks
    cd dwmblocks
    make
    make install
    ```
    Install scroll
    ```
    cd ~/bin
    git clone https://github.com/monroeclinton/scroll
    cd scroll
    make
    make install
    ```
5. Create `~/.xinitrc` and add:
    ```
    exec dwm
    ```
6. Run `startx` on login by adding this to `~/.bash_profile`
    ```
    if [[ -z $DISPLAY ]] && [[ $(tty) = /dev/tty1 ]]; then exec startx; fi
    ```
7. Install fonts:
    ```
    sudo pacman -Syu noto-fonts
    ```
