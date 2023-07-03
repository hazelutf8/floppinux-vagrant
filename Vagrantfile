# -*- mode: ruby -*-
# vi: set ft=ruby :

# Following Floppinux 0.2.1 build instructions for x64, and referencing x86 steps
# x86 instructions https://web.archive.org/web/20220520224748/https://bits.p1x.in/floppinux-an-embedded-linux-on-a-single-floppy/
# x64 supliment    https://web.archive.org/web/20220520224018/https://bits.p1x.in/how-to-build-32-bit-floppinux-on-a-64-bit-os/

# Tested with Vagrant using Virtualbox as VM backend

# Usage:
# #vagrant halt       # Stop box operation
# #vagrant destroy -f # Delete box contents
# vagrant up          # Bring up new/existing box
# #vagrant provision  # Rerun provisioning scripts in Vagrantfile
# vagrant ssh         # Open interactive console session
#
# vagrant halt -f ; vagrant up ; vagrant ssh
# SECONDS=0 ; vagrant halt -f ; vagrant destroy -f ; vagrant up ; echo "Elapsed ${SECONDS}" ; vagrant ssh

Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "generic/debian11"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  #config.vm.synced_folder ".", "/vagrant"
  config.vm.synced_folder "../data_debian11_x64", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of cores on the VM:
  #   vb.cpus = 2
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end

  # View the documentation for the provider you are using for more
  # information on available options.

  # Root shell script provisioning
  $root_script = <<-ROOT_SHELL
    echo "=== SCRIPT 1"
    echo "Root Hello World Script"
  ROOT_SHELL
  config.vm.provision "shell", inline: $root_script, privileged: true

  # User shell script provisioning
  $user_script = <<-USER_SHELL
    echo "=== SCRIPT 2"
    echo "User Hello World Script"
    echo "Threads $(cat /proc/cpuinfo | grep -i 'cpu cores' | wc -l)"
    echo "RAM $(cat /proc/meminfo | grep MemTotal)"
    sleep 5
  USER_SHELL
  config.vm.provision "shell", inline: $user_script, privileged: false

  $user_workdir = <<-USER_SHELL
    echo "=== SCRIPT 3"
    cd ~
    export VARS=/home/vagrant/vars.sh
    echo "export VARS=${VARS}"                        > "${VARS}"
    echo 'echo Sourcing ENV ${VARS}'                 >> "${VARS}"
    echo 'export HOME="$(dirname ${VARS})"'          >> "${VARS}"
    echo 'export BASE="${HOME}/my-linux-distro"'     >> "${VARS}"
    echo 'export CORES=$(($(nproc) + 1))'            >> "${VARS}"
    echo 'export ARCH=x86'                           >> "${VARS}"
    echo "$(pwd) ${VARS}"
    source "${VARS}"

    mkdir -p "${BASE}"
  USER_SHELL
  config.vm.provision "shell", inline: $user_workdir, privileged: false

  $root_requirements = <<-ROOT_SHELL
    echo "=== SCRIPT 4"
    source /home/vagrant/vars.sh
    
    echo "Update References"
    apt update

    echo "Install specified requrements"
    apt install -y git             | tail -n 2
    apt install -y flex            | tail -n 2
    apt install -y bison           | tail -n 2
    apt install -y libncurses-dev  | tail -n 2
    apt install -y qemu-system-x86 | tail -n 2
    apt install -y syslinux        | tail -n 2

    echo "Implicit Kernel requirements"
    apt install -y libelf-dev      | tail -n 2
    apt install -y libssl-dev      | tail -n 2
    apt install -y bc              | tail -n 2

    echo "Implicit floppy creation requirements"
    apt install -y dosfstools      | tail -n 2 # mkdosfs
    # Add /sbin/mkdosfs to PATH
    echo 'export PATH="${PATH}:/sbin"' >> "${VARS}"
  ROOT_SHELL
  config.vm.provision "shell", inline: $root_requirements, privileged: true

  $user_musl_fetch = <<-USER_SHELL
    echo "=== SCRIPT 5"
    source /home/vagrant/vars.sh
    
    echo 'export MUSL_TARGET="i486-linux-musl"'                              >> "${VARS}"
    echo 'export MUSL_CROSS="-cross"'                                        >> "${VARS}"
    echo 'export MUSL_URL="https://musl.cc/${MUSL_TARGET}${MUSL_CROSS}.tgz"' >> "${VARS}"
    echo 'export MUSL_SHA256="https://musl.cc/SHA512SUMS"'                   >> "${VARS}"
    echo 'export MUSL_DIR="${BASE}/${MUSL_TARGET}${MUSL_CROSS}"'             >> "${VARS}"
    echo 'export MUSL_TGZ="${MUSL_DIR}.tgz"'                                 >> "${VARS}"
    source /home/vagrant/vars.sh
    
    echo "Fetch MUSL"
    cd "${BASE}"
    if [ ! -f ${MUSL_TGZ} ] ; then
      wget --no-verbose "${MUSL_SHA256}"
      wget --no-verbose "${MUSL_URL}" -O "${MUSL_TGZ}"
      if sha512sum SHA512SUMS ; then
        echo "MUSL SHA512 Success"
      else
        echo "MUSL SHA512 Error ..."
        exit 1
      fi
      if [ ! -d "${MUSL_DIR}" ] ; then
        tar -xvf "${MUSL_TGZ}" | tail -n 10
      fi
    fi
    echo "Reporting MUSL downloaded items"
    echo 'export MUSL_PATH="${BASE}/${MUSL_TARGET}${MUSL_CROSS}/bin"'                   >> "${VARS}"
    echo 'export MUSL_CROSS_COMPILE="${MUSL_TARGET}-"'                                  >> "${VARS}"
    echo 'export MUSL_SYSROOT="${BASE}/${MUSL_TARGET}${MUSL_CROSS}"'                    >> "${VARS}"
    # Other CFLAGS: -march=i386 -msoft-float -static
    # See also $(${CROSS_COMPILE}gcc --target-help)
    # See also $(${CROSS_COMPILE}strip --help)
    echo 'export MUSL_EXTRA_CFLAGS=\"-march=i386 -static -I${BASE}/${MUSL_TARGET}${MUSL_CROSS}/include\"' >> "${VARS}"
    echo 'export MUSL_EXTRA_LDFLAGS=\"-L${BASE}/${MUSL_TARGET}${MUSL_CROSS}/lib\"'    >> "${VARS}"

    echo "Updating global CC info to default to MUSL gcc cross (optional)"
    echo '#export ARCH=${ARCH}'                                                   >> "${VARS}"
    echo '#export CC=gcc'                                                         >> "${VARS}"
    echo '#export CXX=g++'                                                        >> "${VARS}"
    echo 'export PATH="${MUSL_PATH}:${PATH}"'                                     >> "${VARS}"
    echo 'export CROSS_COMPILE=${MUSL_CROSS_COMPILE}'                             >> "${VARS}"
    echo '#export EXTRA_CFLAGS=${MUSL_EXTRA_CFLAGS}'                               >> "${VARS}"
    echo '#export EXTRA_LDFLAGS=${MUSL_EXTRA_LDFLAGS}'                             >> "${VARS}"
    source /home/vagrant/vars.sh
  USER_SHELL
  config.vm.provision "shell", inline: $user_musl_fetch, privileged: false
  
  $user_linux_fetch = <<-USER_SHELL
    echo "=== SCRIPT 6"
    source /home/vagrant/vars.sh
    echo 'export KERNEL_TAG="v5.13-rc2"' >> "${VARS}"
    source /home/vagrant/vars.sh
    
    echo "Following version from published instructions"
    echo "Archive link: https://web.archive.org/web/20220520224748/https://bits.p1x.in/floppinux-an-embedded-linux-on-a-single-floppy/"
    echo "Linux kernel ${KERNEL_TAG}"
    # Note, 32-bit instructions from 2021-May-22 specify v5.13.0-rc2+
    # and   64-bit instructions from 2021-Jun-21 specify v5.13.0.rc3+
    # https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git # Github mirror is usually faster
    cd ${BASE}
    [ ! -d ./linux ] && \
      git clone \
        --depth=1 \
        https://github.com/torvalds/linux.git \
        -b ${KERNEL_TAG}
    echo Done Clone
    du -h -d 1 ${BASE}/linux | cut -f 1
  USER_SHELL
  config.vm.provision "shell", inline: $user_linux_fetch, privileged: false

  $user_linux_config = <<-USER_SHELL
    echo "=== SCRIPT 7"
    source /home/vagrant/vars.sh
    echo 'export KERNEL_CONFIG_URL="https://web.archive.org/web/20220520224748/https://krzysztofjankowski.com/floppinux/downloads/0.2.1/linux/.config"' >> "${VARS}"
    echo 'export KERNEL_CONFIG_FILE="${BASE}/floppinux_0.2.1_kernel.config"' >> "${VARS}"
    source /home/vagrant/vars.sh

    cd ${BASE}/linux
    du -h -d 1 . | tail -n 1

    echo "Get Floppinux Kernel Config"
    [ ! -f "${KERNEL_CONFIG_FILE}" ] && wget "${KERNEL_CONFIG_URL}" -O "${KERNEL_CONFIG_FILE}"

    echo "Configure Linux Kernel"
    cd ${BASE}/linux
    #git reset --hard ; git clean -xdf  # Force clean kernel reconfigure and rebuild
    if [ ! -f .config ] ; then
      echo "Set Kernel config"
      cat "${KERNEL_CONFIG_FILE}" > "${BASE}/linux/.config"
      # Note, 80386 might be possible when cmpxchg instruction emulation enabled by CONFIG_X86_CMPXCHG64 (enabled by PAE support)
      # See https://musl.libc.org/doc/1.1.24/manual.html
      # See https://lore.kernel.org/all/CAJF2gTQF-e+OUw9VLFOUvFbyroMnxsYyYxCJepQVWnvOTsx1HQ@mail.gmail.com/T/

      # Optional force cc to -march=386
      sed -i "s|march=i486|march=i386|g" "${BASE}/linux/arch/x86/Makefile_32.cpu"
    fi
  USER_SHELL
  config.vm.provision "shell", inline: $user_linux_config, privileged: false

  $user_linux_build = <<-USER_SHELL
    echo "=== SCRIPT 8"
    source /home/vagrant/vars.sh

    echo "Build Floppinux Kernel"
    cd ${BASE}/linux
    #make ARCH=${ARCH} bzImage -j ${CORES}
    CMD="make ARCH=${ARCH} CROSS_COMPILE=${CROSS_COMPILE} bzImage -j ${CORES}"
    echo ${CMD}
    sleep 5
    ${CMD}
    ls -lh $(find . -name bzImage)
    cp -f $(find . -name bzImage | grep ${ARCH} | tail -n 1) ${BASE}/
    file ${BASE}/bzImage
  USER_SHELL
  config.vm.provision "shell", inline: $user_linux_build, privileged: false

  $user_busybox_fetch = <<-USER_SHELL
    echo "=== SCRIPT 9"
    source /home/vagrant/vars.sh
    echo 'export BUSYBOX_VER="1.33.1"' >> "${VARS}"
    echo 'export BUSYBOX_URL="https://busybox.net/downloads/busybox-${BUSYBOX_VER}.tar.bz2"' >> "${VARS}"
    echo 'export BUSYBOX_DIR="${BASE}/busybox-${BUSYBOX_VER}"' >> "${VARS}"
    echo 'export BUSYBOX_TBZ="${BUSYBOX_DIR}.tar.bz2"' >> "${VARS}"
    echo 'export BUSYBOX_CONFIG_URL="https://web.archive.org/web/20220520224748/https://krzysztofjankowski.com/floppinux/downloads/0.2.1/linux/.config"' >> "${VARS}"
    echo 'export BUSYBOX_CONFIG_FILE="${BASE}/floppinux_0.2.1_busybox.config"' >> "${VARS}"
    source /home/vagrant/vars.sh
    
    echo "Fetch BusyBox source"
    cd "${BASE}"
    if [ ! -f "${BUSYBOX_TBZ}" ] ; then
      wget --no-verbose "${BUSYBOX_URL}.sha256" -O "${BUSYBOX_TBZ}.sha256"
      wget --no-verbose "${BUSYBOX_URL}" -O "${BUSYBOX_TBZ}"
      if sha256sum "${BUSYBOX_TBZ}.sha256" ; then
        echo "Busybox SHA256 Success"
      else
        echo "Busybox SHA256 Error ..."
        exit 1
      fi
    fi
    [   -d "${BUSYBOX_DIR}" ] && rm -rf "${BUSYBOX_DIR}" # Force clean BusyBox reconfig and rebuild
    [ ! -d "${BUSYBOX_DIR}" ] && tar -xjvf "${BUSYBOX_TBZ}" | tail -n 10

    #echo "Get Floppinux Busybox Config (unused)"
    #sleep 5
    #cd ${BASE}
    #[ ! -f "${BUSYBOX_CONFIG_FILE}" ] && wget "${BUSYBOX_CONFIG_URL}" -O "${BUSYBOX_CONFIG_FILE}"
  USER_SHELL
  config.vm.provision "shell", inline: $user_busybox_fetch, privileged: false

  $user_busybox_config = <<-USER_SHELL
    echo "=== SCRIPT 10"
    source /home/vagrant/vars.sh

    cd "${BUSYBOX_DIR}"
    if [ ! -f .config ] ; then
      echo "Init BusyBox config"
      # See: https://busybox.net/FAQ.html#configure
      #make ARCH=${ARCH} allnoconfig | tail -n 20  # Minimal config
      make ARCH=${ARCH} defconfig   | tail -n 20  # Typical config

      echo "Configure BusyBox Cross Compiler Options"
      # Always enabled items, LFS prevents uoff_t typedef error, others set MUSL+GCC cross compiler toolchain
      # Need to encompass the escaped quotes, so the .config has double quote literals around the toolchain values
      SET_A='\"${MUSL_PATH}/${MUSL_CROSS_COMPILE}\"'
      SET_B='\"${MUSL_SYSROOT}\"'
      SET_C='\"${MUSL_EXTRA_CFLAGS}\"'
      SET_D='\"${MUSL_EXTRA_LDFLAGS}\"'
      sed -i "s|.*CONFIG_LFS.*|CONFIG_LFS=y|"                                            .config
      sed -i "s|.*CONFIG_CROSS_COMPILER_PREFIX.*|CONFIG_CROSS_COMPILER_PREFIX=${SET_A}|" .config
      sed -i "s|.*CONFIG_SYSROOT.*|CONFIG_SYSROOT=${SET_B}|"                             .config
      sed -i "s|.*CONFIG_EXTRA_CFLAGS.*|CONFIG_EXTRA_CFLAGS=${SET_C}|"                   .config
      sed -i "s|.*CONFIG_EXTRA_LDFLAGS.*|CONFIG_EXTRA_LDFLAGS=${SET_D}|"                 .config
      cat "${BUSYBOX_DIR}/.config" | grep -i cross
    fi
  USER_SHELL
  config.vm.provision "shell", inline: $user_busybox_config, privileged: false

  $user_busybox_build = <<-USER_SHELL
    echo "=== SCRIPT 11"
    source /home/vagrant/vars.sh

    echo "Build BusyBox"
    cd "${BUSYBOX_DIR}"
    CMD="make ARCH=${ARCH} -j ${CORES}"
    echo ${CMD}
    sleep 5
    ${CMD}
    make ARCH=${ARCH} install
    mv _install ${BASE}/filesystem
    du -h -d 1 ${BASE}/filesystem | tail -n 1
    find ${BASE}/filesystem | cpio -H newc -o | gzip -9 > ${BASE}/filesystem.cpio.gz
    du -h -d 1 ${BASE}/filesystem.cpio.gz | tail -n 1
    rm ${BASE}/filesystem.cpio.gz
  USER_SHELL
  config.vm.provision "shell", inline: $user_busybox_build, privileged: false

  $user_busybox_filesystem = <<-USER_SHELL
    echo "=== SCRIPT 12"
    source /home/vagrant/vars.sh

    echo "Populate Filesystem Directories"
    cd "${BASE}/filesystem"
    # Restore to local user if script previously run
    sudo chown -R $USER:$USER .
    mkdir -pv {dev,proc,etc,init.d,sys,tmp}
    sudo mknod dev/console c 5 1
    sudo mknod dev/null c 1 3

    echo "Populate Startup Message"
    MSG=${BASE}/filesystem/welcome
    KERNEL_VER="$(cd "${BASE}/linux/" ; git describe --always --dirty)"
    MUSL_VER="$(${BASE}/${MUSL_TARGET}${MUSL_CROSS}/${MUSL_TARGET}/lib/libc.so 2>&1 | grep -i version)"
    MUSL_CC_VER="$(${MUSL_CROSS_COMPILE}gcc --version 2>&1 | head -n 1)"
    CROSS_CC_VER="$(${CROSS_COMPILE}gcc --version 2>&1 | head -n 1)"
    HOST_CC_VER="$(gcc --version 2>&1 | head -n 1)"
    BUILD_DATE="$(date -u)"
    echo ""                                               > "${MSG}"
    echo "  --- Floppinux ---"                           >> "${MSG}"
    echo "  Linux Kernel    ${KERNEL_VER}"               >> "${MSG}"
    echo "  BusyBox Version ${BUSYBOX_VER}"              >> "${MSG}"
    echo "  MUSL       LIBC ${MUSL_VER}"                 >> "${MSG}"
    echo "  MUSL       CC   ${MUSL_CC_VER}"              >> "${MSG}"
    echo "  HOST CROSS CC   ${CROSS_CC_VER}"             >> "${MSG}"
    echo "  HOST       CC   ${HOST_CC_VER}"              >> "${MSG}"
    echo "  Date            ${BUILD_DATE}"               >> "${MSG}"
    echo "  -----------------"                           >> "${MSG}"
    echo ""                                              >> "${MSG}"
    cat "${MSG}"

    echo "Populate inittab"
    INITTAB=${BASE}/filesystem/etc/inittab
    echo "::sysinit:/init.d/rc"          > "${INITTAB}"
    echo "::askfirst:/bin/sh"           >> "${INITTAB}"
    echo "::restart:/sbin/init"         >> "${INITTAB}"
    echo "::ctrlaltdel:/sbin/reboot"    >> "${INITTAB}"
    echo "::shutdown:/bin/umount -a -r" >> "${INITTAB}"

    echo "Populate init script"
    INITSCRIPT=${BASE}/filesystem/init.d/rc
    echo "#!/bin/sh"                 > "${INITSCRIPT}"
    echo "mount -t proc none /proc" >> "${INITSCRIPT}"
    echo "mount -t sysfs none /sys" >> "${INITSCRIPT}"
    #echo "clear"                    >> "${INITSCRIPT}"
    echo "cat welcome"              >> "${INITSCRIPT}"
    echo "/bin/sh"                  >> "${INITSCRIPT}"
    chmod +x ${INITSCRIPT}

    echo "Apply chown to filesystem"
    cd "${BASE}/filesystem"
    sudo chown -R root:root .

    echo "Compress filesystem"
    [ -f ${BASE}/rootfs.cpio.xz ] && rm ${BASE}/rootfs.cpio.xz
    [ -f ${BASE}/rootfs.cpio.gz ] && rm ${BASE}/rootfs.cpio.gz

    # If using XZ compression, can make dictionary window larger than floppy size for best compression
    cd "${BASE}/filesystem"
    find . | cpio -H newc -o | xz --check=crc32 --lzma2=dict=10240KiB > ${BASE}/rootfs.cpio.xz

    # Gzip compression, has limited window sizes
    cd "${BASE}/filesystem"
    find . | cpio -H newc -o | gzip -9 > ${BASE}/rootfs.cpio.gz

    # Default to xz not gz due to improved compression
    du -h -d 1 "${BASE}/filesystem" | tail -n 1
    ls -lah ${BASE}/rootfs.cpio.*
    [ -f ${BASE}/rootfs.cpio.gz ] && rm ${BASE}/rootfs.cpio.gz
  USER_SHELL
  config.vm.provision "shell", inline: $user_busybox_filesystem, privileged: false

  $user_bootimg = <<-USER_SHELL
    echo "=== SCRIPT 13"
    source /home/vagrant/vars.sh

    echo "Create Syslinux Config"
    cd "${BASE}"
    SYS="${BASE}/syslinux.cfg"
    BUILD_DATE="$(date -u)"
    echo "DEFAULT linux"                          > "${SYS}"
    echo "LABEL linux"                           >> "${SYS}"
    echo "  SAY [ BOOTING LINUX ${BUILD_DATE} ]" >> "${SYS}"
    echo "  KERNEL bzImage"                      >> "${SYS}"
    if [ -f ${BASE}/rootfs.cpio.xz ] ; then
      echo "  APPEND initrd=rootfs.cpio.xz"      >> "${SYS}"
    else
      echo "  APPEND initrd=rootfs.cpio.gz"      >> "${SYS}"
    fi
    chmod +x "${SYS}"
  USER_SHELL
  config.vm.provision "shell", inline: $user_bootimg, privileged: false

  $user_bootfloppy = <<-USER_SHELL
    echo "=== SCRIPT 14"
    source /home/vagrant/vars.sh

    echo "Create 1.44M Floppy image file"
    FLOPPY="${BASE}/floppinux.img"
    cd "${BASE}"
    [ -f "${FLOPPY}" ] && rm "${FLOPPY}"
    dd if=/dev/zero of="${FLOPPY}" bs=1k count=1440
    mkdosfs "${FLOPPY}"
    syslinux --install "${FLOPPY}"

    echo "Place data onto floppy"
    DST=/mnt
    sudo mount -o loop "${FLOPPY}"   "${DST}"
    sudo cp "${BASE}/bzImage"        "${DST}"
    if [ -f ${BASE}/rootfs.cpio.xz ] ; then
      sudo cp "${BASE}/rootfs.cpio.xz" "${DST}"
    else
      sudo cp "${BASE}/rootfs.cpio.gz" "${DST}"
    fi
    sudo cp "${BASE}/syslinux.cfg"   "${DST}"

    echo "Floppy Content Size"
    cd "${DST}"
    du -k .
    cd "${BASE}"
    sudo umount "${DST}"

    echo "Artifact export"
    if [ -f "${FLOPPY}" ] ; then
      OUTFILE="$(date '+%Y-%m-%d_%H-%M-%S')"
      OUTFILE="${FLOPPY%.*}_${OUTFILE}.img"
      cp "${FLOPPY}" "${OUTFILE}"
      if [ -d /vagrant_data ] ; then
        cp "${OUTFILE}" /vagrant_data/
        ls -la "/vagrant_data/$(basename ${OUTFILE})"
      fi
    fi

    echo "Report Approximate Usage"
    gzip -c "${FLOPPY}" > "${FLOPPY}.gz"
    SZ_TOTAL=$(du -k "${FLOPPY}"    | cut -f1)
    SZ_COMPZ=$(du -k "${FLOPPY}.gz" | cut -f1)
    SZ_FRACT=$(( ( ${SZ_COMPZ} * 100 ) / ${SZ_TOTAL} ))
    echo "Total  $(printf "%10s" "${SZ_TOTAL}")"
    echo "Data   $(printf "%10s" "${SZ_COMPZ}")"
    echo "Approx $(printf "%10s" "${SZ_FRACT}")%"
  USER_SHELL
  config.vm.provision "shell", inline: $user_bootfloppy, privileged: false

  $user_bashrc_append = <<-USER_SHELL
    echo "=== SCRIPT BASHRC"
    . ~/.bashrc
    echo "Append vars.sh to .bashrc if missing"
    if [ "_${VARS}" == "_" ] ; then
      echo "source /home/vagrant/vars.sh" >> ~/.bashrc
    fi
    echo "Done BASHRC"
  USER_SHELL
  config.vm.provision "shell", inline: $user_bashrc_append, privileged: false
end
