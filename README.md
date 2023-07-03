# Floppinux Builder

Using a known environment host, provided by Vagrant, create a Floppinux image

## Use

First startup
```sh
mkdir -p ../data_debian11_x64
vagrant up
ls -l ../data_debian11_x64/*.img
```

Interactive Usage
```sh
vagrant up
vagrant ssh
```

Rerun Scripts
```sh
vagrant provision
```

Close VM
```sh
vagrant halt -f
```

Clean Rebuild
```sh
vagrant destroy -f
vagrant up
```

## Requirements

* Vagrant
* Virtualbox (or other backend)

## References

Attempting to follow the Floppinux 0.2.1 build instructions for x64, and referencing x86 steps as required.

* Floppinux Github https://github.com/w84death/floppinux
* x86 instructions https://web.archive.org/web/20220520224748/https://bits.p1x.in/floppinux-an-embedded-linux-on-a-single-floppy/
* x64 supliment    https://web.archive.org/web/20220520224018/https://bits.p1x.in/how-to-build-32-bit-floppinux-on-a-64-bit-os/
* Linux Kernel Source        https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git
* Github Linux Kernel Mirror https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git
* Floppinux Kernel Config    https://web.archive.org/web/20220520224748/https://krzysztofjankowski.com/floppinux/downloads/0.2.1/linux/.config
* BusyBox Source             https://busybox.net/downloads/busybox-1.33.1.tar.bz2
* BusyBox Configuration      https://busybox.net/FAQ.html#configure
* MUSL CROSS CC              https://musl.cc/i486-linux-musl-cross.tgz
* MUSL LIBC Manual           https://musl.libc.org/doc/1.1.24/manual.html
