.. _dhcp:

DHCPd einrichten
================


Aus dem :ref:`netzplan` wird sich eine Rage gezogen.

In den Header der /etc/dhcp/dhcpd.conf kommt::

    default-lease-time 300;
    max-lease-time 3600;

Wir wählen hier eine kurze Lease Time, damit die Clients maximal 5 Minuten offline sind.

Pro Mesh-Wolke verteilen wir jeweils eine Range (z.B. für Wiesbaden)::

    subnet 10.56.0.0 netmask 255.255.192.0 {
        authoritative;
        range 10.56.16.0 10.56.31.255;

        # Use our own IP as gateway for our clients
        option routers 10.56.0.X;

        # DNS servers to be pushed to our clients.
        # This will usually be our IP address and all other
        # gateways, too.
        option domain-name-servers 10.56.0.X, 10.56.0.Y, 10.56.0.Z;

        option domain-name ".ffwi.org";
        option domain-search "ffwi.org", "user.ffwi.org";

        # ntp servers
        option ntp-servers 10.56.0.X, 10.56.0.Y, 10.56.0.Z;
    }

Wichtig:
*domain-name-server* und *ntp-servers* auf sich selbst und alle anderen Gates setzen.

Unter etc/default/isc-dhcp-server konfigurieren wir, auf welchen Interfaces der dhcpd lauschen soll.

:see:
    - :ref:`interfaces`

Wir wählen die beiden Brücken::

    INTERFACES="mzBR wiBR"

.. _radvd:

RAdvD einrichten
================

Die Konfigurationsdatei muss man sich selbst erzeugen. Es gibt Beispiele unter ``/usr/share/doc/radvd/examples/``.

Pro Mesh-Wolke verteilen wir jeweils ein Prefix.

/etc/radvd.conf (z.B. für Mainz)::

    interface mzBR
    {
        AdvSendAdvert on;
        IgnoreIfMissing on;
        MaxRtrAdvInterval 200;

        prefix fd37:b4dc:4b1e::/64
        {};

        RDNSS fd37:b4dc:4b1e::a25:X fd37:b4dc:4b1e::a25:Y fd37:b4dc:4b1e::a25:Z
        {};
    };

Wichtig:

*RDNSS* auf sich selbst und alle anderen Gates setzen.
