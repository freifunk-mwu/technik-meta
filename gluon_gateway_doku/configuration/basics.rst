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

    #
    # reserved values
    #
    255     local
    254     main
    253     default
    0       unspec
    #
    # local
    #
    37      mz
    56      wi
    # icvpn
    42      icvpn

.. _kernel_parameters:

Wichtige Kernel Parameter
-------------------------

/etc/sysctl.conf::

    # Freifunk specific settings
    net.ipv4.ip_forward=1
    net.ipv6.conf.all.forwarding=1

    net.ipv6.conf.all.autoconf = 0
    net.ipv6.conf.default.autoconf = 0
    net.ipv6.conf.eth0.autoconf = 0

    net.ipv6.conf.all.accept_ra = 0
    net.ipv6.conf.default.accept_ra = 0
    net.ipv6.conf.eth0.accept_ra = 0

    # <3.18 Kernel noch folgendes hinzufügen:
    net.bridge.bridge-nf-call-arptables = 0
    net.bridge.bridge-nf-call-ip6tables = 0
    net.bridge.bridge-nf-call-iptables = 0

Der letzte Teil sorgt dafür, dass die firewall (ip(6)tables) und arptables nicht im Bridgeinterface filtern. Bridges sollen wirklich nur als Switch laufen!
In neueren Kerneln gibt es jetzt ein eigenes Modul (brdige_filter) dass man laden muss, um das zu aktivieren, bis jetzt ist dieses verhalten per default an.

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

.. _packages:

Pakete
------

Pakete aus den Standard-Repos installieren::

        apt-get install -y apache2 bind9 bird bridge-utils git iproute iptables \ 
        iptables-persistent isc-dhcp-server man-db mosh ntp openvpn python-argparse \
        python3 python3-netifaces  radvd rrdtool sysfsutils tinc vim vnstat vnstati

Pakete aus den eigenen Repositories installieren::

    apt-get install -y alfred alfred-json batadv-vis batctl batman-adv-dkms fastd

.. _sysfs_parameter

Sysfs Parameter
---------------

Wir erhöhen auf den Gateways die Hop Penalty auf den Wert 60, damit mehr Traffic über Wifi Links geschickt wird.

Die Datei ``/etc/sysfs.d/99-batman-hop-penalty.conf`` muss mit folgendem Inhalt angelegt werden::

    class/net/mzBAT/mesh/hop_penalty = 60
    class/net/wiBAT/mesh/hop_penalty = 60

Diese Einstellung ist prinzipiell für jedes Batman Interface vorzunehmen, hier am Beispiel von ``mzBAT`` und ``wiBAT``.


.. _ntp:

NTP
---

Da die Kisten recht viel mit Crypto machen, ist es von Vorteil eine halbwegs genaue Uhrzeit parat zu haben.

Die ``/etc/ntp.conf`` bleibt nahezu unverändert::

    # /etc/ntp.conf, configuration for ntpd; see ntp.conf(5) for help

    driftfile /var/lib/ntp/ntp.drift

    # Specify one or more NTP servers.
    server 0.de.pool.ntp.org
    server 1.de.pool.ntp.org
    server 2.de.pool.ntp.org
    server 3.de.pool.ntp.org

    # Use Ubuntu's ntp server as a fallback.
    server ntp.ubuntu.com

    # By default, exchange time with everybody, but don't allow configuration.
    restrict -4 default kod notrap nomodify nopeer noquery
    restrict -6 default kod notrap nomodify nopeer noquery

    # Local users may interrogate the ntp server more closely.
    restrict 127.0.0.1
    restrict ::1

Im :ref:`dhcp` werden alle Gateways als Zeitquellen konfiguriert und verteilt.
