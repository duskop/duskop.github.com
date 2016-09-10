---
layout: post
title:  Install Nginx and Configure PHP-FPM for Grav on OS X 10.9
---


### Install Nginx ###

```
$ brew update
$ brew upgrade
$ brew doctor
```


```
$ brew install nginx
< ... [ snip ] ... >

Docroot is: /usr/local/var/www

The default port has been set in /usr/local/etc/nginx/nginx.conf to 8080 so that
nginx can run without sudo.

nginx will load all files in /usr/local/etc/nginx/servers/.

To have launchd start nginx now and restart at login:
  brew services start nginx
Or, if you don't want/need a background service you can just run:
  nginx
==> Summary
   /usr/local/Cellar/nginx/1.10.1: 7 files, 972.3K
```


```
$ brew services status nginx
==> Tapping homebrew/services
Cloning into '/usr/local/Library/Taps/homebrew/homebrew-services'...
remote: Counting objects: 7, done.
remote: Compressing objects: 100% (6/6), done.
remote: Total 7 (delta 0), reused 3 (delta 0), pack-reused 0
Unpacking objects: 100% (7/7), done.
Checking connectivity... done.
Tapped 0 formulae (32 files, 46K)
Error: Unknown command `status`
usage: [sudo] brew services [--help] <command> [<formula>|--all]

Small wrapper around `launchctl` for supported formulae, commands available:
   cleanup Get rid of stale services and unused plists
   list    List all services managed by `brew services`
   restart Gracefully restart service(s)
   start   Start service(s)
   stop    Stop service(s)

Options, sudo and paths:

  sudo   When run as root, operates on /Library/LaunchDaemons (run at boot!)
  Run at boot:  /Library/LaunchDaemons
  Run at login: /Users/dusko/Library/LaunchAgents
```


### Configure Nginx Run on Port 80 or Another System Port (a.k.a Well-Known Port) ###

System Ports (a.k.a Well-Known Ports): The ports from 1 through 1023, which are intended for systems processes running as the superuser.   


Start Nginx for the first time.

```
$ brew services start nginx
```

When you start nginx for the first time, the system displays the following dialog box:      

Do you want the application "nginx" to accept incoming network connections?    
Clicking Deny may limit the application’s behavior. This setting can be changed in the Firewall pane of Security & Privacy preferences.   

Click "Allow" button.    


When installed with Homebrew on OS X, nginx by default runs on port 8080.      

```
$ grep -n -i listen /usr/local/etc/nginx/nginx.conf
36:        listen       8080;
57:        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
63:        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
85:    #    listen       8000;
86:    #    listen       somename:8080;
99:    #    listen       443 ssl;
```



Change the port nginx listens to from 8080 to 80.      

```
$ sed -n 36,36p /usr/local/etc/nginx/nginx.conf
        listen       8080;

$ sed -i.bak '36s/8080/80/' /usr/local/etc/nginx/nginx.conf

$ sed -n 36,36p /usr/local/etc/nginx/nginx.conf
        listen       80;

$ ls -lhrt /usr/local/etc/nginx | tail -2
-rw-r--r--  1 dusko  admin   2.6K  5 Sep 12:08 nginx.conf.bak
-rw-r--r--  1 dusko  admin   2.6K  5 Sep 12:09 nginx.conf

$ diff --unified=0 /usr/local/etc/nginx/nginx.conf.bak /usr/local/etc/nginx/nginx.conf
--- /usr/local/etc/nginx/nginx.conf.bak	2016-09-05 12:08:04.000000000 -0700
+++ /usr/local/etc/nginx/nginx.conf	2016-09-05 12:09:31.000000000 -0700
@@ -36 +36 @@
-        listen       8080;
+        listen       80;
```


Since nginx is now configured with a system port, you will need to start nginx service as the superuser.


```
$ brew services list
Name  Status  User Plist
nginx stopped      

$ brew services start nginx
==> Successfully started `nginx` (label: homebrew.mxcl.nginx)

$ brew services list
Name  Status  User  Plist
nginx started dusko /Users/dusko/Library/LaunchAgents/homebrew.mxcl.nginx.plist
```


However, when checked whether nginx is running and listening to port 80:

```
$ ps aux | grep nginx

$ sudo lsof -i -P | grep -i listen | egrep -w '.80|:80'

$ netstat -an | egrep -w '.80|:80'
```


For next step, stop nginx.

```
$ brew services list
Name  Status  User  Plist
nginx started dusko /Users/dusko/Library/LaunchAgents/homebrew.mxcl.nginx.plist

$ brew services stop nginx
Stopping `nginx`... (might take a while)
==> Successfully stopped `nginx` (label: homebrew.mxcl.nginx)

$ brew services list
Name  Status  User Plist
nginx stopped      
```


