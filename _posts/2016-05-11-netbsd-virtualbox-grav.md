---
layout: post
title:  Grav CMS on NetBSD 7 VM in VirtualBox
---

On the host OS:

```
$ VBoxManage createvm --ostype NetBSD_64 --name NetBSDWeb --register

$ VBoxManage modifyvm NetBSDWeb --memory 2048 --vram 32 --cpus 1 --audio coreaudio --audiocontroller ac97 --biosbootmenu messageandmenu --clipboard bidirectional --usb on --usbehci on --nic1 nat --natpf1 "guestssh,tcp,,5555,,22" --natpf1 "guesthttp,tcp,,7777,,80" --nictype1 82540EM

$ VBoxManage storagectl NetBSDWeb --name IDE --add ide --controller PIIX4 --bootable on
$ VBoxManage createhd --filename /Users/dusko/VirtualBox\ VMs/NetBSDWeb/NetBSDWeb.vdi --size 20480 --format VDI
$ VBoxManage storageattach NetBSDWeb --storagectl IDE --port 0 --device 0 --type hdd --medium /Users/dusko/VirtualBox\ VMs/NetBSDWeb/NetBSDWeb.vdi
$ VBoxManage storageattach NetBSDWeb --storagectl IDE --port 0 --device 1 --type dvddrive --medium /Users/dusko/NetBSD-7.0-amd64.iso
$ VBoxManage startvm NetBSDWeb
```


```
1. Install NetBSD
>a: Installation messages in English
Keyboard type:  >a: unchanged
>a: Install NetBSD to hard disk
Shall we continue?  >b: Yes
Available disks   >a: wd0 (20G, VBOX HARDDISK)
>a: This is the correct geometry
>b: Use the entire disk
Do you want to install the NetBSD bootcode?  >a: Yes
>b: Use existing partition sizes
>x: Partition sizes ok
Please enter a name for your NetBSD disk [VBOX HARDDISK  ]:  <enter>
Shall we continue?  >b: Yes
Bootblocks selection  >a: Use BIOS console
>x: Exit
>a: Full installation
Install from  >a: CD-ROM / DVD / install image media
>Hit enter to continue
>a: Configure network
Available interfaces  >a: wm0
Network media type [autoselect]: <enter>
Perform autoconfiguration?  >a: Yes
Your host name:  nbsdweb

The following are the values you entered.
DNS Domain:             telus
Host Name:              nbsdweb
Nameserver:             84.200.69.80
Primary Interface:      wm0
Media type:             autoselect
Host IP:                10.0.2.15
Netmask:                255.255.255.0
IPv4 Gateway:           10.0.2.2
IPv6 autoconf:

Are they OK?  >a: Yes

Is the network information you entered accurate for this machine
in regular operation and do you want it installed in /etc?
>a: Yes

>b: Timezone
>America > America/Vancouver
>Exit

>d: Change root password
>a: Yes
New password: ******************
Retype new password: ******************

>e: Enable installation of binary packages
>a: Host                   ftp.NetBSD.org
 b: Base directory         pub/pkgsrc/packages/NetBSD
 c: Package directory      /amd64/7.0/All
 d: User                   ftp
 e: Password 
 f: Proxy
 g: Additional packages
 h: Configure network
 i: Quit installing binary pkgs
 x: Install pkgin and update package summary

>x: Install pkgin and update package summary
>Hit enter to continue

>g: Enable sshd
>h: Enable ntpd
>i: Run ntpdate at boot

>o: Add a user
8 character username to add: dusko
Do you wish to add this user to group wheel?
>a: Yes
User shell  >a: /bin/sh
New password:
Retype new password:

>x: Finished configuring
>Hit enter to continue

>x: Exit Install System
# shutdown -p now 
```


On the host OS:
```
$ VBoxManage storageattach NetBSDWeb --storagectl IDE --port 0 --device 1 --medium emptydrive
$ VBoxManage startvm NetBSDWeb

$ ssh -p 5555 dusko@localhost
```


