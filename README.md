# Backport a package to Debian stable taking libimobiledevice as example

It is April 2018 and someone gave me a new phone running the latest iOS. But unfortunately, my Debian can't talk to it. I have to send myself emails to retrieve pictures from the camera. Really?

The great [libimobiledevice](https://github.com/libimobiledevice/libimobiledevice) received some patches just a few weeks ago that should fix it, but they won't be released for my stable Debian system. I had a look at Debian Backports, but couldn't find the package in there. Installing libimobiledevice from source could also be risky. It will overwrite existing files and who knows if it uninstalls correctly. 
So it's time to find out how to build a Debian package from the latest source.

I started with the excellent [Debian Building Tutorial](https://wiki.debian.org/BuildingTutorial).
There you learn how to download the source of your currently installed package and build it.
If any of the following instructions go wrong, go back to the building tutorial and learn how to build stuff in general.

For a Debian build, you need two things. First the source code. And second the Debian build scripts. You get the current source and build scripts with `apt source libimobiledevice`.
But I would like a newer version. I tried to build the latest source with the Debian build scripts in there, but they were too old.
Next I looked at the Debian repository at git://git.debian.org/git/pkg-gtkpod/packages/libimobiledevice.git, but the latest commit is from June 2016.
I then looked at the Ubuntu package and found the source and build scripts there. They work. :-)
Later I also discovered that Debian does have the latest source on there website, just not in the Git repository.
But here I will document what I did with the Ubuntu packages downloaded from https://launchpad.net/ubuntu/bionic/+source/libimobiledevice.

```sh
# download
wget https://launchpad.net/ubuntu/+archive/primary/+files/libimobiledevice_1.2.1~git20171128.5a854327+dfsg.orig.tar.bz2
wget https://launchpad.net/ubuntu/+archive/primary/+files/libimobiledevice_1.2.1~git20171128.5a854327+dfsg-0.1.debian.tar.xz

# verify that you got the right files
echo '0bb652c0eaa3be1adea5f37fc08a129d06e5a7fc75f21fd5e17a80e5fb485d0d libimobiledevice_1.2.1~git20171128.5a854327+dfsg.orig.tar.bz2' | sha256sum -c
echo '415ae0bebb500f66737cf4356ce3068a8a730c8ca201f8c040c8265f010c3d6c libimobiledevice_1.2.1~git20171128.5a854327+dfsg-0.1.debian.tar.xz' | sha256sum -c

# unpack
tar xf libimobiledevice_1.2.1~git20171128.5a854327+dfsg.orig.tar.bz2
cd libimobiledevice-1.2.1
tar xf ../libimobiledevice_1.2.1~git20171128.5a854327+dfsg-0.1.debian.tar.xz

# install requirements
sudo apt install build-essential fakeroot devscripts
sudo apt build-dep libimobiledevice6

# build the debian packages
debuild -b -uc -us

# install the packages
sudo dpkg -i ../*.deb
```

The debuild manual doesn't explain the used parameters very well. They are passed on to dpkg-buildpackage and its manpage says:

```
-b     Equivalent to --build=binary or --build=any,all.
-uc, --unsigned-changes
       Do not sign the .buildinfo and .changes files (long option since dpkg 1.18.8).
-us, --unsigned-source
       Do not sign the source package (long option since dpkg 1.18.8).
```
