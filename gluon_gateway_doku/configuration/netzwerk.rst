.. _netzwerk:

.. _hostname:

Hostname setzen
===============

Den Hostnamen setzen (ohne Domain)::

    echo "hostname" > /etc/hostname

/etc/hosts (am Beispiel von Lotuswurzel)::

    127.0.0.1       localhost
    176.9.204.70    lotuswurzel.freifunk-wiesbaden.de       lotuswurzel

    # The following lines are desirable for IPv6 capable hosts
    ::1     localhost ip6-localhost ip6-loopback
    ff02::1 ip6-allnodes
    ff02::2 ip6-allrouters
    2a01:4f8:150:8570::2    lotuswurzel.freifunk-wiesbaden.de    lotuswurzel

.. _routing_table:

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

.. _ip_forward:

Wichtige Kernel Parameter
=========================

/etc/sysctl.conf::

    # Freifunk specific settings
    net.ipv4.ip_forward=1

    net.bridge.bridge-nf-call-arptables = 0
    net.bridge.bridge-nf-call-ip6tables = 0
    net.bridge.bridge-nf-call-iptables = 0

    net.ipv6.conf.all.forwarding=1

    net.ipv6.conf.all.autoconf = 0
    net.ipv6.conf.default.autoconf = 0
    net.ipv6.conf.eth0.autoconf = 0

    net.ipv6.conf.all.accept_ra = 0
    net.ipv6.conf.default.accept_ra = 0
    net.ipv6.conf.eth0.accept_ra = 0

Danach neuladen::

    sysctl -p /etc/sysctl.conf