So, when nginx is using a system port, you have to start it as root.

```
$ sudo brew services start nginx
==> Successfully started `nginx` (label: homebrew.mxcl.nginx)

$ brew services list
Name  Status  User Plist
nginx started root /Library/LaunchDaemons/homebrew.mxcl.nginx.plist

$ ps aux | grep nginx
nobody          26810   0.0  0.0  2464232   1000   ??  S    12:43pm   0:00.00 nginx: worker process  
root            26808   0.0  0.0  2466052   2088   ??  Ss   12:43pm   0:00.02 nginx: master process /usr/local/opt/nginx/bin/nginx -g daemon off;  

$ sudo lsof -i -P | grep -i listen | egrep -w '.80|:80'
nginx     26808           root    6u  IPv4 0x4dea8798d127c69      0t0  TCP *:80 (LISTEN)
nginx     26810         nobody    6u  IPv4 0x4dea8798d127c69      0t0  TCP *:80 (LISTEN)

$ netstat -an | egrep -w '.80|:80'
tcp4       0      0  *.80                   *.*                    LISTEN     
```


### Configure Nginx to Give Permissions to Your Regular User Account ###


```
$ grep -n -i user /usr/local/etc/nginx/nginx.conf
2:#user  nobody;
21:    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
23:    #                  '"$http_user_agent" "$http_x_forwarded_for"';
```


```
$ head  /usr/local/etc/nginx/nginx.conf

#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;

```

