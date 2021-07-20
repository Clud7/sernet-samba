# sernet-samba - ALMA LINUX
## Full tutorial how to install your Sernet Samba+ 
(Active Directory - Domain Controller)
> Works on Windows 10 and 11

### What you need:
1. First of all you need an active Subscription to Samba+
> You can buy one with 300€ here --> https://shop.samba.plus/en/


2. Server with min 30GB Storage and 4GB Ram.


3. Setup a new clean installation of Alma Linux
> You can download alma linux here --> https://almalinux.org/

## PART 1 (Updates)
- Update your system to the latest version. With root user use the command `yum update`.
- Install the EPEL Release Repo. With root user use the command `yum install epel-release -y`.
- Enable Cockpit Web Console `systemctl enable --now cockpit.socket`

## Part 2 (Disable Selinux & Firewalld & Virtualization Service)
Check Selinux with `sestatus`
To permanently disable SELinux on your Alma Linux system:
- Open the `/etc/selinux/config` file and set the SELINUX mod to disabled.
```
    # This file controls the state of SELinux on the system.
    # SELINUX= can take one of these three values:
    #       enforcing - SELinux security policy is enforced.
    #       permissive - SELinux prints warnings instead of enforcing.
    #       disabled - No SELinux policy is loaded.
    SELINUX=disabled
    # SELINUXTYPE= can take one of these two values:
    #       targeted - Targeted processes are protected,
    #       mls - Multi Level Security protection.
    SELINUXTYPE=targeted`
```
- To permanently disable firewalld
```
    # systemctl stop firewalld
    # systemctl disable firewalld
```
- To permanently disable virtualization service
```
    # systemctl stop libvirtd.service
    # systemctl disable libvirtd.service
```


## Part 3 (install webmin)
(info --> http://www.webmin.com/rpm.html )
- Create the `/etc/yum.repos.d/webmin.repo` file containing 
```
[Webmin]
name=Webmin Distribution Neutral
#baseurl=https://download.webmin.com/download/yum
mirrorlist=https://download.webmin.com/download/yum/mirrorlist
enabled=1
```
- You should also fetch and install my GPG key with which the packages are signed, with the commands:
```
wget http://www.webmin.com/jcameron-key.asc
rpm --import jcameron-key.asc
```
- Install with command `yum install webmin`


- Restart your Alma Linux system `shutdown -r now`

## Part 4 (Install Software)
AS root install:
- htop - An interactive process viewer with command `yum install htop`
- iftop - A Real Time Linux Network Bandwidth Monitoring Tool with command `yum install iftop`
- iotop – Monitor Linux Disk I/O Activity and Usage Per-Process Basis with command `yum install iotop`

## Part 5 SERNET REPO
- Create the `/etc/yum.repos.d/sernet-samba-4.14.repo` file containing 
```
[sernet-samba-4.14]
name=SerNet Samba 4.14 Packages (almalinux-8)
type=rpm-md
baseurl=https://KEY:PASSWORD@download.sernet.de/subscriptions/samba/4.14/almalinux/8/
gpgcheck=1
gpgkey=https://KEY:PASSWORD@download.sernet.de/subscriptions/samba/4.14/almalinux/8/repodata/repomd.xml.key
enabled=1
```
> Note that you have to change they KEY and the PASSWORD. As key use your username from https://oposso.samba.plus/subscription.php and as password the password that you have set.

## Part 6 Install Sernet-Samba
- Use the following command `yum install sernet-samba --allowerasing` we use allowerasing to replace conflicting packages.
