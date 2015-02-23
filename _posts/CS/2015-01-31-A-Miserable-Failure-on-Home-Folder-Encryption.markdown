---
layout: post
title: Simulate the Magnetic Field of a Solenoid
tags: Linux 
---

# A Miserable Failure on Home Folder Encryption #

A couple days ago, I decided to re-install and upgrade Ubuntu, as I really want to reach the latest versions of many developer tools and softwares etc., without having to compile their source codes from scrath. (And being my first attempt on Unix, I have pretty messed up many things during this trial period.) So I wanna start over.

What bothers me most was the size of the root (`/`) partition. The guide I followed on the topic of partitioning says 10--20GB would be enough.

Now I suspect he is no user of texlive or _Mathematica_. The two beasts add up to 10GB+ after full installation. Of course it can be reduced, for example by not installing entire texlive (`texlive-full` or the so-called "vanilla" texlive). In that case, potentially needed packages have to be installed on-the-fly, which you would not want to happen. 

So anyway, I re-installed Ubuntu. But I made a mistake that I luckily didn't make during my first install: _I chosed to encrypt home folder._ 

So what? So in hindsight, I, being Chinese, is doomed in some fashion. 

Linux has a maximum file length of 255 characters for most filesystems (including EXT4, on which I installed my system). After encryption, actual usable length drops to 143 characters. See [this question][SE] for details. And it becomes severe when multi-byte encoded characters are used, like CJK characters. For which case only up to 45 characters are allowed.

[SE]: http://unix.stackexchange.com/questions/32795/what-is-the-maximum-allowed-filename-and-folder-size-with-ecryptfs

No, I'm not a maniac for naming files that long. (Some) torrent files are. Though I can rename torrent files or files to be downloaded, I cannot touch intermediate files generated during the downloading process.

I think any one of the followings will solve the problem:
1. Unencrypt `home/usrname` directory.
2. Change environment variable of Transmission (my torrent client) so that hopefully it will store temporary files outside of `home/usrname` directory.

Both are complicated. Neither Do I like.

I then asked professionals on [Ask Ubuntu][ask] for help. Eventually, it turns out I can simply move the desired directory (`~/.config/transmission`) out of my home directory,

    cp -rp ~/.config/transmission /home/TooLong/transmission

and link it back to the original place.

    ln -s /home/TooLong/transmission/ ~/.config/transmission

That looks like the best I can do. And it finally worked.

[ask]: http://askubuntu.com/
