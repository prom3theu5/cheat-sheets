### Install RHEL

1.  You'll need to install RHEL in a Virtual Machine (VM). It doesn't matter which hypervisor you use. I used ProxMox / QEMU.
    
    > Just to make things easier later, I went ahead and set the root password, and created an Administrator account
    
2.  Under Software Selection, I picked Minimal Install to keep things as small as possible.
3.  Once installed, reboot into RHEL and login as root.

### Unregister VM

> During the Installation, you had to register this machine with Red Hat. To make a clean image (that can be used over an over), it's best to remove the subscription, and unregister the VM.

1.  Run these commands to fully unregister, and remove all subscriptions:
    -   `subscription-manager remove --all`
    -   `subscription-manager unregister`
    -   `subscription-manager clean`

### Create Tarball and Transfer to WSL Host

1.  Create a tarball of the file system:
    -   `cd /`
    -   `tar cvfzp rhel8.tar.gz bin dev etc home lib lib64 media opt run root sbin srv tmp usr var`
        
        > Note: these are not all of the directories under / and the ones not listed are intentionally excluded
        
2.  Transfer the file rhel8.tar.gz to your host system.
    
    > To make it easy, I used WinSCP to connect to the VM (from the host), and copy the file
    

### Create WSL Instance and Start It

1.  Create a directory for WSL
    -   `mkdir c:\wsl`
2.  Create the WSL instance by importing the tarball:
    -   `wsl --import RHEL c:\wsl rhel8.tar.gz --version 2`
3.  Create a shortcut to wsl -d rhel to start, or start manually.


### OPTIONAL: Change Default user in WSL Instance

> -   You'll notice it starts as the root user. If you would like to change this, you can go to `HKCU\Software\Microsoft\Windows\CurrentVersion\Lxss` and change the `Decimal` value of `DefaultUid` for your distro to whatever you want. See image below for reference.
>     -   Typically this would be 1000 for the first user created (including the user you created during the setup process).


### Re-register with Red Hat

> In order to get updates, you'll need to re-register this instance with Red Hat

1.  You can use these commands to re-register:
    -   `subscription-manager register --username <username> --password <secret>`
    -  `subscription-manager refresh`
    -  `subscription-manager attach --auto`

2.  You should now be able to get updates using the command below:
  - `dnf upgrade`

If you changed the `DefaultUid` above in the registry, and you're not currently logged into the WSL instance as `root`, you'll need to append `sudo` in front of all of the commands above

### All Done!
