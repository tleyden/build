# Docker container for Docker 7.6

# See https://github.com/ceejatec/naked-docker/ for details about the
# construction of the base image.

FROM ceejatec/naked-debian:7.6
MAINTAINER ceej@couchbase.com

USER root

# Install SSH server
RUN apt-get update && \
    apt-get install -y openssh-server sudo && \
    rm -f /var/cache/apt/archives/*.deb && \
    mkdir /var/run/sshd # update 3

# Create couchbase user with password-less sudo privs, and give
# ownership of /opt/couchbase
RUN useradd couchbase -G sudo -m -s /bin/bash && \
    mkdir -p /opt/couchbase && chown -R couchbase:couchbase /opt/couchbase && \
    echo 'couchbase:couchbase' | chpasswd && \
    sed -ri 's/ALL\) ALL/ALL) NOPASSWD:ALL/' /etc/sudoers

# JDK for Jenkins.
RUN apt-get update && apt-get install -y openjdk-7-jre-headless && \
    rm -f /var/cache/apt/archives/*.deb

# GCC (from ceejatec/debian-gcc-build)
COPY build/local /usr/local
RUN  (echo "/usr/local/lib64"; cat /etc/ld.so.conf) > /tmp/ld.so.conf && \
     mv /tmp/ld.so.conf /etc && ldconfig && \
     ln -s gcc /usr/local/bin/cc && \
     dpkg -i /usr/local/debs/*.deb && \
     ln -s libgcc_s.so.1 /lib/x86_64-linux-gnu/libgcc_s.so

# Install Couchbase build dependencies.
# Note: lsb-release currently required for "cbdeps" third-party build
# mechanism, but we hope to remove that requirement as it brings in
# a lot of unnecessary packages.
RUN apt-get update && apt-get install -y libmpc2 libmpfr4 libgmp10 \
    ccache git-core tar libssl-dev libnuma-dev rubygems build-essential \
    rake ncurses-dev python-dev devscripts debhelper ed man curl libc6-i386 lsb-release \
    cpp-4.6- gcc-4.6- && \
    rm -f /var/cache/apt/archives/*.deb

# paramiko for testrunner
RUN apt-get update && apt-get install -y python-paramiko

# Install third-party build dependencies (python 2.6 can hopefully be removed
# in future)
RUN apt-get update && apt-get install -y m4 python2.6 python2.6-dev && \
    rm -f /var/cache/apt/archives/*.deb

# Install autoconf and friends - necessary for building some third-party deps
# from source, not for Couchbase.
RUN mkdir /tmp/deploy && \
    curl -L http://ftpmirror.gnu.org/autoconf/autoconf-2.69.tar.gz -o /tmp/deploy/autoconf-2.69.tar.gz && \
    cd /tmp/deploy && tar -xzf autoconf-2.69.tar.gz && \
    cd autoconf-2.69 && ./configure --prefix=/usr/local && make -j8 && make install && \
    curl -L http://ftpmirror.gnu.org/automake/automake-1.14.tar.gz -o /tmp/deploy/automake-1.14.tar.gz && \
    cd /tmp/deploy && tar -xzf automake-1.14.tar.gz && \
    cd automake-1.14 && ./configure --prefix=/usr/local && make -j8 && make install && \
    curl -L http://ftpmirror.gnu.org/libtool/libtool-2.4.2.tar.gz -o /tmp/deploy/libtool-2.4.2.tar.gz && \
    cd /tmp/deploy && tar -xzf libtool-2.4.2.tar.gz && \
    cd libtool-2.4.2 && ./configure --prefix=/usr/local && make -j8 && make install && \
    cd /tmp && rm -rf /tmp/deploy

RUN echo 'PATH="/usr/lib/ccache:$PATH"' >> /home/couchbase/.profile
RUN mkdir /tmp/deploy && \
    curl https://cmake.org/files/v2.8/cmake-2.8.12.2-Linux-i386.sh -o /tmp/deploy/cmake.sh && \
    (echo y; echo n) | sh /tmp/deploy/cmake.sh --prefix=/usr/local && \
    curl https://storage.googleapis.com/golang/go1.5.2.linux-amd64.tar.gz -o /tmp/deploy/go.tar.gz && \
    cd /usr/local && tar xzf /tmp/deploy/go.tar.gz && \
    cd bin && for file in /usr/local/go/bin/*; do ln -s $file; done && \
    curl https://storage.googleapis.com/git-repo-downloads/repo -o /usr/local/bin/repo && \
    chmod a+x /usr/local/bin/repo && \
    cd /tmp && rm -rf /tmp/deploy

# Expose SSH daemon and run our builder startup script
EXPOSE 22
ADD .ssh /home/couchbase/.ssh
COPY build/couchbuilder_start.sh /usr/sbin/
ENTRYPOINT [ "/usr/sbin/couchbuilder_start.sh" ]
CMD [ "default" ]



