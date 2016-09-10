---
layout: post
title:  Create and Maintain Jekyll GitHub Page - Mac OS X
---

### Create and Maintain Jekyll GitHub Page - Mac OS X ###

```
$ sudo gem install jekyll
$ sudo gem install github-pages
$ cd
$ git clone https://github.com/plusjade/jekyll-bootstrap.git duskop.github.com
$ # This git command will create ~/duskop.github.com directory
$ cd ~/duskop.github.com
$ printf "source 'https://rubygems.org'\n" > Gemfile
$ printf "gem 'github-pages'\n" >> Gemfile
$ sudo gem install bundler
$ which bundler
/usr/local/bin/bundler
$ bundler install
```


#### Starting Jekyll ####

```
$ cd ~/duskop.github.com
$ bundle exec jekyll serve
... [ snip ] ...
Server address: http://127.0.0.1:4000/
Server running... press ctrl-c to stop.

$ nc 127.0.0.1 4000
GET /
```


### Add a New SSH Key to Your GitHub Account ###

```
$ git remote set-url origin git@github.com:duskop/duskop.github.com.git
$ git push origin master
Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```


Since there are no existing SSH keys in ```~/.ssh``` directory (By default, the filenames of the public keys are one of the following: id_dsa.pub, id_ecdsa.pub, id_ed25519.pub, id_rsa.pub), you need to add a new SSH key to your GitHub account:

```
$ ls -alh ~/.ssh/
total 8
drwx------   3 dusko  staff   102B 14 Aug 15:44 .
drwxr-xr-x+ 24 dusko  staff   816B 14 Aug 16:55 ..
-rw-r--r--   1 dusko  staff   803B 14 Aug 16:59 known_hosts
```


Generate SSH authentication key.

```
$ ssh-keygen -t rsa -b 4096 -C "youremail@somedomain.com"
```

Replace "youremail@somedomain.com" with your real email address.


When you're prompted to "Enter a file in which to save the key," press Enter. This accepts the default file location.   

```
Enter a file in which to save the key (/Users/dusko/.ssh/id_rsa): [Press enter] 
```


At the prompt, type a secure passphrase.     

```
Enter passphrase (empty for no passphrase): [Type a passphrase]
Enter same passphrase again: [Type passphrase again]
```


```
$ ls -alh ~/.ssh/
total 24
drwx------   5 dusko  staff   170B 14 Aug 17:11 .
drwxr-xr-x+ 24 dusko  staff   816B 14 Aug 16:55 ..
-rw-------   1 dusko  staff   3.2K 14 Aug 17:11 id_rsa
-rw-r--r--   1 dusko  staff   743B 14 Aug 17:11 id_rsa.pub
-rw-r--r--   1 dusko  staff   803B 14 Aug 16:59 known_hosts
```


Add your SSH key (private key file) to the ssh-agent     

Enable ssh-agent - start the ssh-agent in the background: 

```
eval "$(ssh-agent -s)"
Agent pid 35828
```


```
$ ssh-add ~/.ssh/id_rsa
Enter passphrase for /Users/dusko/.ssh/id_rsa: 
Identity added: /Users/dusko/.ssh/id_rsa (/Users/dusko/.ssh/id_rsa)
```


Copy the SSH key (public key file) to your clipboard. When copying your key, don't add any newlines or whitespace.  
   
```
$ pbcopy < ~/.ssh/id_rsa.pub 
```


Straight from [Adding a new SSH key to your GitHub account](https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/): 

* Log in to your GitHub account.     
* In the top right corner of any page, click your profile photo, then click **Settings**.    
* In the user settings sidebar, click **SSH and GPG keys**.   
* Click **New SSH key** or **Add SSH key**.    
* In the "Title" field, add a descriptive label for the new key. For example, if you're using a personal Mac, you might call this key "Personal MacBook Air".    
* Paste your key into the "Key" field.    
* Click **Add SSH key**.   
* If prompted, confirm your GitHub password.     



#### Add the poole Git Repository ####

```
$ cd
$ wget https://github.com/poole/poole/archive/master.zip
$ file master.zip 
master.zip: Zip archive data, at least v1.0 to extract
$ unzip master.zip 
$ rm -i master.zip 
remove master.zip? y
$ mv poole-master duskop.github.com
$ cd ~/duskop.github.com
$ printf "# duskop.github.com" > README.md
$ git init
Initialized empty Git repository in /Users/dusko/duskop.github.com/.git/
$ git add README.md 
$ git commit -m "First commit"
$ git remote add origin git@github.com:duskop/duskop.github.com.git
$ git push -u origin master
$ git add --all
$ git commit -m "Added all files"
$ git config --global user.name "Dusko Pijetlovic"
$ git config --global user.email "youremail@somedomain.com"
$ # Replace "youremail@somedomain.com" with your real email address.
$ git commit --amend --reset-author
$ git push -u origin master
```


