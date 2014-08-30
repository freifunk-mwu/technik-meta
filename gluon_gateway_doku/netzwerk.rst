.. _netzwerk

Hostname setzen
===============

``echo "hostname" > /etc/hostname`` nur hostname ohne domain

/etc/hosts (am Beispiel von Lotuswurzel)::

    127.0.0.1       localhost
    176.9.204.70    lotuswurzel.freifunk-wiesbaden.de       lotuswurzel

    # The following lines are desirable for IPv6 capable hosts
    ::1     localhost ip6-localhost ip6-loopback
    ff02::1 ip6-allnodes
    ff02::2 ip6-allrouters
    2a01:4f8:150:8570::2    lotuswurzel.freifunk-wiesbaden.de    lotuswurzel

Routing Tables einrichten
=========================

/etc/iproute2/rt_tables::

    [...]
    37      mz
    56      wi
    # icvpn
    370     icvpn-mz
    560     icvpn-wi
    [...]

IP Forwarding setzen
====================

/etc/sysctl.conf::

    net.ipv4.ip_forward = 1
    net.ipv6.conf.all.forwarding = 1

Danach neuladen::

    sysctl -p /etc/sysctl.conf
