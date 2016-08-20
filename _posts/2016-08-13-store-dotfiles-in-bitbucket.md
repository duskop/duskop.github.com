---
layout: post
title:  Store Your Dotfiles in a Bare Git Repository in Bitbucket
---

### 

OS: Mac OS X 10.9.5

```
$ printf "$HOME\n"
/Users/dusko

$ printf "$SHELL\n"
/bin/zsh

$ git init --bare $HOME/.cfg
Initialized empty Git repository in /Users/dusko/.cfg/

$ ls -ld /Users/dusko/.cfg
drwxr-xr-x  9 dusko  staff  306 22 May 15:26 /Users/dusko/.cfg

$ which git
/usr/local/bin/git

$ whereis git
/usr/bin/git

$ git --version
git version 2.6.1

$ /usr/local/bin/git --version
git version 2.6.1

$ /usr/bin/git --version
git version 1.9.5 (Apple Git-50.3)

$ ls -lh .zshrc*
lrwxr-xr-x  1 dusko  staff    35B 12 Oct  2015 .zshrc -> /Users/dusko/.zprezto/runcoms/zshrc
... ... ...
```


Login into your Bitbucket account.   
In Bitbucket web interface:  Repositories > Create repository   
Repository name: cfg,  Access level: This is a private repository,  Repository type: Git   

```
$ cp /Users/dusko/.zprezto/runcoms/zshrc /Users/dusko/.zprezto/runcoms/zshrc.2016-05-22_1530.bak

$ printf "\n" >> /Users/dusko/.zprezto/runcoms/zshrc
$ printf "alias cfg='/usr/local/bin/git --git-dir=$HOME/.cfg/ --work-tree=$HOME'\n" >> /Users/dusko/.zprezto/runcoms/zshrc

$ whereis diff
/usr/bin/diff

$ /usr/bin/diff --unified=0 /Users/dusko/.zprezto/runcoms/zshrc.2016-05-22_1530.bak /Users/dusko/.zprezto/runcoms/zshrc
--- /Users/dusko/.zprezto/runcoms/zshrc.2016-05-22_1530.bak     2016-05-22 15:36:26.000000000 -0700
+++ /Users/dusko/.zprezto/runcoms/zshrc 2016-05-22 19:36:32.000000000 -0700
@@ -125,0 +126,2 @@
+
+alias cfg='/usr/local/bin/git --git-dir=/Users/dusko/.cfg/ --work-tree=/Users/dusko'

$ pwd
/Users/dusko

$ cfg status
On branch master

Initial commit

nothing to commit (create/copy files and use "git add" to track)

$ which cfg
cfg: aliased to /usr/local/bin/git --git-dir=/Users/dusko/.cfg/ --work-tree=/Users/dusko

$ printf ".cfg\n" > .gitignore

$ git clone --bare https://bitbucket.org/duskop/cfg.git $HOME/.cfg
fatal: destination path '/Users/dusko/.cfg' already exists and is not an empty directory.

$ rm -rf ~/.cfg

$ git clone --separate-git-dir=$HOME/.cfg https://bitbucket.org/duskop/cfg.git cfg-tmp
Cloning into 'cfg-tmp'...
Username for 'https://bitbucket.org': duskop
Password for 'https://duskop@bitbucket.org': 
warning: You appear to have cloned an empty repository.
Checking connectivity... done.

$ ls -ld ~/.cfg
drwxr-xr-x  9 dusko  staff  306 22 May 17:26 /Users/dusko/.cfg

$ cfg status
On branch master

Initial commit

Untracked files:
  (use "git add <file>..." to include in what will be committed)

    ... ... ... 
    ... ... ... 
    .gitignore
    ... ... ... 
    .mutt
    ... ... ... 
    .vimrc
    ... ... ... 


nothing added to commit but untracked files present (use "git add" to track)
```


```
$ cfg add .vimrc
$ cfg commit -m "Add vimrc"
[master (root-commit) 7afb825] Add vimrc
 Committer: Dusko Pijetlovic <dusko@Duskos-MacBook-Pro.local>
Your name and email address were configured automatically based
on your username and hostname. Please check that they are accurate.
You can suppress this message by setting them explicitly. Run the
following command and follow the instructions in your editor to edit
your configuration file:

    git config --global --edit

After doing this, you may fix the identity used for this commit with:

    git commit --amend --reset-author

 1 file changed, 337 insertions(+)
 create mode 100644 .vimrc
```


```
$ cfg status
On branch master
Your branch is based on 'origin/master', but the upstream is gone.
  (use "git branch --unset-upstream" to fixup)
Untracked files:
  (use "git add <file>..." to include in what will be committed)

    ... ... ...
    ... ... ...

nothing added to commit but untracked files present (use "git add" to track)

```


```
$ cfg push
warning: push.default is unset; its implicit value has changed in
Git 2.0 from 'matching' to 'simple'. To squelch this message
and maintain the traditional behavior, use:

  git config --global push.default matching

To squelch this message and adopt the new behavior now, use:

  git config --global push.default simple

When push.default is set to 'matching', git will push local branches
to the remote branches that already exist with the same name.

Since Git 2.0, Git defaults to the more conservative 'simple'
behavior, which only pushes the current branch to the corresponding
remote branch that 'git pull' uses to update the current branch.

See 'git help config' and search for 'push.default' for further information.
(the 'simple' mode was introduced in Git 1.7.11. Use the similar mode
'current' instead of 'simple' if you sometimes use older versions of Git)

Username for 'https://bitbucket.org': duskop
Password for 'https://duskop@bitbucket.org': 
Counting objects: 3, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 6.57 KiB | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To https://bitbucket.org/duskop/cfg.git
 * [new branch]      master -> master
```


Log in your Bitbucket account and check whether sync worked with your remote repository.


#### References ####

[Ask HN: What do you use to manage dotfiles?](https://news.ycombinator.com/item&#63;id=11070797)   
[The best way to store dotfiles: A bare Git repository](https://developer.atlassian.com/blog/2016/02/best-way-to-store-dotfiles-git-bare-repo/)

