FROM ubuntu:xenial

MAINTAINER The xCAT Project

ENV container docker

ARG xcat_version=latest
ARG xcat_reporoot=http://xcat.org/files/xcat/repos/apt
ARG xcat_baseos=xenial

# Don't start any optional services except for the few we need.
RUN find /etc/systemd/system \
    /lib/systemd/system \
    -path '*.wants/*' \
    -not -name '*journald*' \
    -not -name '*systemd-tmpfiles*' \
    -not -name '*systemd-user-sessions*' \
    -exec rm \{} \;

RUN mkdir -p /xcatdata/etc/dhcp && rm -rf /etc/dhcp && ln -sf -t /etc /xcatdata/etc/dhcp && \
    mkdir -p /xcatdata/etc/goconserver && ln -sf -t /etc /xcatdata/etc/goconserver && \
    mkdir -p /xcatdata/etc/xcat && ln -sf -t /etc /xcatdata/etc/xcat && \
    mkdir -p /xcatdata/install && ln -sf -t / /xcatdata/install && \
    mkdir -p /xcatdata/tftpboot && ln -sf -t / /xcatdata/tftpboot && \
    mkdir -p /xcatdata/.xcat && ln -sf -t /root /xcatdata/.xcat

RUN echo "APT::Get::Install-Recommends \"false\";\nAPT::Get::Install-Suggests \"false\";" >> /etc/apt/apt.conf && \
    echo "deb [allow-insecure=yes] ${xcat_reporoot}/${xcat_version}/xcat-core  ${xcat_baseos} main" > /etc/apt/sources.list.d/xcat-core.list && \
    echo "deb [allow-insecure=yes] ${xcat_reporoot}/${xcat_version}/xcat-dep  ${xcat_baseos} main" > /etc/apt/sources.list.d/xcat-dep.list

# Workaround for systemctl issue in postinst script of below package
RUN apt-get update && apt-get install -y --allow-unauthenticated goconserver xcat-server || \
    bash -c "cat /dev/null > /var/lib/dpkg/info/xcat-server.postinst; cat /dev/null > /var/lib/dpkg/info/goconserver.postinst;dpkg --configure -a"

RUN apt-get update && apt-get install -y --allow-unauthenticated \
       xcat \
       sudo \
       psmisc \
       openssh-server \
       rsyslog && \
    apt-get clean

RUN sed -i -e 's|PermitRootLogin prohibit-password|PermitRootLogin yes|g' \
           -e 's|#PermitRootLogin prohibit-password|PermitRootLogin yes|g' \
           -e 's|#UseDNS yes|UseDNS no|g' /etc/ssh/sshd_config && \
    echo "root:cluster" | chpasswd && \
    sudo a2enmod ssl && \
    ln -s ../sites-available/default-ssl.conf /etc/apache2/sites-enabled/ssl.conf && \
    touch /etc/NEEDINIT

RUN systemctl enable apache2 && \
    systemctl enable ssh && \
    systemctl enable isc-dhcp-server && \
    systemctl enable rsyslog && \
    systemctl enable xcatd

ADD entrypoint.sh /etc/rc.d/rc.local
RUN chmod +x /etc/rc.d/rc.local ; systemctl enable rc-local

VOLUME [ "/xcatdata", "/var/log/xcat" ]

CMD [ "/sbin/init" ]