```
$ su
# cd /usr && cvs -q -z2 -d anoncvs@anoncvs.NetBSD.org:/cvsroot checkout -r pkgsrc-2015Q4 -P pkgsrc > /home/dusko/pkgsrc_via_cvs.txt 2>&1

# echo "# Recommended CVS configuration file from the pkgsrc guide" > /root/.cvsrc
# echo "#     http://www.netbsd.org/docs/pkgsrc/getting.html#getting-via-cvs" >> /root/.cvsrc
# echo "cvs -q -z2" >> /root/.cvsrc
# echo "checkout -P" >> /root/.cvsrc
# echo "update -dP" >> /root/.cvsrc
# echo "diff -upN" >> /root/.cvsrc
# echo "rdiff -u" >> /root/.cvsrc
# echo "release -d" >> /root/.cvsrc

# cd /usr/pkgsrc && cvs update -dP

# export PKG_PATH="http://ftp.NetBSD.org/pub/pkgsrc/packages/$(uname -s)/$(uname -m)/$(uname -r)/All" 

# ftp -o - `echo $PKG_PATH` | grep -i php- | grep 5 
<a href="ap22-suphp-0.7.2.tgz">ap22-suphp-0.7.2.tgz</a>                     12-Jan-2016 16:08             75kB
<a href="asp2php-0.77.3.tgz">asp2php-0.77.3.tgz</a>                       10-Jan-2016 12:35             44kB
<a href="mserv-php-0.90nb1.tgz">mserv-php-0.90nb1.tgz</a>                    09-Mar-2016 15:54             16kB
<a href="p5-PHP-Serialization-0.34nb5.tgz">p5-PHP-Serialization-0.34nb5.tgz</a>         04-Jan-2016 14:54              8kB
<a href="php-5.5.33.tgz">php-5.5.33.tgz</a>                           08-Mar-2016 20:12           5489kB
<a href="php-5.6.19.tgz">php-5.6.19.tgz</a>                           08-Mar-2016 20:16           5543kB


# pkg_add -v php-5.6.19

# ftp -o - `echo $PKG_PATH` | grep -i php | grep 5 | grep gd 
<a href="php55-gd-5.5.33.tgz">php55-gd-5.5.33.tgz</a>                      08-Mar-2016 20:18            146kB
<a href="php56-gd-5.6.19.tgz">php56-gd-5.6.19.tgz</a>                      08-Mar-2016 20:28            146kB
<a href="php70-gd-7.0.4.tgz">php70-gd-7.0.4.tgz</a>                       09-Mar-2016 21:46            145kB

# pkg_add -v php56-gd

# cp /usr/pkg/etc/php.ini /usr/pkg/etc/php.ini.original.bak
# grep -n extension\= /usr/pkg/etc/php.ini
854:;   extension=modulename.extension
... ... ...
... ... ...
912:;extension=php_xsl.dll

# vi +854 /usr/pkg/etc/php.ini
# sed -n 868,872p /usr/pkg/etc/php.ini
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
; Dynamic Extensions required for Grav ;
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
extension=gd.so


# diff --unified=0 /usr/pkg/etc/php.ini.original.bak /usr/pkg/etc/php.ini
--- /usr/pkg/etc/php.ini.original.bak   2016-05-07 12:58:30.000000000 -0700
+++ /usr/pkg/etc/php.ini        2016-05-07 13:02:32.000000000 -0700
@@ -867,0 +868,5 @@
+;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
+; Dynamic Extensions required for Grav ;
+;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
+extension=gd.so
+


# printf "Check whether gd.so is loading fine.    \n\n"
Check whether gd.so is loading fine.    

# php -v
PHP 5.6.19 (cli) (built: Mar  8 2016 20:15:11) 
Copyright (c) 1997-2016 The PHP Group
Zend Engine v2.6.0, Copyright (c) 1998-2016 Zend Technologies

# ftp -o - `echo $PKG_PATH` | grep -i php | grep 5 | grep curl 
<a href="php55-curl-5.5.33.tgz">php55-curl-5.5.33.tgz</a>                    08-Mar-2016 20:34             35kB
<a href="php56-curl-5.6.19.tgz">php56-curl-5.6.19.tgz</a>                    08-Mar-2016 20:50             35kB
<a href="php70-curl-7.0.4.tgz">php70-curl-7.0.4.tgz</a>                     09-Mar-2016 21:53             34kB

# pkg_add -v php56-curl

# diff --unified=0 /usr/pkg/etc/php.ini.original.bak /usr/pkg/etc/php.ini
--- /usr/pkg/etc/php.ini.original.bak   2016-05-07 12:58:30.000000000 -0700
+++ /usr/pkg/etc/php.ini        2016-05-07 13:09:54.000000000 -0700
@@ -867,0 +868,6 @@
+;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
+; Dynamic Extensions required for Grav ;
+;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
+extension=gd.so
+extension=curl.so
+

# php -v
PHP 5.6.19 (cli) (built: Mar  8 2016 20:15:11) 
Copyright (c) 1997-2016 The PHP Group
Zend Engine v2.6.0, Copyright (c) 1998-2016 Zend Technologies


# ftp -o - `echo $PKG_PATH` | grep -i php | grep crypt 
<a href="php55-mcrypt-5.5.33.tgz">php55-mcrypt-5.5.33.tgz</a>                  08-Mar-2016 20:32             18kB
<a href="php56-mcrypt-5.6.19.tgz">php56-mcrypt-5.6.19.tgz</a>                  08-Mar-2016 21:23             18kB
<a href="php70-mcrypt-7.0.4.tgz">php70-mcrypt-7.0.4.tgz</a>                   09-Mar-2016 22:22             17kB

# pkg_add -v php56-mcrypt
# vi +872 /usr/pkg/etc/php.ini

# diff --unified=0 /usr/pkg/etc/php.ini.original.bak /usr/pkg/etc/php.ini
--- /usr/pkg/etc/php.ini.original.bak   2016-05-07 12:58:30.000000000 -0700
+++ /usr/pkg/etc/php.ini        2016-05-07 13:18:08.000000000 -0700
@@ -867,0 +868,7 @@
+;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
+; Dynamic Extensions required for Grav ;
+;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
+extension=gd.so
+extension=curl.so
+extension=mcrypt.so
+


# php -v
PHP 5.6.19 (cli) (built: Mar  8 2016 20:15:11) 
Copyright (c) 1997-2016 The PHP Group
Zend Engine v2.6.0, Copyright (c) 1998-2016 Zend Technologies


# pkg_add -v openssl


# ftp -o - `echo $PKG_PATH` | grep -i php | grep 5 | grep zip 
<a href="php55-zip-5.5.33.tgz">php55-zip-5.5.33.tgz</a>                     08-Mar-2016 20:33             46kB
<a href="php56-zip-5.6.19.tgz">php56-zip-5.6.19.tgz</a>                     08-Mar-2016 20:45             57kB
<a href="php70-zip-7.0.4.tgz">php70-zip-7.0.4.tgz</a>                      09-Mar-2016 21:51             63kB


# pkg_add -v php56-zip
# vi +873 /usr/pkg/etc/php.ini

# diff --unified=0 /usr/pkg/etc/php.ini.original.bak /usr/pkg/etc/php.ini
--- /usr/pkg/etc/php.ini.original.bak   2016-05-07 12:58:30.000000000 -0700
+++ /usr/pkg/etc/php.ini        2016-05-07 13:21:57.000000000 -0700
@@ -867,0 +868,8 @@
+;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
+; Dynamic Extensions required for Grav ;
+;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
+extension=gd.so
+extension=curl.so
+extension=mcrypt.so
+extension=zip.so
+


# php -v
PHP 5.6.19 (cli) (built: Mar  8 2016 20:15:11) 
Copyright (c) 1997-2016 The PHP Group
Zend Engine v2.6.0, Copyright (c) 1998-2016 Zend Technologies


# ftp -o - `echo $PKG_PATH` | grep -i php | grep 5 | grep mbstring
<a href="php55-mbstring-5.5.33.tgz">php55-mbstring-5.5.33.tgz</a>                08-Mar-2016 20:29            664kB
<a href="php56-mbstring-5.6.19.tgz">php56-mbstring-5.6.19.tgz</a>                08-Mar-2016 20:35            664kB


# pkg_add -v php56-mbstring
# vi +874 /usr/pkg/etc/php.ini

# diff --unified=0 /usr/pkg/etc/php.ini.original.bak /usr/pkg/etc/php.ini
--- /usr/pkg/etc/php.ini.original.bak   2016-05-07 12:58:30.000000000 -0700
+++ /usr/pkg/etc/php.ini        2016-05-07 13:24:46.000000000 -0700
@@ -867,0 +868,9 @@
+;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
+; Dynamic Extensions required for Grav ;
+;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
+extension=gd.so
+extension=curl.so
+extension=mcrypt.so
+extension=zip.so
+extension=mbstring.so
+


# php -v
PHP 5.6.19 (cli) (built: Mar  8 2016 20:15:11) 
Copyright (c) 1997-2016 The PHP Group
Zend Engine v2.6.0, Copyright (c) 1998-2016 Zend Technologies


# ftp -o - `echo $PKG_PATH` | grep -i php | grep 5 | grep xml 
<a href="php55-xmlrpc-5.5.33.tgz">php55-xmlrpc-5.5.33.tgz</a>                  08-Mar-2016 21:25             43kB
<a href="php56-xmlrpc-5.6.19.tgz">php56-xmlrpc-5.6.19.tgz</a>                  09-Mar-2016 00:50             43kB


# pkg_add -v php56-xmlrpc
# vi +875 /usr/pkg/etc/php.ini

# diff --unified=0 /usr/pkg/etc/php.ini.original.bak /usr/pkg/etc/php.ini
--- /usr/pkg/etc/php.ini.original.bak   2016-05-07 12:58:30.000000000 -0700
+++ /usr/pkg/etc/php.ini        2016-05-07 13:27:01.000000000 -0700
@@ -867,0 +868,10 @@
+;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
+; Dynamic Extensions required for Grav ;
+;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
+extension=gd.so
+extension=curl.so
+extension=mcrypt.so
+extension=zip.so
+extension=mbstring.so
+extension=xmlrpc.so
+


# php -v
PHP 5.6.19 (cli) (built: Mar  8 2016 20:15:11) 
Copyright (c) 1997-2016 The PHP Group
Zend Engine v2.6.0, Copyright (c) 1998-2016 Zend Technologies


# ftp -o - `echo $PKG_PATH` | grep -i php | grep 5 | grep apcu 
<a href="php55-apcu-4.0.7.tgz">php55-apcu-4.0.7.tgz</a>                     09-Mar-2016 05:57             51kB
<a href="php56-apcu-4.0.7.tgz">php56-apcu-4.0.7.tgz</a>                     09-Mar-2016 06:01             51kB

# pkg_add -v php56-apcu
# vi +876 /usr/pkg/etc/php.ini


#  diff --unified=0 /usr/pkg/etc/php.ini.original.bak /usr/pkg/etc/php.ini
--- /usr/pkg/etc/php.ini.original.bak   2016-05-07 12:58:30.000000000 -0700
+++ /usr/pkg/etc/php.ini        2016-05-07 13:30:25.000000000 -0700
@@ -867,0 +868,11 @@
+;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
+; Dynamic Extensions required for Grav ;
+;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
+extension=gd.so
+extension=curl.so
+extension=mcrypt.so
+extension=zip.so
+extension=mbstring.so
+extension=xmlrpc.so
+extension=apcu.so
+


# php -v
PHP 5.6.19 (cli) (built: Mar  8 2016 20:15:11) 
Copyright (c) 1997-2016 The PHP Group
Zend Engine v2.6.0, Copyright (c) 1998-2016 Zend Technologies


# ftp -o - `echo $PKG_PATH` | grep -i php | grep 5 | grep opcache 
<a href="php55-opcache-5.5.33.tgz">php55-opcache-5.5.33.tgz</a>                 08-Mar-2016 21:26             67kB
<a href="php55-zendopcache-7.0.5.tgz">php55-zendopcache-7.0.5.tgz</a>              09-Mar-2016 05:57             70kB
<a href="php56-opcache-5.6.19.tgz">php56-opcache-5.6.19.tgz</a>                 09-Mar-2016 01:12             73kB


# pkg_add -v php56-opcache
# vi +877 /usr/pkg/etc/php.ini


# diff --unified=0 /usr/pkg/etc/php.ini.original.bak /usr/pkg/etc/php.ini
--- /usr/pkg/etc/php.ini.original.bak   2016-05-07 12:58:30.000000000 -0700
+++ /usr/pkg/etc/php.ini        2016-05-07 13:32:44.000000000 -0700
@@ -867,0 +868,12 @@
+;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
+; Dynamic Extensions required for Grav ;
+;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
+extension=gd.so
+extension=curl.so
+extension=mcrypt.so
+extension=zip.so
+extension=mbstring.so
+extension=xmlrpc.so
+extension=apcu.so
+zend_extension=opcache.so
+


# php -v
PHP 5.6.19 (cli) (built: Mar  8 2016 20:15:11) 
Copyright (c) 1997-2016 The PHP Group
Zend Engine v2.6.0, Copyright (c) 1998-2016 Zend Technologies
    with Zend OPcache v7.0.6-dev, Copyright (c) 1999-2016, by Zend Technologies


# ftp -o - `echo $PKG_PATH` | grep -i php | grep 5 | grep xcache 
<a href="php55-xcache-3.2.0nb1.tgz">php55-xcache-3.2.0nb1.tgz</a>                09-Mar-2016 06:18            116kB
<a href="php56-xcache-3.2.0nb1.tgz">php56-xcache-3.2.0nb1.tgz</a>                09-Mar-2016 06:18            117kB


# pkg_add -v php56-xcache
# vi +878 /usr/pkg/etc/php.ini

# diff --unified=0 /usr/pkg/etc/php.ini.original.bak /usr/pkg/etc/php.ini
--- /usr/pkg/etc/php.ini.original.bak   2016-05-07 12:58:30.000000000 -0700
+++ /usr/pkg/etc/php.ini        2016-05-07 13:36:18.000000000 -0700
@@ -867,0 +868,13 @@
+;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
+; Dynamic Extensions required for Grav ;
+;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
+extension=gd.so
+extension=curl.so
+extension=mcrypt.so
+extension=zip.so
+extension=mbstring.so
+extension=xmlrpc.so
+extension=apcu.so
+zend_extension=opcache.so
+extension=xcache.so
+


# php -v
PHP 5.6.19 (cli) (built: Mar  8 2016 20:15:11) 
Copyright (c) 1997-2016 The PHP Group
Zend Engine v2.6.0, Copyright (c) 1998-2016 Zend Technologies
    with XCache v3.2.0, Copyright (c) 2005-2014, by mOo
    with Zend OPcache v7.0.6-dev, Copyright (c) 1999-2016, by Zend Technologies
    with XCache Optimizer v3.2.0, Copyright (c) 2005-2014, by mOo
    with XCache Cacher v3.2.0, Copyright (c) 2005-2014, by mOo
    with XCache Coverager v3.2.0, Copyright (c) 2005-2014, by mOo


# ftp -o - `echo $PKG_PATH` | grep -i yaml
... ... ...
... ... ...
<a href="libyaml-0.1.6nb1.tgz">libyaml-0.1.6nb1.tgz</a>                     30-Dec-2015 18:04            115kB
... ... ...
... ... ...


# pkg_add -v libyaml


# ftp -o - `echo $PKG_PATH` | grep -i php | grep 5 | grep xdebug 
<a href="php55-xdebug-2.3.3.tgz">php55-xdebug-2.3.3.tgz</a>                   09-Mar-2016 05:59            101kB
<a href="php56-xdebug-2.3.3.tgz">php56-xdebug-2.3.3.tgz</a>                   09-Mar-2016 06:02            101kB


# pkg_add -v php56-xdebug
# vi +880 /usr/pkg/etc/php.ini

# diff --unified=0 /usr/pkg/etc/php.ini.original.bak /usr/pkg/etc/php.ini
--- /usr/pkg/etc/php.ini.original.bak   2016-05-07 12:58:30.000000000 -0700
+++ /usr/pkg/etc/php.ini        2016-05-07 13:43:25.000000000 -0700
@@ -867,0 +868,14 @@
+;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
+; Dynamic Extensions required for Grav ;
+;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
+extension=gd.so
+extension=curl.so
+extension=mcrypt.so
+extension=zip.so
+extension=mbstring.so
+extension=xmlrpc.so
+extension=apcu.so
+zend_extension=opcache.so
+extension=xcache.so
+zend_extension=xdebug.so
+


# php -v
PHP 5.6.19 (cli) (built: Mar  8 2016 20:15:11) 
Copyright (c) 1997-2016 The PHP Group
Zend Engine v2.6.0, Copyright (c) 1998-2016 Zend Technologies
    with XCache v3.2.0, Copyright (c) 2005-2014, by mOo
    with Zend OPcache v7.0.6-dev, Copyright (c) 1999-2016, by Zend Technologies
    with Xdebug v2.3.3, Copyright (c) 2002-2015, by Derick Rethans
    with XCache Optimizer v3.2.0, Copyright (c) 2005-2014, by mOo
    with XCache Cacher v3.2.0, Copyright (c) 2005-2014, by mOo
    with XCache Coverager v3.2.0, Copyright (c) 2005-2014, by mOo


# printf "PEAR (PHP Extension and Application Repository) has the PHP Yaml module.    \n\n"
PEAR (PHP Extension and Application Repository) has the PHP Yaml module.    

# printf "PEAR includes PECL (PHP Extension Community Library).     \n\n"
PEAR includes PECL (PHP Extension Community Library).     


# ftp -o - `echo $PKG_PATH` | grep -i php | grep 5 | grep pear
<a href="php55-pear-1.10.1.tgz">php55-pear-1.10.1.tgz</a>                    08-Mar-2016 20:17            364kB
... ... ...
... ... ...
<a href="php56-pear-1.10.1.tgz">php56-pear-1.10.1.tgz</a>                    08-Mar-2016 20:18            364kB
... ... ...
... ... ...


# pkg_add -v php56-pear


# grep -n include_path /usr/pkg/etc/php.ini | grep pkg | grep php
710:include_path = ".:/usr/pkg/lib/php"


# which pear
/usr/pkg/bin/pear
# whereis pear
/usr/pkg/bin/pear

# which pecl
/usr/pkg/bin/pecl
# whereis pecl
/usr/pkg/bin/pecl


# pear search yaml 
Retrieving data...0%
no packages found that match pattern "yaml", for channel pear.php.net.


# pecl search yaml 
Retrieving data...0%
Matched packages, channel pecl.php.net:
=======================================
Package Stable/(Latest) Local
yaml    2.0.0RC7 (beta)       YAML-1.1 parser and emitter


# pecl install yaml
downloading yaml-1.2.0.tar ...
Starting to download yaml-1.2.0.tar (230,400 bytes)
.................................................done: 230,400 bytes
9 source files, building
running: phpize
Configuring for:
PHP Api Version:         20131106
Zend Module Api No:      20131226
Zend Extension Api No:   220131226
Cannot find autoconf. Please check your autoconf installation and the
$PHP_AUTOCONF environment variable. Then, rerun this script.

ERROR: `phpize' failed


