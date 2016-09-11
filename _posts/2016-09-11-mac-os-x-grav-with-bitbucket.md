---
layout: post
title:  Configure Grav with Bitbucket on OS X 10.9 Mavericks
---

This document assumes that you already installed Git and Grav on your system.     

References:      
* [Grav Development with GitHub - Part 1](https://getgrav.org/blog/developing-with-github-part-1)     
* [Grav Development with GitHub - Part 2](https://getgrav.org/blog/developing-with-github-part-2)     


Change the current directory to where you installed Grav.    


In Bitbucket web interface:  Repositories > Create repository   


Select:     
* Repository name: grav    
* Access level: [X] This is a private repository     
* Repository type: (X) Git     
* Advanced settings > Project management: [X] Issue tracking      

Click on "Create repository" button.    


From the new repository's page:   
"Your repository is empty — let's put some bits in your bucket."      

Under "Command line" heading, "I'm starting from scratch" link (https://bitbucket.org/duskop/grav#command-line-scratch):     

* Set up your local directory    
[Set up Git](https://www.atlassian.com/git/tutorials/install-git/) on your machine if you haven't already.    

```
$ mkdir /path/to/your/project
$ cd /path/to/your/project
$ git init
$ git remote add origin https://duskop@bitbucket.org/duskop/grav.git
```

This documents assumes that you already installed Grav so skip the first step.     

The git init command creates a new Git repository. It can be used to convert an existing, unversioned project to a Git repository or initialize a new empty repository.   

```
$ cd /Users/dusko/grav
$ git init
$ git remote add origin https://duskop@bitbucket.org/duskop/grav.git
```

* Add all files from the Grav directory, commit and push.      

```
$ git add .
$ git commit -m 'Initial commit'
$ git push -u origin master
Password for 'https://duskop@bitbucket.org':
```


Create a .gitignore file.     


```
$ printf ".sass-cache\n" >> .gitignore
$ printf "composer.lock\n" >> .gitignore
$ printf "cache/*\n" >> .gitignore 
$ printf "assets/*\n" >> .gitignore
$ printf "logs/*\n" >> .gitignore 
$ printf "images/*\n" >> .gitignore
$ printf "user/data/*\n" >> .gitignore 
```


```
$ cat /Users/dusko/grav/.gitignore 
.sass-cache
composer.lock
cache/*
assets/*
logs/*
images/*
user/data/*
```


### Updating - Pushing Updates to Git Bitbucket ###

After making the changes, change the directory where your Grav site is located.        

```
$ cd /Users/dusko/grav
```


```
$ git add --all
$ git commit -m "Added a new post"
$ git push -u origin master
```


