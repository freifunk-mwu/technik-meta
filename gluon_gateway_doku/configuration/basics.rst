.. _basics:

Grundkonfiguration
==================

.. _hostname:

Hostname
--------

Den Hostnamen setzen (ohne Domain)::

    echo "hostname" > /etc/hostname

/etc/hosts (am Beispiel von Lotuswurzel)::

    127.0.0.1       localhost
    144.76.209.100    lotuswurzel.freifunk-mwu.de       lotuswurzel

    # The following lines are desirable for IPv6 capable hosts
    ::1     localhost ip6-localhost ip6-loopback
    ff02::1 ip6-allnodes
    ff02::2 ip6-allrouters
    2a01:4f8:192:520::b4dc:4b1e    lotuswurzel.freifunk-mwu.de    lotuswurzel

.. _routing_tables:

Routing Tabellen
----------------

/etc/iproute2/rt_tables::

    [...]
    37      mz
    56      wi
    # icvpn
    42      icvpn
    [...]

.. _kernel_parameters:

Wichtige Kernel Parameter
-------------------------

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

.. _repositories:

Benötigte Repositories
----------------------

* Freifunk MWU Repository einbinden::

    add-apt-repository ppa:freifunk-mwu/freifunk-ppa

* add-apt-repository Installieren (optional)::

    apt-get install software-properties-common

* Neoraiders Repository einbinden (für fastd)::

    echo "deb http://repo.universe-factory.net/debian/ sid main" > /etc/apt/sources.list.d/freifunk.list
    apt-key adv --keyserver keyserver.ubuntu.com --recv 16EF3F64CB201D9C

* Repo für aktuelle BIRD Version::

    add-apt-repository ppa:cz.nic-labs/bird

Zum Schluss::

    apt-get update
    apt-get dist-upgrade

.. _crontab_path:

Crontab PATH
------------

    PATH=/home/admin/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

.. _packages:

Pakete
------

Pakete aus den Standard-Repos installieren::

    xargs apt-get install -y

        bridge-utils
        git
        iproute
        iptables
        iptables-persistent
        isc-dhcp-server
        ntp
        #openssl
        openvpn
        python3
        python-argparse
        radvd
        rrdtool
        bind9
        mosh
        man-db
        vim
        tinc
        bird

Pakete aus den eigenen Repositories installieren::

    xargs apt-get install -y

        fastd
        batctl
        batman-adv-dkms
        alfred
        alfred-json
        batadv-vis

An dieser Stelle sei auf die :ref:`scripts` hingewiesen. Dort ist hinterlegt wie diese installiert und eingerichtet werden