# ftp -o - `echo $PKG_PATH` | grep -i autoconf 
<a href="autoconf-2.69nb6.tgz">autoconf-2.69nb6.tgz</a>                     30-Dec-2015 17:21            887kB
<a href="autoconf-archive-2015.02.24.tgz">autoconf-archive-2015.02.24.tgz</a>          10-Jan-2016 17:31            546kB
<a href="autoconf213-2.13nb5.tgz">autoconf213-2.13nb5.tgz</a>                  01-Jan-2016 01:20            244kB
<a href="p5-Config-AutoConf-0.311nb1.tgz">p5-Config-AutoConf-0.311nb1.tgz</a>          04-Jan-2016 02:53             36kB


# pkg_add -v autoconf

# pecl install yaml
# vi +881 /usr/pkg/etc/php.ini


# diff --unified=0 /usr/pkg/etc/php.ini.original.bak /usr/pkg/etc/php.ini
--- /usr/pkg/etc/php.ini.original.bak   2016-05-07 12:58:30.000000000 -0700
+++ /usr/pkg/etc/php.ini        2016-05-07 13:58:20.000000000 -0700
@@ -867,0 +868,15 @@
+;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
+; Dynamic Extensions required for Grav ;
+;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
+extension=gd.so
+extension=curl.so
+extension=mcrypt.so
+extension=zip.so
+extension=mbstring.so
+extension=xmlrpc.so
+extension=apcu.so
+zend_extension=opcache.so
+extension=xcache.so
+zend_extension=xdebug.so
+extension=yaml.so
+