From [How I Created a Beautiful and Minimal Blog Using Jekyll, Github Pages, and poole](http://joshualande.com/jekyll-github-pages-poole):   

Create an Archive page which lists all of blog posts. To do this, create the file archive.md which shows a dynamic list of all blog posts:   

{% highlight html %}
{% raw %}
$ cd ~/duskop.github.com

$ vim archive.md
$ cat archive.md
---
layout: page
title: Archive
---

## Blog Posts

{% for post in site.posts %}
  * {{ post.date | date_to_string }} &raquo; [ {{ post.title }} ]({{ post.url }})
{% endfor %}
{% endraw %}
{% endhighlight %}


```
$ git add --all
$ git commit -m "Added archive.md"
$ git push -u origin master
```


Add a navigation bar at the top of the website with links to the About page, Archive page, and the feed. To do this, modify the file ```_config.yml``` to define a dictionary of pages to show in header:

```
$ cp ~/duskop.github.com/_config.yml ~/duskop.github.com/_config.yml.bak
$ vim ~/duskop.github.com/_config.yml
$ diff --unified=0 ~/duskop.github.com/_config.yml.bak ~/duskop.github.com/_config.yml 
--- /Users/dusko/duskop.github.com/_config.yml.bak	2016-08-14 20:20:37.000000000 -0700
+++ /Users/dusko/duskop.github.com/_config.yml	2016-08-14 20:32:04.000000000 -0700
@@ -5,3 +5,3 @@
-title:               Poole
-tagline:             The Jekyll Butler
-url:                 http://getpoole.com
+title:               Dusko Pijetlovic
+tagline:             Dusko Pijetlovic
+url:                 http://duskopijetlovic.com
@@ -20,3 +20,3 @@
-  name:              Mark Otto
-  url:               https://twitter.com/mdo
-  email:             markdotto@gmail.com
+  name:              Dusko Pijetlovic
+  url:               http://duskopijetlovic.com
+  email:             dusko@duskopijetlovic.com
@@ -25 +25 @@
-version:             2.0.0
+version:             1.0.0
@@ -27 +27 @@
-  repo:              https://github.com/poole/poole
+  repo:              https://duskop.github.io
@@ -32,0 +33,10 @@
+
+# Pages list
+# From:
+#   http://joshualande.com/jekyll-github-pages-poole
+#
+# This is the list of pages to incldue in the header of the website.
+pages_list:       
+  About: '/about'
+  Archive: '/archive'
+  Feed: '/atom.xml'
```


```
$ rm -i ~/duskop.github.com/_config.yml.bak
remove /Users/dusko/duskop.github.com/_config.yml.bak? y
```


Modify the file ```_layouts/default.html``` to loop over this list, creating links to each of the pages:

```
$ cp ~/duskop.github.com/_layouts/default.html ~/duskop.github.com/_layouts/default.html.bak
$ vim ~/duskop.github.com/_layouts/default.html
$ diff --unified=0 ~/duskop.github.com/_layouts/default.html.bak ~/duskop.github.com/_layouts/default.html 
--- /Users/dusko/duskop.github.com/_layouts/default.html.bak	2016-08-14 20:25:44.000000000 -0700
+++ /Users/dusko/duskop.github.com/_layouts/default.html	2016-08-14 20:29:02.000000000 -0700
@@ -11,2 +11,5 @@
-          <a href="{{ site.baseurl }}/" title="Home">{{ site.title }}</a>
-          <small>{{ site.tagline }}</small>
+          <a href="/" title="Home">{{ site.title }}</a>
+
+          {% for page in site.pages_list %}
+              &nbsp;&nbsp;&nbsp;<small><a href="{{ page[1]  }}">{{ page[0] }}</a></small>
+          {% endfor %}
```


```
$ rm -i ~/duskop.github.com/_layouts/default.html.bak
remove /Users/dusko/duskop.github.com/_layouts/default.html.bak? y
```


```
$ git add --all
$ git commit -m "Modified _config.yml and _layouts/default.html"
$ git push -u origin master
```


```
$ printf "source 'https://rubygems.org'\n" > Gemfile
$ printf "gem 'github-pages'\n" >> Gemfile
```


#### Add or Update the Content  ####

To add a new post, create a Markdown file in ```_posts``` directory.


```
$ vim ~/duskop.github.com/_posts/2016-08-14-my-post-about-something.md
```


Ensure that you have a proper header:

```
---
layout: post
title:  My Post About Something
---
```


```
$ git add --all
$ git commit -m "Added a new post"
$ git push -u origin master
```

