# Ubuntu-22.04-LXC-Desktop
Ubuntu-24.04-LXC-Desktop for Proxmox

Ubuntu proxmox LXC Container with desktop


Part A: setting the template.
1.	Create a template for ubuntu, give it 100G of storage, 2 or 4 cpu (can change to what you want), specify the ip you want (ex: 192.168.1.10/24), provide a gateway or leave it DHCP and check the IP on the LXC container terminal.

2.	Log in as root once you start the new lxc add an admin user and add it to sudo group:

>  #adduser admin
>
>  #usermod -aG sudo admin

3.	Edit Source.list: 
>  #sudo nano /etc/apt/sources.list

Add the following sources: 

>  deb http://archive.ubuntu.com/ubuntu/ jammy main restricted universe multiverse
>
>  deb-src http://archive.ubuntu.com/ubuntu/ jammy main restricted universe multiverse
>
>  deb http://archive.ubuntu.com/ubuntu/ jammy-updates main restricted universe multiverse
>
>  deb-src http://archive.ubuntu.com/ubuntu/ jammy-updates main restricted universe multiverse
>
>  deb http://archive.ubuntu.com/ubuntu/ jammy-security main restricted universe multiverse
>
>  deb-src http://archive.ubuntu.com/ubuntu/ jammy-security main restricted universe multiverse
>
>  deb http://archive.ubuntu.com/ubuntu/ jammy-backports main restricted universe multiverse
>
>  deb-src http://archive.ubuntu.com/ubuntu/ jammy-backports main restricted universe multiverse
>
>  deb http://archive.canonical.com/ubuntu jammy partner
>
>  deb-src http://archive.canonical.com/ubuntu jammy partner

3.	Update:
>  #sudo apt update
> 
>  #sudo apt full-upgrade

4.	Install the desktop environment: during the install you will see a screen asking which desktop you want:

  select lighdm.
>  #sudo apt install xfce4 xfce4-goodies xorg dbus-x11 x11-xserver-utils

  since we want to access the lxc through remote RDP, install XRDP
>  #sudo apt install xrdp ((If you want to use the latest version of XRDP go to part B)

6.	Verify xrdp installed correctly and is running: 
>  #sudo systemctl status xrdp

7.	Once RDP is installed, we need to add this code, so that desktop manager know what to use for display:
>  #update-alternatives --set x-session-manager /usr/bin/xfce4-session

8.	Install Firefox browser or any other software you like to have in your templates (Terminal/chrome browser, for example)
>  #sudo apt-get install firefox -y

9.	Create a backup of the new created LXC as as ZSTD. first, turn off the lxc container and then do the backup from Proxmox GUI. Pve-> select container —> Backup now—> compression select ZSTD (fast and good)

10.	Once the backup is completed. Go the Shell of Proxmox: copy the new file created to the folder where Proxmox saves templates and give it a new name.In my case the templates are saved in the following folder, your will be different, look under your zpool or where you saved your selected to save templates when you download them:Template-folder: /your-zfs-pool-name/zfs-iso-template/template/cacheBackup-folder-file: /your-zfs-pool/zfs-backups/dump/vzdump-lxc-111-2020_07_27-14_52_18.tar.gz cp backup-folder-file template-folder/new-name.tar.gz

11.	That's it! Now when you go to create new container, you will have a new template that you can use to create new containers with desktop environment and all software per-installed.

Part B: installing XRDP with sound to work:
1.	get latest download link of xrd-installer-1.2 from this website:
>  #cd /
> 
>  #cd opt
> 
>  #sudo wget https://c-nergy.be/downloads/xRDP/xrdp-installer-1.5.2.zip
> 
>  #sudo unzip xrdp-installer-1.5.2.zip
> 
>  #sudo chmod +x xrdp-installer-1.5.2.zip
> 
>  #./xrdp-installer-1.5.2.sh --sound
> 
>  #sudo reboot
> 

https://c-nergy.be/blog/?p=19951
--help or -h          => will display a basic help menu
--sound or -s         => will enable sound redirection 
--loginscreen or -l   => will customize the xRDP login screen 
--remove or -r        => will remove the xrdp package 
--custom or -c        => will perform a custom installation (i.e. compiled from sources)
--dev or -d           => will perform a custom installation using dev branch (unstable version)
--unsupported or -u   => will bypass the Check os and will run against unsupported os (use it at your own risk!!)
--perm or -p          => fix permissions on xrdp files (uncommon situation)


2.	create a new user if you don't already have one (as root):
>  #adduser admin
>  #usermod -aG sudo admin
>  #reboot
reboot container.

3.	After rebooting the container, i was getting Dummy output in the Volume control and no sound output. To fix the issue restart pulseaudio server in the container by running the following command:
>  #pulseaudio -k

After pulseaudio is restarted, the sound should start to work with xrdp sink appearing in the volume control.

Part C: You can use/try a different desktop environment than xfce4, after all this is the beauty of LXC. before turning the new container to a template try different desktop environments. once you make your choice, remove/delete old desktop then set container as template in order to keep it to a small size. for UBUNTU-MATE desktop use the following:
>  #sudo apt install lightdm ubuntu-mate-core ubuntu-mate-desktop -y
>
>  #sudo update-alternatives --set x-session-manager /usr/bin/mate-session

___
Proxmox host, edit the config file at /etc/pve/lxc/<CTID>.conf
Code:
#nano /etc/pve/lxc/<CTID>.conf
features: mount=fuse,nesting=1
lxc.mount.entry: /dev/fuse dev/fuse none bind,create=file,optional
lxc.mount.auto: cgroup:rw
___
sudo apt install squashfuse fuse
sudo poweroff
______
/lib/systemd/systemd-udevd --daemon
____
https://forum.proxmox.com/threads/ubuntu-snaps-inside-lxc-container-on-proxmox.36463/

References/Credits: I want to give credits to the following developers and their sites. i wouldn't have came up with the steps without the help of information they provided. https://github.com/neutrinolabs/xrdp https://linuxize.com/post/how-to-install-xrdp-on-ubuntu-18-04/ https://github.com/bmullan/ciab-remote-desktop/blob/master/mk-cn1-environment.sh https://linuxconfig.org/8-best-ubuntu-desktop-environments-18-04-bionic-beaver-linux http://c-nergy.be/blog/?p=14888 https://www.closingtags.com/custom-lxc-container-templates-in-proxmox-5-0/ https://blog.simos.info/how-to-run-...pps-in-lxd-containers-on-your-ubuntu-desktop/