# php -v
PHP 5.6.19 (cli) (built: Mar  8 2016 20:15:11) 
Copyright (c) 1997-2016 The PHP Group
Zend Engine v2.6.0, Copyright (c) 1998-2016 Zend Technologies
    with XCache v3.2.0, Copyright (c) 2005-2014, by mOo
    with Zend OPcache v7.0.6-dev, Copyright (c) 1999-2016, by Zend Technologies
    with Xdebug v2.3.3, Copyright (c) 2002-2015, by Derick Rethans
    with XCache Optimizer v3.2.0, Copyright (c) 2005-2014, by mOo
    with XCache Cacher v3.2.0, Copyright (c) 2005-2014, by mOo
    with XCache Coverager v3.2.0, Copyright (c) 2005-2014, by mOo


# ftp -o - `echo $PKG_PATH` | grep -i php | grep 5 | grep ap 
... ... ...
<a href="ap22-php56-5.6.19.tgz">ap22-php56-5.6.19.tgz</a>                    08-Mar-2016 21:23           2719kB
... ... ... 
... ... ... 
<a href="ap24-php56-5.6.19.tgz">ap24-php56-5.6.19.tgz</a>                    09-Mar-2016 06:25           2719kB
... ... ... 
... ... ... 


# pkg_add -v ap24-php56
# cp /usr/pkg/share/examples/rc.d/apache /etc/rc.d/apache
```


```
# grep -w name\= /etc/rc.d/apache
name="apache"

