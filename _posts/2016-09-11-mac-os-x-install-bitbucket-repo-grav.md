---
layout: post
title:  Install Bitbucket Repo in Production - on OS X 10.9 Mavericks
---

This document requires you having SSH access to the web server, and also git installed on that server.      

Example project: [Grav](https://getgrav.org)       

References:      
* [Grav Development with GitHub - Part 1](https://getgrav.org/blog/developing-with-github-part-1)     
* [Grav Development with GitHub - Part 2](https://getgrav.org/blog/developing-with-github-part-2)     


Log into the production server.    

Navigate to your www directory.    

```
$ ls -ld /usr/share/nginx/www/
drwxr-xr-x 3 root root 4096 Sep  5 02:48 /usr/share/nginx/www/
```


```
$ cd /usr/share/nginx/www/
```


Replace "/usr/share/nginx/www" with the path to your www directory.    

Use git to grab the latest version of your repository and copy it down to the production server. I will install the site in a subdirectory of the root of the www directory. This command will create a subdirectory with the same name as the Bitbucket's git repository name. 

```
$ sudo git clone https://duskop@bitbucket.org/duskop/grav.git
Cloning into 'grav'...
Password for 'https://duskop@bitbucket.org': 
```


**Note:** If you wanted to clone directly into the root of your www directory, you would need to add a space and period. In that case, the git command would not create a subdirectory "grav". Instead, it would pull down the content of the Bitbucket git repository directly into the www directory:

```
$ sudo git clone https://duskop@bitbucket.org/duskop/grav.git .
Cloning into 'grav'...
Password for 'https://duskop@bitbucket.org': 
```


Adjust permissions.     

```
$ sudo chown -R pi:www-data /usr/share/nginx/www/grav
$ sudo find /usr/share/nginx/www/grav -type d -exec chmod 0775 {} \;
$ sudo find /usr/share/nginx/www/grav -type f -exec chmod 0664 {} \;
$ sudo find /usr/share/nginx/www/grav/bin -type f -exec chmod 0764 {} \;
$ sudo chown -R www-data /usr/share/nginx/www/grav/cache/
$ sudo chown -R www-data /usr/share/nginx/www/grav/vendor/
```


The nginx recommended method of serving up multiple websites is to create sites-available and sites-enabled directories within the nginx install. You create your conf files in sites-available and then symlink them to sites-enabled if you want to enable them. This way you enable and disable websites based on whether the conf is symlinked in sites-enabled.    

```
$ grep -n include /etc/nginx/nginx.conf
29:	include /etc/nginx/mime.types;
74:	#include /etc/nginx/naxsi_core.rules;
89:	include /etc/nginx/conf.d/*.conf;
90:	include /etc/nginx/sites-enabled/*;
```


My "sites-available" directory's default file, with comments and blank lines removed.     

grep note:    
grep -v '[[:space:]]#' -> Invert match: remove commented lines that are indented (with preceding spaces).   
grep -v ^#             -> Remove non-indented commented out lines, that is, lines with the comment symbol (#) in the first column.    
grep -v "^$"           -> Remove blank (empty) lines.     

```
$ grep -v '[[:space:]]#' /etc/nginx/sites-available/default | grep -v ^# | grep -v "^$"
server {
   listen 80;
   server_name duskopijetlovic.com;
   root /usr/share/nginx/www/grav;
   location / {
       try_files $uri $uri/ /index.php?$query_string;
   }
   location ~* \.php$ {
       fastcgi_split_path_info ^(.+\.php)(/.+)$;
       try_files                  $uri =404;
       fastcgi_index              index.php;
       fastcgi_intercept_errors   on;
       fastcgi_pass               unix:/var/run/php5-fpm.sock;
       fastcgi_param              SCRIPT_FILENAME  $document_root$fastcgi_script_name;
       fastcgi_buffer_size 8k;
       fastcgi_buffers 32 8k;
       fastcgi_busy_buffers_size 8k;
       fastcgi_temp_file_write_size 8k;
       include                    fastcgi_params;
       fastcgi_param              SCRIPT_FILENAME $request_filename;
       fastcgi_param              PATH_INFO $2;
   }
   location ~* \.(js|css|png|jpg|jpeg|gif|swf|xml|txt)$ {
       expires 14d;
   }
   location ~* \.(ico|pdf|flv)$ {
       expires 1y;
   }
}
server {
	root /usr/share/nginx/www;
	index index.html index.htm;
	server_name localhost;
	location / {
		try_files $uri $uri/ /index.html;
	}
	location /doc/ {
		alias /usr/share/doc/;
		autoindex on;
		allow 127.0.0.1;
		allow ::1;
		deny all;
	}
}
```


Restart nginx.

```
$ sudo /etc/init.d/nginx status
[ ok ] nginx is running.

$ sudo /etc/init.d/nginx stop
Stopping nginx: nginx.

$ sudo /etc/init.d/nginx status
[FAIL] nginx is not running ... failed!
 
$ sudo /etc/init.d/nginx start
Starting nginx: nginx.

$ sudo /etc/init.d/nginx status
[ ok ] nginx is running.
```


Test.

```
$ nc duskopijetlovic.com 80
```

Type in headers as following:   

```
GET / HTTP/1.1
Host: duskopijetlovic.com
```

After you've typed in the headers, press ENTER twice, and the server will send back the requested page.    

```
HTTP/1.1 200 OK
Server: nginx/1.2.1
Date: Fri, 09 Sep 2016 20:47:50 GMT
Content-Type: text/html;charset=UTF-8
Transfer-Encoding: chunked
Connection: keep-alive
Vary: Accept-Encoding
X-Powered-By: PHP/5.6.21-1~dotdeb+zts+7.1
Set-Cookie: grav-site-cd65cc1=85ib1uvmo63c0dkajmuoo7nhc2; expires=Fri, 09-Sep-2016 21:17:50 GMT; Max-Age=1800; path=/
Pragma: no-cache
Set-Cookie: grav-site-cd65cc1=85ib1uvmo63c0dkajmuoo7nhc2; expires=Fri, 09-Sep-2016 21:17:50 GMT; Max-Age=1800; path=/; domain=duskopijetlovic.com; httponly
Cache-Control: max-age=604800
Expires: Fri, 16 Sep 2016 20:47:50 GMT

16c1
<!DOCTYPE html>
<html lang="en">
<head>
            <meta charset="utf-8" />
        <title>Blog | Dusko Pijetlovic</title>
        <meta name="generator" content="GravCMS" />
<meta name="description" content="The personal website of Dusko Pijetlovic." />
        <meta name="viewport" content="width=device-width, initial-scale=1" />
        <link rel="icon" type="image/png" href="/user/themes/saturn/img/favicon.png" />

... [ snip ] ...
... [ snip ] ...
... [ snip ] ...

        </footer>

    </body>
</html>



0

^C
```


### To Update - Pull Updates from Bitbucket Git ###


Log into the production server.    

Navigate to your Grav project directory.    

```
$ ls -ld /usr/share/nginx/www/
drwxr-xr-x 4 root root 4096 Sep  9 19:43 /usr/share/nginx/www/

$ ls -ld /usr/share/nginx/www/grav
drwxrwxr-x 13 pi www-data 4096 Sep  9 22:38 /usr/share/nginx/www/grav
```


```
$ cd /usr/share/nginx/www/grav
```


```
$ sudo git add .

$ sudo git pull 
Password for 'https://duskop@bitbucket.org': 
```


#### Fixing Git Pull Errors ####

* Method 1: Use "sudo git commit -a".      

```
$ sudo git pull
Password for 'https://duskop@bitbucket.org': 
remote: Counting objects: 477, done.
remote: Compressing objects: 100% (205/205), done.
remote: Total 477 (delta 228), reused 477 (delta 228)
Receiving objects: 100% (477/477), 237.32 KiB | 339 KiB/s, done.
Resolving deltas: 100% (228/228), completed with 209 local objects.
From https://bitbucket.org/duskop/grav
   1f23bf7..faaf16f  master     -> origin/master
Removing user/plugins/form/classes/form_serializable.php
Removing system/blueprints/media/rename.yaml
Removing system/blueprints/media/move.yaml
Removing system/blueprints/media/meta.yaml
Removing images/5/9/8/c/e/598ce8d086fcf1081a301e033e0640ed3700a653-about.jpeg
Removing images/0/d/4/e/a/0d4ea0897da7b7f10b68e68220b105815917400c-post-img.jpeg
Removing images/0/8/2/d/7/082d724d6b1098966125bccb89670c1eb4f6ccab-post-img.jpeg
Removing cache/twig/ff/ff3369cb22796a54d339918ae5fc978914079293b3dee2c30ef6350a7b647c77.php
Removing cache/twig/fb/fbf857c1b70b81c307792255a93c43d3d8b6d061af258b652aba4bc4799bddd7.php
... [ snip ] ...
... [ snip ] ...
CONFLICT (modify/delete): cache/twig/59/591d8b206fb68c41309da83003d601979d8aee7d87c275997a8896d090bc4894.php deleted in faaf16f52a62706f2afc4f2cc2a120306d5db4d7 and modified in HEAD. Version HEAD of cache/twig/59/591d8b206fb68c41309da83003d601979d8aee7d87c275997a8896d090bc4894.php left in tree.
Removing cache/twig/58/58c2d6a11c74a9b41c547d78eb07e0bcd6e0eb64d10a1133088c9b3d5ed9da25.php
... [ snip ] ...
... [ snip ] ...
Auto-merging cache/compiled/files/80156c0b764f4c523c272b37ec659826.yaml.php
CONFLICT (content): Merge conflict in cache/compiled/files/80156c0b764f4c523c272b37ec659826.yaml.php
... [ snip ] ...
... [ snip ] ...
Auto-merging bin/plugin
Auto-merging bin/gpm
Auto-merging bin/composer.phar
Automatic merge failed; fix conflicts and then commit the result.
```


```
$ sudo bin/grav clear-cache
```


```
$ sudo git pull
M	CHANGELOG.md
M	bin/composer.phar
M	bin/gpm
M	bin/plugin
M	cache/compiled/blueprints/master-grav.localhost.php
... [ snip ] ...
... [ snip ] ...
Pull is not possible because you have unmerged files.
Please, fix them up in the work tree, and then use 'git add/rm <file>'
as appropriate to mark resolution, or use 'git commit -a'.
```


```
$ sudo git commit -a
$ sudo git add .
$ sudo git pull
Password for 'https://duskop@bitbucket.org': 
```


```
$ sudo /etc/init.d/nginx status
[ ok ] nginx is running.

$ sudo /etc/init.d/nginx stop
Stopping nginx: nginx.

$ sudo /etc/init.d/nginx status
[FAIL] nginx is not running ... failed!

$ sudo /etc/init.d/nginx start
Starting nginx: nginx.

$ sudo /etc/init.d/nginx status
[ ok ] nginx is running.
```


* Method 2: Quick and dirty - Delete the project and git pull it again / Reinstall a problematic plugin with Grav CLI GPM

```
$ cat /usr/share/nginx/www/grav/logs/grav.log 
[2016-09-11 18:12:32] grav.CRITICAL: Class 'Grav\Plugin\Email\Email' not found - Trace: #0 /usr/share/nginx/www/grav/vendor/filp/whoops/src/Whoops/Run.php(363): Whoops\Run->handleError(1, 'Class 'Grav\\Plu...', '/usr/share/ngin...', 35) #1 [internal function]: Whoops\Run->handleShutdown() #2 {main} [] []
```


```
$ ls -lh /usr/share/nginx/www/grav/user/plugins/email/
total 68K
-rw-rw-r-- 1 pi www-data 2.7K Sep 11 16:57 blueprints.yaml
-rw-rw-r-- 1 pi www-data 2.7K Sep 11 16:57 CHANGELOG.md
drwxrwxr-x 2 pi www-data 4.0K Sep 11 16:57 classes
drwxrwxr-x 2 pi www-data 4.0K Sep 11 16:57 cli
-rw-rw-r-- 1 pi www-data  171 Sep 11 16:57 composer.json
-rw-rw-r-- 1 pi www-data 2.3K Sep 11 16:57 composer.lock
-rw-rw-r-- 1 pi www-data 8.8K Sep 11 16:57 email.php
-rw-rw-r-- 1 pi www-data  239 Sep  9 19:43 email.yaml
-rw-rw-r-- 1 pi www-data  270 Sep 11 16:57 hebe.json
-rw-rw-r-- 1 pi www-data 3.5K Sep 11 16:57 languages.yaml
-rw-rw-r-- 1 pi www-data 1.1K Sep  9 19:43 LICENSE
-rw-rw-r-- 1 pi www-data 6.2K Sep 11 16:57 README.md
drwxrwxr-x 3 pi www-data 4.0K Sep 11 16:57 templates
drwxrwxr-x 4 pi www-data 4.0K Sep 11 16:57 vendor
```


```
$ sudo rm -rf /usr/share/nginx/www/grav
```


```
$ sudo git clone https://duskop@bitbucket.org/duskop/grav.git
```


```
$ sudo chown -R pi:www-data /usr/share/nginx/www/grav
$ sudo find /usr/share/nginx/www/grav -type d -exec chmod 0775 {} \;
$ sudo find /usr/share/nginx/www/grav -type f -exec chmod 0664 {} \;
$ sudo find /usr/share/nginx/www/grav/bin -type f -exec chmod 0764 {} \;
$ sudo chown -R www-data /usr/share/nginx/www/grav/cache/
$ sudo chown -R www-data /usr/share/nginx/www/grav/vendor/
```


```
$ sudo /etc/init.d/nginx restart
```


```
$ sudo bin/gpm index --installed-only

GPM Releases Configuration: Stable

PLUGINS [ 7 ]
---------------------------------------------------------------
| Count | Name           | Slug         | Version | Installed |
===============================================================
| 1     | Taxonomy List  | taxonomylist | v1.2.7  | installed |
| 2     | Pagination     | pagination   | v1.3.2  | installed |
| 3     | Feed           | feed         | v1.5.0  | installed |
| 4     | Error          | error        | v1.5.1  | installed |
| 5     | Email          | email        | v2.5.0  | installed |
| 6     | Problems       | problems     | v1.4.4  | installed |
| 7     | Form           | form         | v2.0.2  | installed |
---------------------------------------------------------------

THEMES [ 1 ]
--------------------------------------------------
| Count | Name    | Slug   | Version | Installed |
==================================================
| 1     | Saturn  | saturn | v1.5.0  | installed |
--------------------------------------------------

You can either get more informations about a package by typing:
    bin/gpm info <package>

Or you can install a package by typing:
    bin/gpm install <package>
```


```
$ sudo bin/gpm uninstall email

Preparing to uninstall Email [v2.5.0]
  |- Checking destination...  ok
  `- Dependencies found...

Finishing up uninstalling Email
  |- Uninstalling Email package...  ok                             
  '- Success!  

Clearing cache

Cleared:  /usr/share/nginx/www/grav/cache/twig/*
Cleared:  /usr/share/nginx/www/grav/cache/compiled/*

Touched: /usr/share/nginx/www/grav/user/config/system.yaml
```


```
$ sudo bin/gpm install email

GPM Releases Configuration: Stable



Preparing to install Email [v2.5.0]
  |- Downloading package...   100%
  |- Checking destination...  ok
  |- Installing package...    ok                             
  '- Success!  


Clearing cache

Cleared:  /usr/share/nginx/www/grav/cache/compiled/*

Touched: /usr/share/nginx/www/grav/user/config/system.yaml
```


```
$ sudo chown -R pi:www-data /usr/share/nginx/www/grav
$ sudo find /usr/share/nginx/www/grav -type d -exec chmod 0775 {} \;
$ sudo find /usr/share/nginx/www/grav -type f -exec chmod 0664 {} \;
$ sudo find /usr/share/nginx/www/grav/bin -type f -exec chmod 0764 {} \;
$ sudo chown -R www-data /usr/share/nginx/www/grav/cache/
$ sudo chown -R www-data /usr/share/nginx/www/grav/vendor/
```


```
$ sudo /etc/init.d/nginx restart
```


