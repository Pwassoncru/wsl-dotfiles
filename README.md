# How to install a clean i3 configuration on WSL (ubuntu 18.04)

The scripts used  are all inspired from [Wyene's github](https://github.com/Xyene/wsl-dotfiles.git). I suggest you read his README first. This repo is a tutorial on how to install every tool needed to build a clean  i3 environment on WSL. If you are not on a proxy, you can safely ignore every option that passes through the proxy, such as -x for curl.

## Windows binaries

### Ubuntu

First download ubuntu:
[ubuntu1804](https://aka.ms/wsl-ubuntu-1804)

Change the extension of the downloaded file from `.appx` to `.zip`. Unzip the folder.
The executable should be called 'ubuntu1804'. Follow the instructions.

### X server

Download VcXsrv to enable graphical applications. In ubuntu:

    echo 'export DISPLAY=localhost:0' >> ~/.bashrc

## Scripts

### Windows Scripts

You can already write the launching scripts. It won't work properly before the end of the tuto, but it allows you to have logs of your config and see where it fails.

Create a file `ubuntu.vbs` containing the following three lines. You may want to change the paths.

    Set shell = CreateObject("WScript.Shell" )
    shell.Run """C:\Program Files\VcXsrv\vcxsrv.exe"" :0 -screen 0 @1 -nodecoration -wgl"
    shell.Run """C:\Users\Username\Downloads\Ubuntu\ubuntu1804.exe"" -c "". ~/.bashrc && export DISPLAY=localhost:0.0 && i3"""

To launch it more easily, you can also create a file `ubuntu.bat` containing:

    cscript C:\Users\Username\Downloads\Ubuntu\ubuntu.vbs

P.S. You may want to look up on Internet how to pin a bat file to the taskbar

### Ubuntu Scripts

Here are some scripts to setup the tools that will be downloaded.

    git clone https://github.com/Pwassoncru/wsl-dotfiles.git
    cd wsl-dotfiles
    cp -r .config ~/
    cp -r .scripts ~/
    cp .Xresources ~/
    
You may want to modify `~/scripts/walternate` to setup your own desktop background.



## apt

Set up apt proxy.

    sudo echo 'Acquire::http::Proxy "http://proxy_addr:proxy_port";' >> /etc/apt/apt.conf
    sudo apt update
    sudo apt upgrade


## usefull libs

    sudo apt install hsetroot
    sudo apt install compton
    sudo apt install feh
    sudo apt install git
    sudo apt install ssh
    git config --global http.proxy http://proxy_addr:proxy_port
filesystem explorer using vim shortcuts:

    sudo apt install ranger
    sudo apt install x11-xserver-utils
windows switcher, terminal manager, dmenu and ssh launcher:

    sudo apt install rofi

## i3

Here is the real problem. We want to have a clean i3 installation that works with VcXsrv. We will use i3-gaps, allowing us to customize our i3 installation. There are some issues with i3 and i3-gaps on WSL, but there is that commit that is working fine: `fdf5d3`.

### dependencies

    sudo apt install libxcb1-dev libxcb-keysyms1-dev libpango1.0-dev libxcb-util0-dev libxcb-icccm4-dev libyajl-dev libstartup-notification0-dev libxcb-randr0-dev libev-dev libxcb-cursor-dev libxcb-xinerama0-dev libxcb-xkb-dev libxkbcommon-dev libxkbcommon-x11-dev xutils-dev libxcb-shape0-dev autoconf
    git clone https://github.com/Airblader/xcb-util-xrm
    cd xcb-util-xrm
    git submodule update --init
    ./autogen.sh --prefix=/usr
    make
    sudo make install

### i3-gaps
    git clone https://github.com/Airblader/i3
    cd i3
    git checkout fdf5d3aacfdfe89b2ea12d3e1aadc7e46c287896
    sed -i -- 's/default_sanitizers=address/default_sanitizers=/g' configure.ac
    autoreconf --force --install
    mkdir build
    cd build
    ../configure --prefix=/usr --sysconfdir=/etc
    make
    sudo make install
    
Now you should be able to launch the .bat file, and see your background displayed, since i3 read the config file.

## polybar

Polybar is used to customize the status bar.

    sudo apt-get install cmake cmake-data libcairo2-dev libxcb1-dev libxcb-ewmh-dev libxcb-icccm4-dev libxcb-image0-dev libxcb-randr0-dev libxcb-util0-dev libxcb-xkb-dev pkg-config python-xcbgen xcb-proto libxcb-xrm-dev libasound2-dev libmpdclient-dev libiw-dev libcurl4-openssl-dev libpulse-dev libxcb-composite0-dev
    git clone https://github.com/jaagr/polybar.git
    cd polybar && ./build.sh
    
Now, you should see your background and the status bar. You can test it by switching desktops.

## termite

This is the main terminal used by i3 and rofi.

### dependencies
    sudo apt install -y g++ libgtk-3-dev gtk-doc-tools gnutls-bin valac intltool libpcre2-dev libglib3.0-cil-dev libgnutls28-dev libgirepository1.0-dev libxml2-utils gperf build-essential

    git clone https://github.com/thestinger/vte-ng.git
    echo export LIBRARY_PATH="/usr/include/gtk-3.0:$LIBRARY_PATH" # May seems useless, but for some reasons it fixed the gtk path
    cd vte-ng && ./autogen.sh && make && sudo make install

### Termite

    git clone --recursive https://github.com/thestinger/termite.git
    cd termite && make && sudo make install
    sudo ldconfig
    sudo mkdir -p /lib/terminfo/x
    sudo ln -s /usr/local/share/terminfo/x/xterm-termite /lib/terminfo/x/xterm-termite
    
i3 should now be working perfectly.

## npm

Useful to download some applications, such as gtop.

### nodejs package manager

    curl -x http://proxy_addr:proxy_port -sL https://deb.nodesource.com/setup_10.x -o nodejs_script.sh
    vim nodejs_script.sh
    sed -i -- 's/curl -sL/curl -x http:\/\/proxy_addr:proxy_port -sL/g' nodejs_script.sh
    sudo bash nodejs_script.sh
    sudo apt install nodejs

test:

    nodejs -v
    npm -v

Set proxy:

    sudo npm config set proxy http://proxy_addr:proxy_port

### gtop

    sudo npm install gtop -g

## How it works

You have now a i3 environment. Shortcuts are the same as a classic i3, with `Alt` as the mod key.

`Alt+g`: opens the ssh launcher

`Alt+t`: opens the file browser

`Alt+enter`: opens a terminal

`Alt+p`: opens gtop

`Alt+o`: opens htop

 You can read `~/.config/i3/config` for more detailed shortcuts.