# cp /etc/rc.conf /etc/rc.conf.original.bak

# printf "apache=YES\n" >> /etc/rc.conf

# diff --unified=0 /etc/rc.conf.original.bak /etc/rc.conf
--- /etc/rc.conf.original.bak   2016-05-07 16:46:04.000000000 -0700
+++ /etc/rc.conf        2016-05-07 16:46:25.000000000 -0700
@@ -29,0 +30 @@
+apache=YES

# /etc/rc.d/apache start
Starting apache.
AH00557: httpd: apr_sockaddr_info_get() failed for nbsdweb.telus
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 127.0.0.1. Set the 'ServerName' directive globally to suppress this message

# /etc/rc.d/apache status
apache is running as pid 7367.

# cp /etc/hosts /etc/hosts.original.bak
# vi /etc/hosts

# diff --unified=0 /etc/hosts.original.bak /etc/hosts
--- /etc/hosts.original.bak     2016-05-07 16:52:59.000000000 -0700
+++ /etc/hosts  2016-05-07 16:55:02.000000000 -0700
@@ -14,0 +15 @@
+127.0.0.1              mygravblog.localhost localhost.

# cp /usr/pkg/etc/httpd/httpd.conf /usr/pkg/etc/httpd/httpd.conf.original.bak
# grep -n ServerName /usr/pkg/etc/httpd/httpd.conf 
202:# ServerName gives the name and port that the server uses to identify itself.
208:#ServerName www.example.com:80

# diff --unified=0 /usr/pkg/etc/httpd/httpd.conf.original.bak /usr/pkg/etc/httpd/httpd.conf
--- /usr/pkg/etc/httpd/httpd.conf.original.bak  2016-05-07 16:56:44.000000000 -0700
+++ /usr/pkg/etc/httpd/httpd.conf       2016-05-07 16:58:17.000000000 -0700
@@ -208 +208 @@
-#ServerName www.example.com:80
+ServerName mygravblog.localhost:80

# /etc/rc.d/apache restart

# which nc
# whereis nc

# export PKG_PATH="http://ftp.NetBSD.org/pub/pkgsrc/packages/$(uname -s)/$(uname -m)/$(uname -r)/All"

# ftp -o - `echo $PKG_PATH` | grep -i netcat
<a href="gnetcat-0.7.1nb3.tgz">gnetcat-0.7.1nb3.tgz</a>                     10-Jan-2016 15:16             33kB
<a href="netcat-1.10nb3.tgz">netcat-1.10nb3.tgz</a>                       05-Jan-2016 13:36             26kB
<a href="netcat-openbsd-20131208.tgz">netcat-openbsd-20131208.tgz</a>              10-Jan-2016 15:05             21kB
<a href="netcat6-1.0nb3.tgz">netcat6-1.0nb3.tgz</a>                       10-Jan-2016 15:05             31kB


# pkg_add -v netcat
# which nc
/usr/pkg/sbin/nc
# whereis nc
/usr/pkg/sbin/nc

# nc -z -v mygravblog.localhost 80
DNS fwd/rev mismatch: mygravblog.localhost != localhost
mygravblog.localhost [127.0.0.1] 80 (http) open


# nc localhost 80
GET / HTTP/1.1
HOST: mygravblog.localhost
<line feed>

** Notes: **
- No OS cursor caret, the connection is open so you just get a blank line waiting for input. 
- Press <enter> twice after the "HOST:".
- To disconnect, press Ctrl-C (which the shell will display as ^C).


OUTPUT:
HTTP/1.1 200 OK
Date: Sun, 08 May 2016 00:56:38 GMT
Server: Apache/2.4.17 (Unix)
Last-Modified: Fri, 01 Jan 2016 16:32:10 GMT
ETag: "2d-5284850171680"
Accept-Ranges: bytes
Content-Length: 45
Content-Type: text/html

<html><body><h1>It works!</h1></body></html>
^C punt!
```

On the host OS, with your web browser, browse to http://localhost:7777, which 
was configured to forward to guest OS's port 80.


```
$ lynx localhost:7777
```

Print from the 'lynx' text web browser:
```
                          It works!
```

```
# /usr/libexec/locate.updatedb 

# locate mod_php5.so
/usr/pkg/lib/httpd/mod_php5.so


# grep -v ^\# /usr/pkg/etc/httpd/httpd.conf | grep LoadModule
LoadModule authn_file_module lib/httpd/mod_authn_file.so
... ... ...
... ... ...
LoadModule alias_module lib/httpd/mod_alias.so


# grep -n mod_alias.so /usr/pkg/etc/httpd/httpd.conf 
165:LoadModule alias_module lib/httpd/mod_alias.so

# vi +165 /usr/pkg/etc/httpd/httpd.conf 

# diff --unified=0 /usr/pkg/etc/httpd/httpd.conf.original.bak /usr/pkg/etc/httpd/httpd.conf
--- /usr/pkg/etc/httpd/httpd.conf.original.bak  2016-05-07 21:00:34.000000000 -0700
+++ /usr/pkg/etc/httpd/httpd.conf       2016-05-07 20:52:47.000000000 -0700
@@ -165,0 +166,4 @@
+LoadModule php5_module lib/httpd/mod_php5.so
+<FilesMatch \.php$>
+    SetHandler application/x-httpd-php
+</FilesMatch>
@@ -208 +212 @@
-#ServerName www.example.com:80
+ServerName mygravblog.localhost:80

