# NetworkMananger-l2tp

NetworkManager-l2tp is a VPN plugin for NetworkManager 0.9.8 & 1.0 which
provides support for L2TP and L2TP/IPsec (i.e. L2TP over IPsec) connections.

For L2TP support, it uses xl2tpd ( https://www.xelerance.com/software/xl2tpd/ )

For IPsec support, it uses either of the following :
* Libreswan ( https://libreswan.org ) 
* strongSwan ( https://www.strongswan.org )

For details on pre-built packages, known issues and build dependencies,
please visit the Wiki :
* https://github.com/nm-l2tp/NetworkManager-l2tp/wiki

## Building

    ./autogen.sh
    ./configure  # (see below)
    make

The default ./configure settings aren't reasonable and should be explicitly
overridden with ./configure arguments. In the configure examples below, you
may need to change the `--with-pppd-plugin-dir` value to an appropriate
directory that exists.

#### Debian 8 - Jessie (AMD64, i.e. x86-64)

    ./configure \
      --disable-static --prefix=/usr \
      --sysconfdir=/etc --libdir=/usr/lib/x86_64-linux-gnu \
      --libexecdir=/usr/lib/NetworkManager \
      --localstatedir=/var \
      --with-pppd-plugin-dir=/usr/lib/pppd/2.4.6

#### Ubuntu 14.04 - Trusty Tahr

    ./configure \
      --disable-static --prefix=/usr \
      --sysconfdir=/etc \
      --libexecdir=/usr/lib/NetworkManager \
      --localstatedir=/var \
      --with-pppd-plugin-dir=/usr/lib/pppd/2.4.5

#### Fedora 23 (x86-64)

    ./configure \
      --disable-static --prefix=/usr \
      --sysconfdir=/etc --libdir=/usr/lib64 \
      --localstatedir=/var \
      --with-pppd-plugin-dir=/usr/lib64/pppd/2.4.7

## Issue with not stopping system xl2tpd service

NetworkManager-l2tp starts its own instance of xl2tpd and if the system xl2tpd
service is running, its own xl2tpd instance will not be able to use UDP port
1701, so will use an ephemeral port (i.e. random high port).

Although the use of an ephemeral port is considered acceptable in RFC3193, the
L2TP/IPsec standard co-authored by Microsoft and Cisco, there are some
L2TP/IPsec servers and/or firewalls that will have issues if an ephemeral port
is used.

Stopping the system xl2tpd service should free UDP port 1701 and on systemd
based Linux distributions, the xl2tpd service can be stopped with the
following:

    sudo systemctl stop xl2tpd

If stopping the xl2tpd service fixes your VPN connection issue, you can
disable the xl2tpd service from starting at boot time with :

    sudo systemctl disable xl2tpd

## Run-time generated files

The below files located under `/var/run` assume `--localstatedir=/var` or
`--runstatedir=/var/run` were supplied to the configure script at build time.

* /var/run/nm-l2tp-xl2tpd-_UUID_.conf
* /var/run/nm-l2tp-ppp-options-_UUID_
* /var/run/nm-l2tp-xl2tpd-control-_UUID_
* /var/run/nm-l2tp-xl2tpd-_UUID_.pid
* /var/run/nm-l2tp-ipsec-_UUID_.conf
* /etc/ipsec.d/ipsec.nm-l2tp.secrets

where _UUID_ is the NetworkManager UUID for the VPN connection.

NetworkManager-l2tp will append the following line to `/etc/ipsec.secrets` at
run-time if the line is missing:

    include ipsec.nm-l2tp.secrets

## Debugging mode

Issue the following on the command line which will increase xl2tpd and pppd
debugging, also the run-time generated config files will not be cleaned up
after a VPN disconnection :

#### Debian and Ubuntu
    sudo killall -TERM nm-l2tp-service
    sudo /usr/lib/NetworkManager/nm-l2tp-service --debug

#### Fedora
    sudo killall -TERM nm-l2tp-service
    sudo /usr/libexec/nm-l2tp-service --debug

then start your VPN connection and reproduce the problem.

NetworkManager and pppd logging goes to the Systemd journal which can be viewed
by issuing the following which will show the logs since the last boot:

    journalctl --boot

For non-Systemd based Linux distributions, view the appropriate system log
file which is most likely located under `/var/log/`.

## User specified IPsec IKEv1 cipher suites

User specified phase 1 (ike) and phase 2 (esp) cipher suites can be specified
in the IPsec configuration dialog box under Advanced options.

