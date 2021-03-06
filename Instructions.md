# ParrotOS-on-WSL2
# Directions

First, thank you to the following sites on guiding me through the ins and outs of this install   
Installing WSL2 (https://www.windowscentral.com/how-install-wsl2-windows-10)  
MS documentation supprint the above(https://docs.microsoft.com/en-us/windows/wsl/install-win10)  
ParrotSec OS guide to install on WSL2(https://community.parrotsec.org/t/parrot-on-wsl2/14898)  
Additional documentation for ParrotOS on WSL2 (https://exploits.run/parrot-wsl/)  
  
## Prepare your Windows 10 machine  
Start by adding the Windows Subsystem for Linux in the Programs and Features on your Windows 10 box, **NOTE** you must be running:  
>Windows 10, updated to version 2004  
>Build 19041 or higher.  
>64-bit Machine (for Kernel Update)  
  
For proper sanity sake **REBOOT**  
  
Open up PowerShell as admin (All PowerShell references, will be "Powershell as admin", I will shorten this to PAA)  
`dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart`  
  
Next enable the Virtual Machine Platform  
`dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart`  
  
Download the latest WSL2 kernel and install it.  
This is the link to the latest kernel via the Microsoft article earlier  
(https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)  
  
**REBOOT**  
  
Open up Powershell (PAA) and set the default version of WSL to 2  
`wsl --set-default-version 2`  
  
Now off to Microsoft Store and grab your preferred distro, I prefer to use **Debian** so that is what my instructions will reflect.  
Launch the distro when it is downloaded and then provide a user if you desire, (if you escape out of it, you will have a default user of root.)  
Close the app (distro) window  
  
If you recieve the following error: **"Error 0x80370102"** this is either 2 things that I gather.  
1. You need to enable virtualization on your BIOS  
2. You are trying this on a VM (I have no answer to this yet)  
  
Now check to make sure the WSL is defaulted to version 2  
Open up powershell as admin (just in case we need to change the version)  
  
`wsl --list --verbose`
|   NAME   |   STATE   |   VERSION   |
|----------|:---------:|------------:|
|* Debian  |  Stopped  |       2     |
  
If you are not on version 2, you can force it with the following command   
  
wsl --set-version <distribution name> <versionNumber>  (In my example this would be "wsl --set-version Debian 2")  
  
Now that you are on version 2 open the distro app and perform an update and upgrade  
  
`sudo apt update && sudo apt upgrade`
  
When that is finished, grab these packages (thanks to https://exploits.run/parrot-wsl/ if you don't have the gnupg, the script fails)  
  
'sudo apt install wget curl gnupg -y'
  
I would advise you add a new DNS or change to a public DNS (Subsystem in Windows, tries to DNS query through the host machine which causes some issues with parrot-install script)  
If you have never changed your DNS then you just need to edit /etc/resolv.conf and add something like  
>nameserver 8.8.8.8  
>nameserver 1.1.1.1  
  
Now back to the (https://community.parrotsec.org/t/parrot-on-wsl2/14898) the sections that starts Debian -> Parrot  
  
>curl https://raw.githubusercontent.com/ParrotSec/alternate-install/master/parrot-install.sh -o parrot-install.sh  
>chmod a+x parrot-install.sh  
>sudo ./parrot-install.sh  
  
I chose 1 to let it install the core (there were a few packages missed but you can ignore them for now)  
This adds an entry we can edit to get the parrot deb  
  
Change the deb mirror to a more current one   
edit the /etc/apt/sources.list.d/parrot.list    
remove the current deb and replace with the following (exercise caution as you are now trusting an unsigned source)  
`deb [trusted=yes] http://mirrors.mit.edu/parrot/ rolling main contrib non-free`
  
Perform another update and upgrade, this should take some time as it starts to install the parrot base and rolling packages  
`sudo apt update && sudo apt upgrade'
  
After that I went to the next step which was to install the parrot tools  
`sudo apt-get install parrot-interface parrot-interface-full parrot-tools-full`  

When I ran the previous command I received the following error  
"The following packages have unmet dependencies:
 libc6-dev : Breaks: libgcc-8-dev (< 8.4.0-2~) but 8.3.0-6 is to be installed
E: Error, pkgProblemResolver::Resolve generated breaks, this may be caused by held packages."    
So I removed the bad package   
`sudo apt remove libgcc-8-dev`     
and re ran the installer    
`sudo apt-get install parrot-interface parrot-interface-full parrot-tools-full`  

Once that is finished a final update check  
`sudo apt update && sudo apt upgrade'  

now close the debian instance and open it back up and you should see the theme for parrot and have the parrot-upgrade command  
  
Now that is done, I'm sure you want to install a visual client, so xRDP  
`sudo apt install xrdp`  
Now for Quality of Life (changing ports, bpp) to make xRDP look better  
  
`sudo cp /etc/xrdp/xrdp.ini /etc/xrdp/xrdp.ini.bak`  
`sudo sed -i 's/3389/3390/g' /etc/xrdp/xrdp.ini`  
`sudo sed -i 's/max_bpp=32/#max_bpp=32\nmax_bpp=128/g' /etc/xrdp/xrdp.ini`  
`sudo sed -i 's/xserverbpp=24/#xserverbpp=24\nxserverbpp=128/g' /etc/xrdp/xrdp.ini`  
  
Now start the xrdp  
`sudo /etc/init.d/xrdp start`  
Check status of xrdp  
`sudo /etc/init.d/xrdp status`  
  
 # Enjoy you're newly installed and updated Parrot Sec running on WSL 2 on Windows!
  