# /etc/rc.d/apache restart
Stopping apache.
Waiting for PIDS: 2230.
Starting apache.
[Sat May 07 21:11:50.954964 2016] [:crit] [pid 1947:tid 140187566933760] Apache is running a threaded MPM, but your PHP Module is not compiled to be threadsafe.  You need to recompile PHP.
AH00013: Pre-configuration failed
```

Fix: Switch to the prefork MPM, which does not use threads -=> replace mpm_event_module with mpm_prefork_module:

```
# grep -n mpm /usr/pkg/etc/httpd/httpd.conf 
145:LoadModule mpm_event_module lib/httpd/mod_mpm_event.so
146:#LoadModule mpm_prefork_module lib/httpd/mod_mpm_prefork.so
147:#LoadModule mpm_worker_module lib/httpd/mod_mpm_worker.so
468:#Include etc/httpd/httpd-mpm.conf

# grep -n prefork /usr/pkg/etc/httpd/httpd.conf
146:#LoadModule mpm_prefork_module lib/httpd/mod_mpm_prefork.so

# vi +145 /usr/pkg/etc/httpd/httpd.conf

# diff --unified=0 /usr/pkg/etc/httpd/httpd.conf.original.bak /usr/pkg/etc/httpd/httpd.conf
--- /usr/pkg/etc/httpd/httpd.conf.original.bak  2016-05-07 21:00:34.000000000 -0700
+++ /usr/pkg/etc/httpd/httpd.conf       2016-05-07 21:16:06.000000000 -0700
@@ -145,2 +145,2 @@
-LoadModule mpm_event_module lib/httpd/mod_mpm_event.so
-#LoadModule mpm_prefork_module lib/httpd/mod_mpm_prefork.so
+#LoadModule mpm_event_module lib/httpd/mod_mpm_event.so
+LoadModule mpm_prefork_module lib/httpd/mod_mpm_prefork.so
@@ -165,0 +166,4 @@
+LoadModule php5_module lib/httpd/mod_php5.so
+<FilesMatch \.php$>
+    SetHandler application/x-httpd-php
+</FilesMatch>
@@ -208 +212 @@
-#ServerName www.example.com:80
+ServerName mygravblog.localhost:80
@@ -464 +468 @@
-#Include etc/httpd/httpd-mpm.conf
+Include etc/httpd/httpd-mpm.conf


# /etc/rc.d/apache status
apache is not running.

# /etc/rc.d/apache restart
apache not running? (check /var/run/httpd.pid).
Starting apache.

# /etc/rc.d/apache status
apache is running as pid 1557.
```

Vhost Setup
```
# grep -n vhosts /usr/pkg/etc/httpd/httpd.conf
486:#Include etc/httpd/httpd-vhosts.conf

# vi +486 /usr/pkg/etc/httpd/httpd.conf

# diff --unified=0 /usr/pkg/etc/httpd/httpd.conf.original.bak /usr/pkg/etc/httpd/httpd.conf
--- /usr/pkg/etc/httpd/httpd.conf.original.bak  2016-05-07 21:00:34.000000000 -0700
+++ /usr/pkg/etc/httpd/httpd.conf       2016-05-07 21:24:00.000000000 -0700
@@ -145,2 +145,2 @@
-LoadModule mpm_event_module lib/httpd/mod_mpm_event.so
-#LoadModule mpm_prefork_module lib/httpd/mod_mpm_prefork.so
+#LoadModule mpm_event_module lib/httpd/mod_mpm_event.so
+LoadModule mpm_prefork_module lib/httpd/mod_mpm_prefork.so
@@ -165,0 +166,4 @@
+LoadModule php5_module lib/httpd/mod_php5.so
+<FilesMatch \.php$>
+    SetHandler application/x-httpd-php
+</FilesMatch>
@@ -208 +212 @@
-#ServerName www.example.com:80
+ServerName mygravblog.localhost:80
@@ -464 +468 @@
-#Include etc/httpd/httpd-mpm.conf
+Include etc/httpd/httpd-mpm.conf
@@ -482 +486 @@
-#Include etc/httpd/httpd-vhosts.conf
+Include etc/httpd/httpd-vhosts.conf

# grep -n -i mod_rewrite /usr/pkg/etc/httpd/httpd.conf
170:#LoadModule rewrite_module lib/httpd/mod_rewrite.so

# vi +170 /usr/pkg/etc/httpd/httpd.conf

# diff --unified=0 /usr/pkg/etc/httpd/httpd.conf.original.bak /usr/pkg/etc/httpd/httpd.conf
--- /usr/pkg/etc/httpd/httpd.conf.original.bak  2016-05-07 21:00:34.000000000 -0700
+++ /usr/pkg/etc/httpd/httpd.conf       2016-05-07 23:22:39.000000000 -0700
@@ -145,2 +145,2 @@
-LoadModule mpm_event_module lib/httpd/mod_mpm_event.so
-#LoadModule mpm_prefork_module lib/httpd/mod_mpm_prefork.so
+#LoadModule mpm_event_module lib/httpd/mod_mpm_event.so
+LoadModule mpm_prefork_module lib/httpd/mod_mpm_prefork.so
@@ -166 +166,5 @@
-#LoadModule rewrite_module lib/httpd/mod_rewrite.so
+LoadModule php5_module lib/httpd/mod_php5.so
+<FilesMatch \.php$>
+    SetHandler application/x-httpd-php
+</FilesMatch>
+LoadModule rewrite_module lib/httpd/mod_rewrite.so
@@ -208 +212 @@
-#ServerName www.example.com:80
+ServerName mygravblog.localhost:80
@@ -464 +468 @@
-#Include etc/httpd/httpd-mpm.conf
+Include etc/httpd/httpd-mpm.conf
@@ -482 +486 @@
-#Include etc/httpd/httpd-vhosts.conf
+Include etc/httpd/httpd-vhosts.conf


# grep -n DocumentRoot /usr/pkg/etc/httpd/httpd.conf
232:# DocumentRoot: The directory out of which you will serve your
236:DocumentRoot "/usr/pkg/share/httpd/htdocs"
336:    # access content that does not live under the DocumentRoot.


# cp /usr/pkg/etc/httpd/httpd-vhosts.conf /usr/pkg/etc/httpd/httpd-vhosts.conf.original.bak
# vi /usr/pkg/etc/httpd/httpd-vhosts.conf 

