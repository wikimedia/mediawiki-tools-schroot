## MediaWiki-schroot

Running MediaWiki in a full system chroot is a lightweight alternative to
virtualization. The scripts here will set up such a chroot for you, suitable
for installing MediaWiki. A chroot management package called schroot is
used. The host must be a Debian variant of some kind, for example Ubuntu.

The goals of this project are to be:

* Easy to understand, easy to modify. Requires only basic UNIX skills and
  an ability to use the constituent services (HHVM etc.). No puppet.
* Bare metal performance (but can be run on top of a VM if desired).
* Secure
  - One easily-reviewable setup script to run as root.
  - Trust only HHVM and distro packages.
  - No need to trust MediaWiki code.
  - All services run inside the chroot.
* Similar to production: HHVM/Apache/MariaDB.

## Installation

Clone MediaWiki, if you don't have it already:

```
mkdir ~/src/mediawiki
cd ~/src/mediawiki
git clone https://gerrit.wikimedia.org/r/mediawiki/core
git clone https://gerrit.wikimedia.org/r/mediawiki/vendor core/vendor
git clone --recursive https://gerrit.wikimedia.org/r/mediawiki/skins
git clone https://gerrit.wikimedia.org/r/mediawiki/extensions
```

You can use --recursive for extensions, but it will take a long time. Instead
you can get it without --recursive and then get the extensions you need with

```
cd extensions
git submodule update --init <extension>
```

Copy the "config.sample" file to "config". Review and edit it if necessary.
Then review ./setup and run it as root:

```
./setup
```

This will install schroot and set up the guest OS. Then start the schroot
session and services:

```
/etc/init.d/mw-chroot start
```

Then install MediaWiki by navigating to the URL supplied by ./setup. You
can start a shell inside the chroot using:

```
schroot -c mw-session
```

## Security

Note that a chroot is, by its nature, less isolated from the host than a
virtual machine. The network system is shared, so the chroot can connect to
any TCP services running on the host that listen on localhost. And a root
user inside the chroot is able to "break out" of the chroot or directly
perform privileged actions.

In exchange for reduced isolation, we do get some convenience benefits. Bind
mounts allow the webserver to run from the exact same source tree that you
edit from the host, there is no need to copy. Changes to the source are
instantly live. A shared network stack is simple and requires no
configuration. A chroot session starts in under a second, and there is very
little memory overhead.
