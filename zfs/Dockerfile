FROM balenalib/intel-nuc-ubuntu

ENV UDEV=1

RUN install_packages vim cryptsetup curl wget uuid-dev libblkid-dev libssl-dev dkms zlib1g-dev

WORKDIR /usr/src/app

# set this accordingly for the balenaOS version
ENV OSVERSION '2.48.0+rev3.prod'
ENV KERNEL '5.2.10-yocto-standard'

# download and put kernel headers in /lib/modules
RUN mkdir --parents "/lib/modules/$KERNEL"
WORKDIR /lib/modules/$KERNEL
RUN wget --quiet "https://files.balena-cloud.com/images/intel-nuc/$(echo $OSVERSION | sed -e 's/+/%2B/g')/kernel_modules_headers.tar.gz"
RUN tar -xf kernel_modules_headers.tar.gz
RUN mv /lib/modules/$KERNEL/kernel_modules_headers /lib/modules/$KERNEL/build

RUN ln -s /lib64/ld-linux-x86-64.so.2  /lib/ld-linux-x86-64.so.2

# install zfs kernel modules and userland utils
WORKDIR /usr/src
RUN wget --quiet https://github.com/zfsonlinux/zfs/releases/download/zfs-0.8.3/zfs-0.8.3.tar.gz
RUN tar -xf zfs-0.8.3.tar.gz
RUN cd zfs-0.8.3
WORKDIR /usr/src/zfs-0.8.3
RUN ./configure --with-linux=/lib/modules/${KERNEL}/build/
RUN make
RUN make install
RUN ldconfig

WORKDIR /usr/src/app

CMD ["bash"]