# tail -13 /usr/pkg/etc/httpd/httpd-vhosts.conf
    <Directory /var/www/grav>
        Options Indexes FollowSymLinks MultiViews
        DirectoryIndex index.php
        AllowOverride all
        Require all granted
    </Directory>

    ServerAdmin webmaster@dummy-host2.example.com
    DocumentRoot "/var/www/grav"
    ServerName mygravblog.localhost
    ErrorLog "/var/log/httpd/mygravblog-error_log"
    CustomLog "/var/log/httpd/mygravblog-access_log" common
</VirtualHost>


# ls -ld /var/www
drwxr-xr-x  2 root  wheel  512 Sep 25  2015 /var/www

# mkdir -p /var/www/grav
# ls -ld /var/www/grav
drwxr-xr-x  2 root  wheel  512 May  7 21:38 /var/www/grav

# printf "My Grav Blog\n" > /var/www/grav/index.html

# /etc/rc.d/apache restart

# nc localhost 80
GET / HTTP/1.1
HOST: mygravblog.localhost

HTTP/1.1 200 OK
Date: Sun, 08 May 2016 04:47:57 GMT
Server: Apache/2.4.17 (Unix) PHP/5.6.19
Last-Modified: Sun, 08 May 2016 04:41:52 GMT
ETag: "d-5324d4f971480"
Accept-Ranges: bytes
Content-Length: 13
Content-Type: text/html

My Grav Blog
^C punt!
```

Grav Installation 
```
# rm -i /var/www/grav/index.html
remove '/var/www/grav/index.html'? y

# ftp https://github.com/getgrav/grav/releases/download/1.0.10/grav-admin-v1.0.10.zip

# printf "ftp saved it under name octet-stream. \n\n"
ftp saved it under name octet-stream. 

# mv octet-stream grav-admin-v1.0.10.zip
# file grav-admin-v1.0.10.zip 
grav-admin-v1.0.10.zip: Zip archive data, at least v1.0 to extract

# unzip grav-admin-v1.0.10.zip 
... ... ...
... ... ...

# rmdir /var/www/grav/
# mv grav-admin /var/www/grav

# groups dusko
users wheel

# find /var/www/grav -type d -exec chmod 0775 {} \;
# find /var/www/grav -type f -exec chmod 0664 {} \;
# find /var/www/grav/bin -type f -exec chmod 0774 {} \;

# grep apache /etc/passwd
www:*:1001:1000:apache www user:/nonexistent:/sbin/nologin

# grep 1000 /etc/group 
www:*:1000:


# chown -R dusko:www /var/www/grav

# printf "In order to run Grav's GPM (Grav Package Manager), you need to add your user to the group of the user running the web server.  \n\n"
In order to run Grav's GPM (Grav Package Manager), you need to add your user to the group of the user running the web server.  

# usermod -G www dusko
# groups dusko
users wheel www

# exit
$ cd /var/www/grav 

$ bin/gpm index
PHP Fatal error:  Call to undefined function Grav\Common\Config\json_encode() in /var/www/grav/system/src/Grav/Common/Config/CompiledBase.php on line 112
PHP Stack trace:
PHP   1. {main}() /var/www/grav/bin/gpm:0
PHP   2. Pimple\Container->offsetGet() /var/www/grav/bin/gpm:41
... ... ...
... ... ...


$ su
Password:

# ftp -o - `echo $PKG_PATH` | grep -i php | grep json 
<a href="php55-json-5.5.33.tgz">php55-json-5.5.33.tgz</a>                    08-Mar-2016 20:30             19kB
<a href="php56-json-5.6.19.tgz">php56-json-5.6.19.tgz</a>                    08-Mar-2016 20:38             18kB
<a href="php70-json-7.0.4.tgz">php70-json-7.0.4.tgz</a>                     09-Mar-2016 21:49             20kB


# pkg_add -v php56-json

# vi +882 /usr/pkg/etc/php.ini

# diff --unified=0 /usr/pkg/etc/php.ini.original.bak /usr/pkg/etc/php.ini
--- /usr/pkg/etc/php.ini.original.bak   2016-05-07 12:58:30.000000000 -0700
+++ /usr/pkg/etc/php.ini        2016-05-07 22:47:58.000000000 -0700
@@ -867,0 +868,16 @@
+;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
+; Dynamic Extensions required for Grav ;
+;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
+extension=gd.so
+extension=curl.so
+extension=mcrypt.so
+extension=zip.so
+extension=mbstring.so
+extension=xmlrpc.so
+extension=apcu.so
+zend_extension=opcache.so
+extension=xcache.so
+zend_extension=xdebug.so
+extension=yaml.so
+extension=json.so
+

# grep -n "date.timezone" /usr/pkg/etc/php.ini
940:; http://php.net/date.timezone
941:;date.timezone =

# vi +941 /usr/pkg/etc/php.ini

# diff --unified=0 /usr/pkg/etc/php.ini.original.bak /usr/pkg/etc/php.ini
--- /usr/pkg/etc/php.ini.original.bak   2016-05-07 12:58:30.000000000 -0700
+++ /usr/pkg/etc/php.ini        2016-05-08 00:01:23.000000000 -0700
@@ -867,0 +868,16 @@
+;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
+; Dynamic Extensions required for Grav ;
+;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
+extension=gd.so
+extension=curl.so
+extension=mcrypt.so
+extension=zip.so
+extension=mbstring.so
+extension=xmlrpc.so
+extension=apcu.so
+zend_extension=opcache.so
+extension=xcache.so
+zend_extension=xdebug.so
+extension=yaml.so
+extension=json.so
+
@@ -925 +941 @@
-;date.timezone =
+date.timezone = "America/Vancouver" 

# exit
$ pwd
/var/www/grav

$ bin/gpm index

PHP Warning:  Invalid argument supplied for foreach() in /var/www/grav/system/src/Grav/Console/Gpm/IndexCommand.php on line 110
PHP Stack trace:
PHP   1. {main}() /var/www/grav/bin/gpm:0
PHP   2. Symfony\Component\Console\Application->run() /var/www/grav/bin/gpm:57
... ... ...
... ... ...


