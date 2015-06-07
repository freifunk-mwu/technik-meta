.. _ddi:

DNS-DHCP-IP
===========

.. _bind:

BIND
----

/etc/bind/named.conf.options::

    options {
        directory "/var/cache/bind";

        dnssec-validation auto;

        auth-nxdomain no;    # conform to RFC1035
        listen-on { 127.0.0.1; 10.37.0.X; 10.56.0.X; 10.207.0.X; };
        listen-on-v6 { ::1; fd37:b4dc:4b1e::a25:17; fd56:b4dc:4b1e::a38:17; fec0::a:cf:0:38; };

        allow-transfer { any; };
        allow-query { any; };
        allow-recursion { 127.0.0.1; ::1; intern-mz; intern-wi; };
    };


**Wichtig**: *listen-on*, *listen-on-v6* ausschließlich auf die lokalen Interfaces und die IC-VPN Interfaces zeigen lassen.

Jedes Gate ist ein Slave für DNS (DNS Master läuft auf einem Service Server).

DNS-Slave
`````````

Die /etc/bind/named.conf.local bekommt pro Mesh-Wolke (z.B. für Wiesbaden)::

    masters "ns-master-wi" {
        fd56:b4dc:4b1e::a38:103;
    };

    acl "intern-wi" {
        10.56.0.0/16;
        fd56:b4dc:4b1e::/48;
    };

    // Intern Zones for Freifunk
    zone "ffwi.org." {
        type slave;
        file "ffwi.org.db";
        masters { ns-master-wi; };
    };

    zone "user.ffwi.org." {
        type slave;
        file "user.ffwi.org.db";
        masters { ns-master-wi; };
    };

    // Reverse Zones
    zone "56.10.in-addr.arpa" {
        type slave;
        file "56.10.in-addr.arpa.db";
        masters { ns-master-wi; };
    };

    zone "e.1.b.4.c.d.4.b.6.5.d.f.ip6.arpa" {
        type slave;
        file "fd56:b4dc:4b1e_48.ip6.arpa.db";
        masters { ns-master-wi; };
    };



Danach einen DNS-Eintrag auf sich selbst setzen:

.. seealso::
    :ref:`self_dns`

.. _dhcp:

DHCPd
-----

Aus dem :ref:`netzplan` wird sich eine Rage gezogen.

In den Header der /etc/dhcp/dhcpd.conf kommt::

    ddns-update-style none;

    authoritative;
    server-name "lotuswurzel";

    log-facility local6;

    default-lease-time 300;
    min-lease-time 300;
    max-lease-time 300;

Die Direktive **server-name** ist auf den Hostnamen des jeweiligen Gateways anzupassen.
Wir wählen hier eine kurze Lease Time, damit die Clients maximal 5 Minuten offline sind.

Pro Mesh-Wolke verteilen wir jeweils eine Range (z.B. für Wiesbaden)::

    subnet 10.56.0.0 netmask 255.255.192.0 {
        range 10.56.16.0 10.56.31.255;

        # Use our own IP as gateway for our clients
        option routers 10.56.0.X;

        # DNS servers to be pushed to our clients.
        # This will usually be our IP address and all other
        # gateways, too.
        option domain-name-servers 10.56.0.X, 10.56.0.Y, 10.56.0.Z;
        option domain-search "ffwi.org", "user.ffwi.org";

        # NTP Servers pushed to our clients
        # This will usually be our IP address and all other
        # gateways, too.
        option ntp-servers 10.56.0.X, 10.56.0.Y, 10.56.0.Z;

        # Set interface mtu
        option interface-mtu 1350;
    }

Wichtig:
*domain-name-server* und *ntp-servers* auf sich selbst und alle anderen Gates setzen.

Unter etc/default/isc-dhcp-server konfigurieren wir, auf welchen Interfaces der dhcpd lauschen soll.

Wir wählen die beiden Brücken::

    INTERFACES="mzBR wiBR"

.. seealso::
    - :ref:`interfaces`
    - :ref:`self_dns`
    - :ref:`ntp`

.. _radvd:

RAdvD
-----

Die Konfigurationsdatei muss manuell angelegt werden.

Pro Mesh-Wolke verteilen wir jeweils ein Prefix.

/etc/radvd.conf (z.B. für Mainz)::

    interface mzBR
    {
        AdvSendAdvert on;
        IgnoreIfMissing on;
        MaxRtrAdvInterval 900;
        AdvLinkMTU 1350;

        prefix fd37:b4dc:4b1e::/64
        {
                AdvValidLifetime 864000;
                AdvPreferredLifetime 172800;
        };

        RDNSS fd37:b4dc:4b1e::a25:X fd37:b4dc:4b1e::a25:Y fd37:b4dc:4b1e::a25:Z
        {
                FlushRDNSS off;
        };
    };

Wichtig:

*RDNSS* auf sich selbst und alle anderen Gates setzen. Es sind maximal 3 IP-Adressen zulässig.
