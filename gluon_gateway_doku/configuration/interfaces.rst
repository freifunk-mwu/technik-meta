.. _interfaces:

Netzwerk Interfaces
===================

/etc/network/interfaces

Wir richten uns hier je eine Brücke pro Mesh-Wolke, die wir versorgen wollen, ein.

Auf diese Brücken binden sich die Dienste (wie DHCP, DNS, NTP, etc..).
Das hoch/runterfahren der VPNs oder das (neu-)starten von Diensten kann dadurch unabhängig voneinander geschehen.

.. seealso::
    - :ref:`fastd`
    - :ref:`dhcp`
    - :ref:`routing_tables`
    - :ref:`policyrouting`
    - :ref:`exitvpn`

Hier eine Brücke mit IPv4 und IPv6 am Beispiel von Wiesbaden::

    auto wiBR
    iface wiBR inet static
        bridge_ports none
        bridge_fd 0
        bridge_maxwait 0
        address 10.56.0.X
        netmask 255.255.192.0

    iface wiBR inet6 static
        address fd56:b4dc:4b1e::a38:X
        netmask 64

.. seealso::
    - :ref:`routing_tables`
    - :ref:`policyrouting`
    - :ref:`gateway_schema`
    - :ref:`interface_bezeichnung`

Wir haben uns dazu entschieden jegliche Up- & Down Scripte in der /etc/network/interfaces zu verwalten.
Dies gestaltet alles übersichtlicher.

Scripte für :ref:`fastd`::

    allow-hotplug wiVPN
    iface wiVPN inet6 manual
        hwaddress 02:00:0a:38:00:X
        pre-up          /sbin/modprobe batman-adv
        post-up         /usr/sbin/batctl -m wiBAT if add $IFACE
        post-up         /sbin/ip link set dev wiBAT up

Zum Schluss noch für das B.A.T.M.A.N. Interface::

    allow-hotplug wiBAT
    iface wiBAT inet6 manual
        pre-up          /sbin/modprobe batman-adv
        post-up         /sbin/brctl addif wiBR $IFACE
        post-up         /usr/sbin/batctl -m $IFACE it 10000
        post-up         /usr/sbin/batctl -m $IFACE vm server
        post-up         /usr/sbin/batctl -m $IFACE gw server  96mbit/96mbit
        pre-down        /sbin/brctl delif wiBR $IFACE || true