# sed -n 110,110p /var/www/grav/system/src/Grav/Console/Gpm/IndexCommand.php
        foreach ($data as $type => $packages) {


# cp /var/www/grav/system/src/Grav/Console/Gpm/IndexCommand.php /var/www/grav/system/src/Grav/Console/Gpm/IndexCommand.php.original.bak
# vi +110 /var/www/grav/system/src/Grav/Console/Gpm/IndexCommand.php

# diff --unified=0  /var/www/grav/system/src/Grav/Console/Gpm/IndexCommand.php.original.bak /var/www/grav/system/src/Grav/Console/Gpm/IndexCommand.php
--- /var/www/grav/system/src/Grav/Console/Gpm/IndexCommand.php.original.bak     2016-05-07 22:55:14.000000000 -0700
+++ /var/www/grav/system/src/Grav/Console/Gpm/IndexCommand.php  2016-05-07 22:55:44.000000000 -0700
@@ -110 +110 @@
-        foreach ($data as $type => $packages) {
+        foreach ((array) $data as $type => $packages) {


# exit
$ bin/gpm index

You can either get more informations about a package by typing:
    bin/gpm info <package>

Or you can install a package by typing:
    bin/gpm install <package>


# find /var/www/grav/cache -type d -exec chmod 0775 {} \;
```

* Troubleshooting

Sorry, something went terribly wrong!
E_ERROR - Class 'DOMDocument' not found
For further details please review your logs/ folder, or enable displaying of errors in your system configuration.

```
# cat /var/www/grav/logs/grav.log 
[2016-05-08 00:34:09] grav.WARNING: Plugin 'jscomments' enabled but not found! Try clearing cache with `bin/grav clear-cache` [] []
[2016-05-08 00:34:10] grav.CRITICAL: Class 'DOMDocument' not found - Trace: #0 /var/www/grav/vendor/filp/whoops/src/Whoops/Run.php(357): Whoops\Run->handleError(1, 'Class 'DOMDocum...', '/var/www/grav/s...', 67) #1 [internal function]: Whoops\Run->handleShutdown() #2 {main} [] []

# php -i | grep -i -w dom
Configure Command =>  './configure'  '--with-config-file-path=/usr/pkg/etc' '--with-config-file-scan-dir=/usr/pkg/etc/php.d' '--sysconfdir=/usr/pkg/etc' '--localstatedir=/var' '--with-regex=system' '--without-mysql' '--without-iconv' '--without-pear' '--disable-posix' '--disable-dom' '--disable-opcache' '--disable-pdo' '--disable-json' '--enable-cgi' '--enable-mysqlnd' '--enable-xml' '--with-libxml-dir=/usr/pkg' '--enable-ipv6' '--with-openssl=/usr' '--without-readline' '--prefix=/usr/pkg' '--build=x86_64--netbsd' '--host=x86_64--netbsd' '--mandir=/usr/pkg/man' 'build_alias=x86_64--netbsd' 'host_alias=x86_64--netbsd' 'CC=gcc' 'CFLAGS=-O2 '-pthread' '-I/usr/pkg/include' '-I/usr/include'' 'LDFLAGS=-L/usr/pkg/lib '-Wl,-R/usr/pkg/lib' '-L/usr/lib' '-Wl,-R/usr/lib' '-pthread'' 'LIBS=' 'CPPFLAGS=-I/usr/pkg/include '-I/usr/include'' 'CXX=c++' 'CXXFLAGS=-O2 '-pthread' '-I/usr/pkg/include' '-I/usr/include'' 'CXXCPP=cpp'


# php -i | grep -n "disable-dom"
6:Configure Command =>  './configure'  '--with-config-file-path=/usr/pkg/etc' '--with-config-file-scan-dir=/usr/pkg/etc/php.d' '--sysconfdir=/usr/pkg/etc' '--localstatedir=/var' '--with-regex=system' '--without-mysql' '--without-iconv' '--without-pear' '--disable-posix' '--disable-dom' '--disable-opcache' '--disable-pdo' '--disable-json' '--enable-cgi' '--enable-mysqlnd' '--enable-xml' '--with-libxml-dir=/usr/pkg' '--enable-ipv6' '--with-openssl=/usr' '--without-readline' '--prefix=/usr/pkg' '--build=x86_64--netbsd' '--host=x86_64--netbsd' '--mandir=/usr/pkg/man' 'build_alias=x86_64--netbsd' 'host_alias=x86_64--netbsd' 'CC=gcc' 'CFLAGS=-O2 '-pthread' '-I/usr/pkg/include' '-I/usr/include'' 'LDFLAGS=-L/usr/pkg/lib '-Wl,-R/usr/pkg/lib' '-L/usr/lib' '-Wl,-R/usr/lib' '-pthread'' 'LIBS=' 'CPPFLAGS=-I/usr/pkg/include '-I/usr/include'' 'CXX=c++' 'CXXFLAGS=-O2 '-pthread' '-I/usr/pkg/include' '-I/usr/include'' 'CXXCPP=cpp'


# ftp -o - `echo $PKG_PATH` | grep -i php | grep -i dom
<a href="php55-dom-5.5.33.tgz">php55-dom-5.5.33.tgz</a>                     08-Mar-2016 20:21             72kB
<a href="php56-dom-5.6.19.tgz">php56-dom-5.6.19.tgz</a>                     08-Mar-2016 20:33             73kB
<a href="php70-dom-7.0.4.tgz">php70-dom-7.0.4.tgz</a>                      09-Mar-2016 21:47             70kB


# pkg_add -v php56-dom

# vi +883 /usr/pkg/etc/php.ini

# diff --unified=0 /usr/pkg/etc/php.ini.original.bak /usr/pkg/etc/php.ini
--- /usr/pkg/etc/php.ini.original.bak   2016-05-07 12:58:30.000000000 -0700
+++ /usr/pkg/etc/php.ini        2016-05-08 00:49:52.000000000 -0700
@@ -867,0 +868,17 @@
+;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
+; Dynamic Extensions required for Grav ;
+;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
+extension=gd.so
+extension=curl.so
+extension=mcrypt.so
+extension=zip.so
+extension=mbstring.so
+extension=xmlrpc.so
+extension=apcu.so
+zend_extension=opcache.so
+extension=xcache.so
+zend_extension=xdebug.so
+extension=yaml.so
+extension=json.so
+extension=dom.so
+
@@ -925 +942 @@
-;date.timezone =
+date.timezone = "America/Vancouver" 

# /etc/rc.d/apache restart
Stopping apache.
Waiting for PIDS: 6995.
Starting apache.
```

### Browser check on the host ###

* Add the following line ```127.0.0.1   mygravablog.localhost``` to ```/etc/hosts```.
* With your web browser, navigate to ```http://mygravblog.localhost:7777/```.