As per [sed on OSX insert at a certain line](http://unix.stackexchange.com/questions/52131/sed-on-osx-insert-at-a-certain-line):      

By the POSIX standard, a needs to be followed by a \, then that needs to be followed by a newline, then the remainder of the next line will be inserted, until a newline or the end of the -e clause is encountered.


Test:

```
$ sed -e $'2i\\\n# Sep 8, 2016\\\n#-#user  nobody;\\\n#+user  dusko staff;\\\nuser  dusko staff;' /usr/local/etc/nginx/nginx.conf | head

# Sep 8, 2016
#-#user  nobody;
#+user  dusko staff;
user  dusko staff;
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
```


Append lines:

```
$ sed -i .bak -e $'2i\\\n# Sep 8, 2016\\\n#-#user  nobody;\\\n#+user  dusko staff;\\\nuser  dusko staff;' /usr/local/etc/nginx/nginx.conf 
```


Check/confirm changes:

```
$ diff --unified=0 /usr/local/etc/nginx/nginx.conf.bak /usr/local/etc/nginx/nginx.conf
--- /usr/local/etc/nginx/nginx.conf.bak	2016-09-05 12:09:31.000000000 -0700
+++ /usr/local/etc/nginx/nginx.conf	2016-09-05 14:35:58.000000000 -0700
@@ -1,0 +2,4 @@
+# Sep 8, 2016
+#-#user  nobody;
+#+user  dusko staff;
+user  dusko staff;
```


Delete the original "#user  nobody;" line. Test first.   

```
$ head /usr/local/etc/nginx/nginx.conf

# Sep 8, 2016
#-#user  nobody;
#+user  dusko staff;
user  dusko staff;
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
```


```
$ sed '6d' /usr/local/etc/nginx/nginx.conf | head

# Sep 8, 2016
#-#user  nobody;
#+user  dusko staff;
user  dusko staff;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;
```


Actual line deletion.

```
$ sed -i .bak '6d' /usr/local/etc/nginx/nginx.conf 
```


Check:

```
$ head /usr/local/etc/nginx/nginx.conf

# Sep 8, 2016
#-#user  nobody;
#+user  dusko staff;
user  dusko staff;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
```


```
$ diff --unified=0 /usr/local/etc/nginx/nginx.conf.bak /usr/local/etc/nginx/nginx.conf
--- /usr/local/etc/nginx/nginx.conf.bak	2016-09-05 14:35:58.000000000 -0700
+++ /usr/local/etc/nginx/nginx.conf	2016-09-05 14:42:42.000000000 -0700
@@ -6 +5,0 @@
-#user  nobody;
```


### A Side Note: Nginx Default Home Directory ###

```
$ grep -B 2 -n root /usr/local/etc/nginx/nginx.conf
45-
46-        location / {
47:            root   html;
--
... [ snip ] ...

```


The location of the default nginx home directory.    

```
$ which nginx
/usr/local/bin/nginx

$ ls -ld /usr/local/bin/nginx 
lrwxr-xr-x  1 dusko  admin  32  5 Sep 09:46 /usr/local/bin/nginx -> ../Cellar/nginx/1.10.1/bin/nginx

$ ls -lh /usr/local/bin/nginx 
lrwxr-xr-x  1 dusko  admin    32B  5 Sep 09:46 /usr/local/bin/nginx -> ../Cellar/nginx/1.10.1/bin/nginx

$ ls -lh /usr/local/Cellar/nginx/
total 0
drwxr-xr-x  10 dusko  admin   340B  5 Sep 09:46 1.10.1

$ ls -alh /usr/local/Cellar/nginx/1.10.1/
total 560
drwxr-xr-x  10 dusko  admin   340B  5 Sep 09:46 .
drwxr-xr-x   3 dusko  admin   102B  5 Sep 09:46 ..
-rw-r--r--   1 dusko  admin   257K 31 May 06:47 CHANGES
-rw-r--r--   1 dusko  admin   496B  5 Sep 09:46 INSTALL_RECEIPT.json
-rw-r--r--   1 dusko  admin   1.4K 31 May 06:47 LICENSE
-rw-r--r--   1 dusko  admin    49B 31 May 06:47 README
drwxr-xr-x   3 dusko  admin   102B 31 May 06:47 bin
-rw-r--r--   1 dusko  admin   571B  5 Sep 09:46 homebrew.mxcl.nginx.plist
lrwxr-xr-x   1 dusko  admin    16B  5 Sep 09:46 html -> ../../../var/www
drwxr-xr-x   3 dusko  admin   102B 31 May 06:47 share

$ ls -alh /usr/local/Cellar/nginx/1.10.1/html
lrwxr-xr-x  1 dusko  admin    16B  5 Sep 09:46 /usr/local/Cellar/nginx/1.10.1/html -> ../../../var/www

$ ls -alh /usr/local/var/
total 0
drwxr-xr-x   5 dusko  admin   170B  5 Sep 09:46 .
drwxr-xr-x  22 dusko  admin   748B  5 Sep 09:46 ..
drwxr-xr-x   3 dusko  admin   102B  5 Sep 09:46 log
drwxr-xr-x   3 dusko  admin   102B  5 Sep 11:06 run
drwxr-xr-x   4 dusko  admin   136B 31 May 06:47 www

$ ls -alh /usr/local/var/www/
total 16
drwxr-xr-x  4 dusko  admin   136B 31 May 06:47 .
drwxr-xr-x  5 dusko  admin   170B  5 Sep 09:46 ..
-rw-r--r--  1 dusko  admin   537B 31 May 06:47 50x.html
-rw-r--r--  1 dusko  admin   612B 31 May 06:47 index.html
```


```
$ cat /usr/local/var/www/index.html 
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

```


### Activate Server Logs ###

```
$ grep -A 3 -n -i log /usr/local/etc/nginx/nginx.conf
8:#error_log  logs/error.log;
9:#error_log  logs/error.log  notice;
10:#error_log  logs/error.log  info;
11-
12:#pid        logs/nginx.pid;
13-
14-
15-events {
--
24:    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
25-    #                  '$status $body_bytes_sent "$http_referer" '
26-    #                  '"$http_user_agent" "$http_x_forwarded_for"';
27-
--
28:    #access_log  logs/access.log  main;
29-
30-    sendfile        on;
31-    #tcp_nopush     on;
--
44:        #access_log  logs/host.access.log  main;
45-
46-        location / {
47-            root   html;
```


```
$ sed -i .bak '8s/#//' /usr/local/etc/nginx/nginx.conf

$ diff --unified=0 /usr/local/etc/nginx/nginx.conf.bak /usr/local/etc/nginx/nginx.conf
--- /usr/local/etc/nginx/nginx.conf.bak	2016-09-05 14:42:42.000000000 -0700
+++ /usr/local/etc/nginx/nginx.conf	2016-09-05 16:29:37.000000000 -0700
@@ -8 +8 @@
-#error_log  logs/error.log;
+error_log  logs/error.log;
```


```
$ sed -n 24,28p /usr/local/etc/nginx/nginx.conf
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

$ sed -i .bak '24,28s/#//' /usr/local/etc/nginx/nginx.conf

$ diff --unified=0 /usr/local/etc/nginx/nginx.conf.bak /usr/local/etc/nginx/nginx.conf
--- /usr/local/etc/nginx/nginx.conf.bak	2016-09-05 17:02:08.000000000 -0700
+++ /usr/local/etc/nginx/nginx.conf	2016-09-05 17:02:55.000000000 -0700
@@ -24,3 +24,3 @@
-    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
-    #                  '$status $body_bytes_sent "$http_referer" '
-    #                  '"$http_user_agent" "$http_x_forwarded_for"';
+    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
+                      '$status $body_bytes_sent "$http_referer" '
+                      '"$http_user_agent" "$http_x_forwarded_for"';
@@ -28 +28 @@
-    #access_log  logs/access.log  main;
+    access_log  logs/access.log  main;
```


```
$ sed -n 44p /usr/local/etc/nginx/nginx.conf
        #access_log  logs/host.access.log  main;

$ sed -i .bak '44s/#//' /usr/local/etc/nginx/nginx.conf

$ diff --unified=0 /usr/local/etc/nginx/nginx.conf.bak /usr/local/etc/nginx/nginx.conf
--- /usr/local/etc/nginx/nginx.conf.bak	2016-09-05 16:36:24.000000000 -0700
+++ /usr/local/etc/nginx/nginx.conf	2016-09-05 16:39:19.000000000 -0700
@@ -44 +44 @@
-        #access_log  logs/host.access.log  main;
+        access_log  logs/host.access.log  main;
```


Add error_log to the "server" (host) section as well.    
Note that in the following sed command you have to escape eight spaces.      

```
$ sed -i .bak $'45i\\\n\        error_log   logs/host.error.log;\\\n' /usr/local/etc/nginx/nginx.conf 

$ diff --unified=0 /usr/local/etc/nginx/nginx.conf.bak /usr/local/etc/nginx/nginx.conf
--- /usr/local/etc/nginx/nginx.conf.bak	2016-09-05 16:39:19.000000000 -0700
+++ /usr/local/etc/nginx/nginx.conf	2016-09-05 16:46:45.000000000 -0700
@@ -44,0 +45 @@
+        error_log   logs/host.error.log;
```


Test the configuration file.

```
$ sudo nginx -t
nginx: the configuration file /usr/local/etc/nginx/nginx.conf syntax is ok
nginx: [emerg] open() "/usr/local/Cellar/nginx/1.10.1/logs/error.log" failed (2: No such file or directory)
nginx: configuration file /usr/local/etc/nginx/nginx.conf test failed
```


```
$ which nginx
/usr/local/bin/nginx

$ ls -lh /usr/local/bin/nginx 
lrwxr-xr-x  1 dusko  admin    32B  5 Sep 09:46 /usr/local/bin/nginx -> ../Cellar/nginx/1.10.1/bin/nginx

$ ls -lh /usr/local/Cellar/nginx/1.10.1/
total 560
-rw-r--r--  1 dusko  admin   257K 31 May 06:47 CHANGES
-rw-r--r--  1 dusko  admin   496B  5 Sep 09:46 INSTALL_RECEIPT.json
-rw-r--r--  1 dusko  admin   1.4K 31 May 06:47 LICENSE
-rw-r--r--  1 dusko  admin    49B 31 May 06:47 README
drwxr-xr-x  3 dusko  admin   102B 31 May 06:47 bin
-rw-r--r--  1 dusko  admin   571B  5 Sep 09:46 homebrew.mxcl.nginx.plist
lrwxr-xr-x  1 dusko  admin    16B  5 Sep 09:46 html -> ../../../var/www
drwxr-xr-x  3 dusko  admin   102B 31 May 06:47 share

$ ls -lh /usr/local/Cellar/nginx/1.10.1/html
lrwxr-xr-x  1 dusko  admin    16B  5 Sep 09:46 /usr/local/Cellar/nginx/1.10.1/html -> ../../../var/www

$ ls -lh /usr/local/var/
total 0
drwxr-xr-x  3 dusko  admin   102B  5 Sep 09:46 log
drwxr-xr-x  4 dusko  admin   136B  5 Sep 17:11 run
drwxr-xr-x  5 dusko  admin   170B  5 Sep 16:05 www

$ ls -lh /usr/local/var/log/
total 0
drwxr-xr-x  4 dusko  admin   136B  5 Sep 10:32 nginx

$ ls -lh /usr/local/var/log/nginx/
total 16
-rw-r--r--  1 dusko  admin   1.5K  5 Sep 16:20 access.log
-rw-r--r--  1 dusko  admin   1.6K  5 Sep 17:12 error.log
```


```
$ ln -s /usr/local/var/log/nginx /usr/local/Cellar/nginx/1.10.1/logs
```


```
$ ls -lh /usr/local/Cellar/nginx/1.10.1/
total 568
-rw-r--r--  1 dusko  admin   257K 31 May 06:47 CHANGES
-rw-r--r--  1 dusko  admin   496B  5 Sep 09:46 INSTALL_RECEIPT.json
-rw-r--r--  1 dusko  admin   1.4K 31 May 06:47 LICENSE
-rw-r--r--  1 dusko  admin    49B 31 May 06:47 README
drwxr-xr-x  3 dusko  admin   102B 31 May 06:47 bin
-rw-r--r--  1 dusko  admin   571B  5 Sep 09:46 homebrew.mxcl.nginx.plist
lrwxr-xr-x  1 dusko  admin    16B  5 Sep 09:46 html -> ../../../var/www
lrwxr-xr-x  1 dusko  admin    24B  5 Sep 17:22 logs -> /usr/local/var/log/nginx
drwxr-xr-x  3 dusko  admin   102B 31 May 06:47 share
```


```
$ sudo nginx -t
Password:
nginx: the configuration file /usr/local/etc/nginx/nginx.conf syntax is ok
nginx: configuration file /usr/local/etc/nginx/nginx.conf test is successful
```


### Configure PHP-FPM ###

I previously installed Xcode 7.3.1 and Command Line Tools for Xcode. -- PHP version on my OS X 10.11 El Capitan is 5.5.36:      

```
$ which php
/usr/bin/php

$ php --version
PHP 5.5.36 (cli) (built: May 29 2016 01:07:06) 
Copyright (c) 1997-2015 The PHP Group
Zend Engine v2.5.0, Copyright (c) 1998-2015 Zend Technologies

$ php --info | wc -l 
     889

$ php --info | grep -i fpm
```


List all PHP compiled in modules.     

```
$ php --modules | grep -i fpm
```


```
$ locate fpm 
/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.11.sdk/usr/share/man/man8/php-fpm.8
/usr/sbin/php-fpm
/usr/share/man/man8/php-fpm.8
/usr/share/php/php/fpm
/usr/share/php/php/fpm/status.html
```


```
$ php-fpm --info | grep -i fpm
Server API => FPM/FastCGI
php-fpm => active
fpm.config => no value => no value
_ => /usr/sbin/php-fpm
_SERVER["_"] => /usr/sbin/php-fpm
_ENV["_"] => /usr/sbin/php-fpm
```


```
$ locate php | grep ini
/private/etc/php.ini.default
/usr/include/php/Zend/zend_ini.h
/usr/include/php/Zend/zend_ini_parser.h
/usr/include/php/Zend/zend_ini_scanner.h
/usr/include/php/Zend/zend_ini_scanner_defs.h
/usr/include/php/main/php_ini.h
```


You need to adjust "pid" and "error_log" for php-fpm. -- Note: "php-fpm -e" is a command for generating extended information for debugger/profiler.   

```
$ php-fpm -e
[05-Sep-2016 21:27:46] ERROR: failed to open configuration file '/private/etc/php-fpm.conf': No such file or directory (2)
[05-Sep-2016 21:27:46] ERROR: failed to load configuration file '/private/etc/php-fpm.conf'
[05-Sep-2016 21:27:46] ERROR: FPM initialization failed

$ sudo php-fpm -e
No log handling enabled - using stderr logging
Created directory: /var/db/net-snmp
Created directory: /var/db/net-snmp/mib_indexes
[05-Sep-2016 21:27:52] ERROR: failed to open configuration file '/private/etc/php-fpm.conf': No such file or directory (2)
[05-Sep-2016 21:27:52] ERROR: failed to load configuration file '/private/etc/php-fpm.conf'
[05-Sep-2016 21:27:52] ERROR: FPM initialization failed
```


```
$ sudo cp /private/etc/php-fpm.conf.default /private/etc/php-fpm.conf
```


```
$ sudo php-fpm -e
[05-Sep-2016 21:28:57] ERROR: failed to open error_log (/usr/var/log/php-fpm.log): No such file or directory (2)
[05-Sep-2016 21:28:57] ERROR: failed to post process the configuration
[05-Sep-2016 21:28:57] ERROR: FPM initialization failed
```


As per [Can't write to /usr even with sudo](http://apple.stackexchange.com/questions/237778/cant-write-to-usr-even-with-sudo) and [What is the "rootless" feature in El Capitan, really?](http://apple.stackexchange.com/questions/193368/what-is-the-rootless-feature-in-el-capitan-really), the "usr" directory is protected by SIP (System Integrity Protection), a feature introduced with OS X 10.10.    


```
$ ls -ld /usr
drwxr-xr-x@ 13 root  wheel  442 13 Aug 15:54 /usr

$ ls -ld /usr/var
ls: /usr/var: No such file or directory

$ sudo mkdir -p /usr/var/log
Password:
mkdir: /usr/var/log: Operation not permitted
```


```
$ grep -n pid /private/etc/php-fpm.conf
25:;pid = run/php-fpm.pid
310:;   pid                  - the PID of the process;
334:;   pid:                  31330

$ sudo sed -i .bak -e $'26i\\\npid = /usr/local/var/run/php-fpm.pid\\\n' /private/etc/php-fpm.conf
Password:

$ diff --unified=0 /private/etc/php-fpm.conf.bak /private/etc/php-fpm.conf
--- /private/etc/php-fpm.conf.bak	2016-09-05 21:28:49.000000000 -0700
+++ /private/etc/php-fpm.conf	2016-09-05 22:36:50.000000000 -0700
@@ -25,0 +26 @@
+pid = /usr/local/var/run/php-fpm.pid
```


```
$ grep -n error_log php-fpm.conf
32:;error_log = log/php-fpm.log
467:;       (error_log, sessions.save_path, ...).
530:;php_admin_value[error_log] = /var/log/fpm-php.www.log

$ sudo sed -i .bak -e $'34i\\\nerror_log = /usr/local/var/log/php-fpm.log\\\n' /private/etc/php-fpm.conf

$ diff --unified=0 /private/etc/php-fpm.conf.bak /private/etc/php-fpm.conf
--- /private/etc/php-fpm.conf.bak	2016-09-05 22:44:33.000000000 -0700
+++ /private/etc/php-fpm.conf	2016-09-05 22:46:35.000000000 -0700
@@ -33,0 +34 @@
+error_log = /usr/local/var/log/php-fpm.log
```


```
$ ls -lhrt /usr/local/var/log/
total 8
drwxr-xr-x  6 dusko  admin   204B  5 Sep 17:23 nginx
-rw-------  1 root   admin   116B  5 Sep 22:49 php-fpm.log

$ ls -lhrt /usr/local/var/run/
total 16
drwxr-xr-x  7 dusko  admin   238B  5 Sep 10:32 nginx
-rw-r--r--  1 root   admin     6B  5 Sep 17:26 nginx.pid
-rw-r--r--  1 root   admin     5B  5 Sep 22:49 php-fpm.pid

$ sudo cat /usr/local/var/log/php-fpm.log 
[05-Sep-2016 22:49:12] NOTICE: fpm is running, pid 32395
[05-Sep-2016 22:49:12] NOTICE: ready to handle connections

$ cat /usr/local/var/run/php-fpm.pid 
32395

$ ps aux | grep 32395
dusko           32415   1.5  0.0  2454296    816 s004  R+   10:53pm   0:00.01 grep 32395
root            32395   0.0  0.0  2487132    592   ??  Ss   10:49pm   0:00.02 php-fpm -e
```


```
$ grep -n user /private/etc/php-fpm.conf
148:; Unix user/group of processes
149:; Note: The user is mandatory. If the group is not set, the default user's group
151:user = nobody
... [ snip ] ...

$ sudo sed -i .bak '151s/nobody/dusko/' /private/etc/php-fpm.conf
Password:

$ diff --unified=0 /private/etc/php-fpm.conf.bak /private/etc/php-fpm.conf
--- /private/etc/php-fpm.conf.bak	2016-09-05 22:46:35.000000000 -0700
+++ /private/etc/php-fpm.conf	2016-09-05 23:05:57.000000000 -0700
@@ -151 +151 @@
-user = nobody
+user = dusko

$ sudo sed -i .bak '152s/nobody/staff/' /private/etc/php-fpm.conf

$ diff --unified=0 /private/etc/php-fpm.conf.bak /private/etc/php-fpm.conf
--- /private/etc/php-fpm.conf.bak	2016-09-05 23:05:57.000000000 -0700
+++ /private/etc/php-fpm.conf	2016-09-05 23:07:23.000000000 -0700
@@ -152 +152 @@
-group = nobody
+group = staff
```


You can choose either a socket or TCP/IP address. I decided to use a socket.       

```
$ sudo sed -i .bak '166s/listen/#listen/' /private/etc/php-fpm.conf 
Password:

$ diff --unified=0 /private/etc/php-fpm.conf.bak /private/etc/php-fpm.conf 
--- /private/etc/php-fpm.conf.bak	2016-09-05 23:07:23.000000000 -0700
+++ /private/etc/php-fpm.conf	2016-09-06 01:21:18.000000000 -0700
@@ -166 +166 @@
-listen = 127.0.0.1:9000
+#listen = 127.0.0.1:9000

$ sudo sed -i .bak -e $'167i\\\nlisten = /usr/local/var/run/php-fpm.sock\\\n' /private/etc/php-fpm.conf
Password:

$ diff --unified=0 /private/etc/php-fpm.conf.bak /private/etc/php-fpm.conf
--- /private/etc/php-fpm.conf.bak	2016-09-06 01:21:18.000000000 -0700
+++ /private/etc/php-fpm.conf	2016-09-06 01:27:28.000000000 -0700
@@ -166,0 +167 @@
+listen = /usr/local/var/run/php-fpm.sock
```



The nginx recommended method of serving up multiple websites used to be to create sites-available and sites-enabled directories within the nginx install. You would create your conf files in sites-available and then symlink them to sites-enabled if you wanted to enable them. This way you could enable and disable websites based on whether the conf is symlinked in sites-enabled. When installed with Homebrew, nginx is located in /usr/local/etc/nginx. 

```
$ sudo /usr/libexec/locate.updatedb
Password:

$ locate nginx.conf
/usr/local/etc/nginx/nginx.conf
```


With this Homebrew install, that method changed and it is now to place sites in "servers" directory.     

```
$ grep -n include /usr/local/etc/nginx/nginx.conf
21:    include       mime.types;
74:        #    include        fastcgi_params;
120:    include servers/*;
```


```
$ sudo vi /usr/local/etc/nginx/servers/grav.localhost
```


In this file paste:    

```
server {
    #listen 80;
    index index.html index.php;

    ## Begin - Server Info
    root /Users/dusko/grav;
    server_name grav.localhost;
    ## End - Server Info

    ## Begin - Index
    # for subfolders, simply adjust:
    # `location /subfolder {`
    # and the rewrite to use `/subfolder/index.php`
    location / {
        try_files $uri $uri/ /index.html /index.php;
    }
    ## End - Index

    ## Begin - PHP
    location ~ \.php$ {
        # Choose either a socket or TCP/IP address
        fastcgi_pass unix:/usr/local/var/run/php-fpm.sock;
        # fastcgi_pass 127.0.0.1:9000;

        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root/$fastcgi_script_name;
    }
    ## End - PHP

    ## Begin - Security
    # deny all direct access for these folders
    location ~* /(.git|cache|bin|logs|backups|tests)/.*$ { return 403; }
    # deny running scripts inside core system folders
    location ~* /(system|vendor)/.*\.(txt|xml|md|html|yaml|php|pl|py|cgi|twig|sh|bat)$ { return 403; }
    # deny running scripts inside user folder
    location ~* /user/.*\.(txt|md|yaml|php|pl|py|cgi|twig|sh|bat)$ { return 403; }
    # deny access to specific files in the root folder
    location ~ /(LICENSE.txt|composer.lock|composer.json|nginx.conf|web.config|htaccess.txt|\.htaccess) { return 403; }
    ## End - Security

    access_log  logs/grav.localhost.access.log  main;
    error_log   logs/grav.localhost.error.log; 
}
```


Set the hostname in your /etc/hosts file to something more memorable. 

```
$ locate hosts | grep etc
... [ snip ] ...
/private/etc/hosts
/private/etc/hosts.equiv
... [ snip ] ...

$ cat /private/etc/hosts
##
# Host Database
#
# localhost is used to configure the loopback interface
# when the system is booting.  Do not change this entry.
##
127.0.0.1	localhost
255.255.255.255	broadcasthost
::1             localhost 

$ sudo sed -i .bak -e $'8i\\\n127.0.0.1\       grav.localhost\\\n' /private/etc/hosts
Password:

$ diff --unified=0 /private/etc/hosts.bak /private/etc/hosts
--- /private/etc/hosts.bak	2016-08-30 03:03:55.000000000 -0700
+++ /private/etc/hosts	2016-09-05 21:19:55.000000000 -0700
@@ -7,0 +8 @@
+127.0.0.1       grav.localhost
```


Restart php-fpm and nginx.

```
$ ps aux | grep fpm
dusko           39421   0.0  0.0  2444056    788 s006  S+    5:32pm   0:00.00 grep fpm
dusko           39418   0.0  0.0  2487132    636   ??  S     5:32pm   0:00.00 php-fpm
dusko           39417   0.0  0.0  2487132    612   ??  S     5:32pm   0:00.00 php-fpm
root            39416   0.0  0.0  2487132    804   ??  Ss    5:32pm   0:00.00 php-fpm

$ sudo kill -TERM 39416
Password:

$ ps aux | grep fpm
dusko           39467   0.0  0.0  2423376     24 s006  R+    5:34pm   0:00.00 grep fpm

$ sudo php-fpm

$ ps aux | grep fpm
dusko           39475   0.0  0.0  2423376    212 s006  R+    5:34pm   0:00.00 grep fpm
dusko           39472   0.0  0.0  2486108    648   ??  S     5:34pm   0:00.00 php-fpm
dusko           39471   0.0  0.0  2486108    672   ??  S     5:34pm   0:00.00 php-fpm
root            39470   0.0  0.0  2486108    860   ??  Ss    5:34pm   0:00.00 php-fpm

$ sudo brew services restart nginx
```


Test.

```
$ mkdir -p /Users/dusko/grav

$ printf '<?php phpinfo();\n' > /Users/dusko/grav/info.php 

$ cat /Users/dusko/grav/info.php 
<?php phpinfo();
```


```
$ lynx --dump http://grav.localhost/info.php
   [1]PHP logo

PHP Version 5.5.36
... [ snip ] ...
```


### Install Grav ###

Download Grav release package you wish to install. I'm going to use the **Saturn Site**.    

```
$ cd
$ wget https://github.com/getgrav/grav-skeleton-saturn-site/releases/download/1.1.0/grav-skeleton-saturn-site-v1.1.0.zip
$ unzip grav-skeleton-saturn-site-v1.1.0.zip 
$ rm -rf /Users/dusko/grav
$ mv /Users/dusko/grav-skeleton-saturn-site /Users/dusko/grav
```


Test.


```
$ nc grav.localhost 80
```

Type in headers as following:

```
GET / HTTP/1.1
Host: duskopijetlovic.com
```


After you've typed in the headers, press ENTER twice, and the server will send back the requested page.


```
HTTP/1.1 200 OK
Server: nginx/1.10.1
Date: Sat, 10 Sep 2016 01:46:46 GMT
Content-Type: text/html
Transfer-Encoding: chunked
Connection: keep-alive
X-Powered-By: PHP/5.5.36
Set-Cookie: grav-site-900030b=6c740128ccf7bf0a9f51e56b801d7789; expires=Sat, 10-Sep-2016 02:16:45 GMT; Max-Age=1800; path=/
Pragma: no-cache
Set-Cookie: grav-site-900030b=6c740128ccf7bf0a9f51e56b801d7789; expires=Sat, 10-Sep-2016 02:16:45 GMT; Max-Age=1800; path=/; domain=grav.localhost; httponly
Cache-Control: max-age=604800
Expires: Sat, 17 Sep 2016 01:46:46 GMT

16b4
<!DOCTYPE html>
<html lang="en">
<head>
            <meta charset="utf-8" />
        <title>Blog | Dusko Pijetlovic</title>
        <meta name="generator" content="GravCMS" />
<meta name="description" content="The personal website of Dusko Pijetlovic." />
... [ snip ] ...
... [ snip ] ...
... [ snip ] ...
    </body>
</html>

0

^C

```


### A Side Note: Use php-config to Get Information about PHP Configuration ###

To get information about PHP configuration and compile options, use php-config.

```
$ php-config
Usage: /usr/bin/php-config [OPTION]
Options:
  --prefix            [/usr]
  --includes          [-I/usr/include/php -I/usr/include/php/main -I/usr/include/php/TSRM -I/usr/include/php/Zend -I/usr/include/php/ext -I/usr/include/php/ext/date/lib]
  --ldflags           [ -L/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.11.Internal.sdk/usr/lib ]
  --libs              [-lresolv  -lcrypto.35 -lssl.35 -lcrypto.35 -lz -lexslt -ltidy -lresolv -ledit -lncurses -lldap -llber -liconv -liconv -lpng -lz -ljpeg -lcrypto.35 -lssl.35 -lcrypto.35 -lcurl -lbz2 -lz -lpcre -lcrypto.35 -lssl.35 -lcrypto.35 -lm  -lxml2 -lz -licucore -lm -lgssapi_krb5 -lkrb5 -lk5crypto -lcom_err -lcurl -lxml2 -lz -licucore -lm -lxml2 -lz -licucore -lm -lnetsnmp -lcrypto.35 -lxml2 -lz -licucore -lm -lxml2 -lz -licucore -lm -lxml2 -lz -licucore -lm -lxml2 -lz -licucore -lm -lxml2 -lz -licucore -lm -lxml2 -lz -licucore -lm -lxml2 -lxslt ]
  --extension-dir     [/usr/lib/php/extensions/no-debug-non-zts-20121212]
  --include-dir       [/usr/include/php]
  --man-dir           [/usr/share/man]
  --php-binary        [/usr/bin/php]
  --php-sapis         [ apache2handler cli fpm]
  --configure-options [--prefix=/usr --mandir=/usr/share/man --infodir=/usr/share/info --disable-dependency-tracking --sysconfdir=/private/etc --with-libdir=lib --enable-cli --with-iconv=/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.11.Internal.sdk/usr --with-config-file-path=/etc --with-config-file-scan-dir=/Library/Server/Web/Config/php --with-libxml-dir=/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.11.Internal.sdk/usr --with-openssl=/usr/local/libressl/include]
  --version           [5.5.36]
  --vernum            [50536]
```


