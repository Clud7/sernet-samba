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
- Use the following command `yum install sernet-samba-ad --allowerasing` we use allowerasing to replace conflicting packages.

- Install also the `yum install sernet-samba-ad-tools` if needed.

## Part 6U Update or Upgrade Sernet-Samba
- Use the repo that you have and update  `yum update sernet-samba-ad` (you should have the latest version).
- After that add the new repo like name=SerNet Samba 4.15 Packages (almalinux-8) (DO NOT DELETE THE OLD REPO 4.14)
- Run `yum clean all` and  `yum update sernet-samba-ad` and you are rdy to go!
- Now you can update your server by `yum update` command.

## Part 6 Configure WinBind and NTP (Chrony as NTP server).
- Install BIND DNS Server with command `yum install bind`
> Do not edit your named server. Just install the bind.


## Part 7 JOIN Existed Domain
Set SERNET-SAMBA Role as AD
Go and edit the file `/etc/default/sernet-samba` and set the `SAMBA_START_MODE="none"` to `SAMBA_START_MODE="ad"`
```
# SAMBA_START_MODE defines how Samba should be started. Valid options are one of
#   "none"    to not enable it at all,
#   "classic" to use the classic smbd/nmbd/winbind daemons
#   "ad"      to use the Active Directory server (which starts the smbd and
#             winbindd on its own)
# (Be aware that you also need to enable the services/init scripts that
# automatically start up the desired daemons.)
SAMBA_START_MODE="ad"
```

Use the following command to join to existed domain
```
samba-tool domain join yourdomain.local DC -U"YOURDOMAIN\Administrator" --dns-backend=BIND9_DLZ --option="idmap_ldb:use rfc2307 = yes" --option="dns forwarder=8.8.8.8"
```

Check Server Synchronization
```
samba-tool drs showrepl
```

Update DNS
```
samba_dnsupdate --verbose --use-samba-tool
```

## PART 8 Setup BIND9 DNS (NAMED)
Check the `/var/lib/samba/bind-dns/named.conf` and remove the comment mark `#` for the correct version.

Update your `named.conf`. 
You can edit the named.conf via the webmin or even better via the following command `vi /etc/named.conf` and don't forget to fix the following:
1) Add your static ip --> `listen-on port`
2) Add the following line --> `tkey-gssapi-keytab "/var/lib/samba/private/dns.keytab";`
3) Add the following line --> `include "/var/lib/samba/bind-dns/named.conf";`
Your Configuration should be like:
```
//
// named.conf
//
// Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
// server as a caching only nameserver (as a localhost DNS resolver only).
//
// See /usr/share/doc/bind*/sample/ for example named configuration files.
//

options {
	listen-on port 53 { 127.0.0.1;
    					YOUR_STATIC_IP;};
	listen-on-v6 port 53 { ::1; };
	directory 	"/var/named";
	dump-file 	"/var/named/data/cache_dump.db";
	statistics-file "/var/named/data/named_stats.txt";
	memstatistics-file "/var/named/data/named_mem_stats.txt";
	secroots-file	"/var/named/data/named.secroots";
	recursing-file	"/var/named/data/named.recursing";
	allow-query     { localhost;
    			YOUR_LAN_NETWORK;
                      	YOUR_LAN2_NETWORK;
                      	};
                      
    forwarders {
    			1.1.1.1;
                8.8.8.8;
               };

	/* 
	 - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
	 - If you are building a RECURSIVE (caching) DNS server, you need to enable 
	   recursion. 
	 - If your recursive DNS server has a public IP address, you MUST enable access 
	   control to limit queries to your legitimate users. Failing to do so will
	   cause your server to become part of large scale DNS amplification 
	   attacks. Implementing BCP38 within your network would greatly
	   reduce such attack surface 
	*/
	recursion yes;

	dnssec-enable yes;
	dnssec-validation yes;

	managed-keys-directory "/var/named/dynamic";

	pid-file "/run/named/named.pid";
	session-keyfile "/run/named/session.key";

	/* 
    	https://fedoraproject.org/wiki/Changes/CryptoPolicy 
    */
	include "/etc/crypto-policies/back-ends/bind.config";
    tkey-gssapi-keytab "/var/lib/samba/private/dns.keytab";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "." IN {
	type hint;
	file "named.ca";
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
include "/var/lib/samba/bind-dns/named.conf";

```

Enable named to start on startup with the command `systemctl enable named`.
Check the status of named --> `systemctl status named`.

# Part 9 Fix your krb5.conf file
Update or Replace your `/etc/krb5.conf` with the file `/var/lib/samba/private/krb5.conf`

# Part 10 Fix your chrony and setup an NTP Server
Chrony is already install on your Alma Linux but you can also check it via --> `yum install chrony`
Check the status of chrony `systemctl status chronyd`.

Edit the configuration file `vi /etc/chrony.conf` and add your main DC as iburst

```
pool time.google.com iburst
server YOURSERVER_IP iburst
```

Allow NTP Queries from:
```
# Allow NTP client access from local network.
#allow 192.168.0.0/16
allow YOURNETWORK
allow YOURNETWORK2
```
