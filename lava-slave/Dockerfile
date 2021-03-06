FROM bitnami/minideb:stretch

RUN apt-get update

# e2fsprogs is for libguestfs
RUN \
 echo 'lava-server   lava-server/instance-name string lava-slave-instance' | debconf-set-selections && \
 echo 'locales locales/locales_to_be_generated multiselect C.UTF-8 UTF-8, en_US.UTF-8 UTF-8 ' | debconf-set-selections && \
 echo 'locales locales/default_environment_locale select en_US.UTF-8' | debconf-set-selections && \
 DEBIAN_FRONTEND=noninteractive apt-get -y install \
 locales \
 vim \
 sudo \
 cu \
 conmux \
 python-setproctitle \
 tftpd-hpa \
 u-boot-tools \
 device-tree-compiler \
 qemu-system \
 qemu-system-arm \
 qemu-system-i386 \
 xnbd-server \
 e2fsprogs

RUN if [ "$(uname -m)" = "x86_64" -o "$(uname -m)" = "x86" ] ;then apt-get -y install qemu-kvm ; fi

# Uncomment the following for having the latest package (you need to comment all build-lava below)
#RUN DEBIAN_FRONTEND=noninteractive apt-get -y install wget gnupg \
# && wget http://images.validation.linaro.org/production-repo/production-repo.key.asc \
# && apt-key add production-repo.key.asc \
# && echo 'deb http://images.validation.linaro.org/production-repo/ stretch-backports main' > /etc/apt/sources.list.d/lava.list \
# && apt-get clean && apt-get update
#RUN DEBIAN_FRONTEND=noninteractive apt-get -y install lava-dispatcher

COPY scripts/build-lava /
RUN sh /build-lava lava-dispatcher 2017.11 2017.11.post1-1

COPY configs/lava-slave /etc/lava-dispatcher/lava-slave

COPY configs/tftpd-hpa /etc/default/tftpd-hpa

COPY scripts/cu-loop /usr/local/bin/
COPY conmux/* /etc/conmux/

# Caution to not use any port between the Linux dynamic port range: 32768-60999
RUN sed -i 's,XNBD_PORT_RANGE_MIN.*,XNBD_PORT_RANGE_MIN=61950,' /usr/lib/python2.7/dist-packages/lava_dispatcher/utils/constants.py
RUN sed -i 's,XNBD_PORT_RANGE_MAX.*,XNBD_PORT_RANGE_MAX=62000,' /usr/lib/python2.7/dist-packages/lava_dispatcher/utils/constants.py

#conmux need cu >= 1.07-24 See https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=336996
RUN echo "deb http://debian.proxad.net/debian/ testing main" >> /etc/apt/sources.list.d/testing.list
RUN apt-get update
RUN DEBIAN_FRONTEND=noninteractive apt-get -y install cu
RUN rm /etc/apt/sources.list.d/testing.list

# Add services helper utilities to start and stop LAVA
COPY scripts/stop.sh .
COPY scripts/start.sh .

EXPOSE 69/udp 80

CMD /start.sh
