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

Danach Ordner für die zone files erstellen::

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
