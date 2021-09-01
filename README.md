# WSL2 Linux repo configuration

## Overview
Descriptions and examples are taken from various resources (see [Acknowledgements](#acknowledgements)).
- [Requirements](#requirements)

WSL2:
- [Installing WSL2](#installing-wsl2)
- [Useful commands](#useful-commands)
- [Importing image](#importing-image)
- [Adding first user](#adding-first-user)
- [Setting startup exports](#setting-startup-exports)

Windows:
- [GUI server](#gui-server)
- [Audio server](#audio-server)

## Requirements  
* Windows 10 ver 2004 (It also works on Home edition)

## WSL2
Linux distribution can be downloaded from MS Store or from https://cloud-images.ubuntu.com/releases/ (files containing `amd64-wsl`).

### Installing WSL2
Open PowerShell as Administrator and use commands in this order.
Enable WSL:
```
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```
Enable "Virtual Machine Platform":
```
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```
Setp WSL2 as default
```
wsl --set-default-version 2
```

### Useful commands
List all distros:
```
wsl -l -v
```
Set image as default:
```
wsl --setdefault <DistributionName>
```
Run specific distro:
```
wsl -d <DistributionName>
```
Update version of distro
```
wsl --set-version <DistributionName> 2
```

### Importing image
**NOTE**: This step can be skipped if image downloaded from MS Store
From any directory:
```
wsl.exe --import <Distribution Name> <Install Folder> <.TAR.GZ File Path>
```
* _Distribution Name_ - name visible in wsl list and used as argument when running specific image
* _Install Folder_ - where wsl image will be placed

### Adding first user
Run commands in WSL.
Add user:
```
adduser username
adduser username sudo
```
Set default user:
```
tee /etc/wsl.conf <<_EOF
[user]
default=${NEW_USER}
_EOF
```
To changes take effect exit WSL via `logout` and restart WSL.

### Setting startup exports
Move to `/etc/profile.d` and create file `custom.sh` using root.
Paste code into the file:
```
# Get IP address of Windows host machine
IP_ADDRESS=$(grep nameserver /etc/resolv.conf | awk '{print $2}')

# GUI
export DISPLAY=$IP_ADDRESS:0.0
# To run Qt apps using QML please comment it
export LIBGL_ALWAYS_INDIRECT=1

# Audio
export PULSE_SERVER=tcp:$IP_ADDRESS;

# This is specific to WSL 2. If the WSL 2 VM goes rogue and decides not to free
# up memory, this command will free your memory after about 20-30 seconds.
#   Details: https://github.com/microsoft/WSL/issues/4166#issuecomment-628493643
alias drop_cache="sudo sh -c \"echo 3 >'/proc/sys/vm/drop_caches' && swapoff -a && swapon -a && printf '\n%s\n' 'Ram-cache and Swap Cleared'\""
```


## Windows

### GUI server
Download and install X server for Windows:  [VcXSrv](https://sourceforge.net/projects/vcxsrv/).

**Running VcXSrv:**

_Page 1_:
- Multiple windows
- Display number: 0

_Page 2_:
- Start no client

_Page 3_:
- _Native opengl_ - To get Qt / QML apps working please **uncheck** option and unset _LIBGL_ALWAYS_INDIRECT_ in Linux.
- _Disable access control_ - **checked**

Windows Security will ask you to allow access for X server. Check both private and public.

In case you forgot it can be updated in **Windows Defender Firewall** -> **Advanced settings** -> **Inbound Rules**.
1. Search for **NameOfYourXServer**
2. Set both private and public connections by setting **Allow the connection**

### Audio server
1. Download [Pulse Server](https://www.freedesktop.org/wiki/Software/PulseAudio/Ports/Windows/Support/) for Windows.
2. Unzip it somewhere on disk
3. Open `etc\pulse\default.pa`
4. Change
```
module module-waveout sink_name=output source_name=input
```
into
```
load-module module-waveout sink_name=output source_name=input record=0
```
5. Change
```
#load-module module-native-protocol-tcp
```
into
```
load-module module-native-protocol-tcp auth-ip-acl=127.0.0.1;172.16.0.0/12
```
5. Open `etc\pulse\daemon.conf`
6. Change
```
; exit-idle-time = 20
```
into
```
exit-idle-time = -1
```

## Acknowledgements
* [Importing image](https://superuser.com/questions/1515246/how-to-add-second-wsl2-ubuntu-distro-fresh-install)
* [Setting up WSL2 GUI](https://stackoverflow.com/questions/61110603/how-to-set-up-working-x11-forwarding-on-wsl2)
* [Clearing memory cache](https://www.youtube.com/watch?v=4PwClrUCqJM&t=3s)
* [Pulse audio setup](https://discourse.ubuntu.com/t/getting-sound-to-work-on-wsl2/11869)

## Author 
Emil Sawicki

## License
MIT
