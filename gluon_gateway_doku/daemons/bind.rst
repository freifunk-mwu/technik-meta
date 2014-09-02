.. _bind:

Bind einrichten
===============

/etc/bind/named.conf.options::

    options {
        directory "/var/cache/bind";

        dnssec-validation auto;

        auth-nxdomain no;    # conform to RFC1035
        listen-on { 127.0.0.1; 10.37.0.X; 10.56.0.X; };
        listen-on-v6 { ::1; fd37:b4dc:4b1e::0a25:17; fd56:b4dc:4b1e::0a38:17; };

        allow-query { 127.0.0.1; ::1; intern-mz; intern-wi; };
        allow-recursion { 127.0.0.1; ::1; intern-mz; intern-wi; };
    };


**Wichtig**: *listen-on*, *listen-on-v6* auf sich selbst zeigen lassen.

Jedes Gate ist ein Slave für DNS (Das Vereins-Gate ist Master).

DNS-Master ist mangels Vereins-Gate erstmal auf Hinterschinken.

DNS-Master
----------

Die /etc/bind/named.conf.local bekommt pro Mesh-Wolke (z.B. für Mainz)::

    acl "ns-slaves-mz" {
        10.37.0.7/32;
        10.37.0.23/32;
        fd37:b4dc:4b1e::a25:7/128;
        fd37:b4dc:4b1e::a25:17/128;
    };

    acl "intern-mz" {
        10.37.0.0/16;
        fd37:b4dc:4b1e::/48;
    };

    // Intern Zones for Freifunk

    zone "ffmz.org." {
        type master;
        file "/etc/bind/ffmz/ffmz.org.master.db";
    };

    zone "user.ffmz.org." {
        type master;
        file "/etc/bind/ffmz/user.ffmz.org.master.db";
    };

    // Reverse Zones

    zone "37.10.in-addr.arpa" {
        type master;
        file "/etc/bind/ffmz/37.10.in-addr.arpa.master.db";
    };

    zone "e.1.b.4.c.d.4.b.7.3.d.f.ip6.arpa" {
        type master;
        file "/etc/bind/ffmz/fd37:b4dc:4b1e_48.ip6.arpa.master.db";
    };

Somit darf man nun die Zone-Files konfigurieren:

* /etc/bind/ffmz/37.10.in-addr.arpa.master.db
* /etc/bind/ffmz/fd37:b4dc:4b1e_48.ip6.arpa.master.db
* /etc/bind/ffmz/ffmz.org.master.db
* /etc/bind/ffmz/local.ffmz.org.master.db
* /etc/bind/ffmz/user.ffmz.org.master.db


Die Verwaltung der öffentlichen Domains übernimmt die separate VM *Linse*

DNS-Slave
---------

Die benötigten Ordner für die Zone-Files erstellen::

    mkdir /etc/bind/ffmz /etc/bind/ffwi
    chown -R bind:bind /etc/bind/ffmz /etc/bind/ffwi

Die /etc/bind/named.conf.local bekommt pro Mesh-Wolke (z.B. für Wiesbaden)::

    masters "ns-master-wi" {
        10.56.0.5;
    };

    acl "intern-wi" {
        10.56.0.0/16;
        fd56:b4dc:4b1e::/48;
    };

    // Intern Zones for Freifunk
    zone "ffwi.org." {
        type slave;
        file "/etc/bind/ffwi/ffwi.org.db";
        masters { ns-master-wi; };
    };

    zone "user.ffwi.org." {
        type slave;
        file "/etc/bind/ffwi/user.ffwi.org.db";
        masters { ns-master-wi; };
    };

    // Reverse Zones
    zone "56.10.in-addr.arpa" {
        type slave;
        file "/etc/bind/ffwi/56.10.in-addr.arpa.db";
        masters { ns-master-wi; };
    };

    zone "e.1.b.4.c.d.4.b.6.5.d.f.ip6.arpa" {
        type slave;
        file "/etc/bind/ffwi/fd56:b4dc:4b1e_48.ip6.arpa.db";
        masters { ns-master-wi; };
    };



Danach einen DNS-Eintrag auf sich selbst setzen:

:see:
    :ref:`self_dns`
