.. _repos_und_pakete:

.. _repositories:

Benötigte Repositories
======================

* Freifunk MWU Repository einbinden::

    add-apt-repository ppa:freifunk-mwu/freifunk-ppa

    add-apt-repository kann mittels::
    
    apt-get install software-properties-common 
    
    installiert werden 
* Neoraiders Repository einbinden (für fastd)::

    echo "deb http://repo.universe-factory.net/debian/ sid main" > /etc/apt/sources.list.d/freifunk.list
    apt-key adv --keyserver keyserver.ubuntu.com --recv 16EF3F64CB201D9C

* Repo für aktuelle BIRD Version::

    add-apt-repository ppa:cz.nic-labs/bird

Zum Schluss::

    apt-get update
    apt-get dist-upgrade


.. _pakete:

Pakete
======

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

