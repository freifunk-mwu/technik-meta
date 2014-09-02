.. _interfaces:

Netzwerk Interfaces
===================

/etc/network/interfaces

Wir richten uns hier je eine Brücke pro Mesh-Wolke die wir versorgen wollen ein.

Auf diese Brücken binden sich die Dienste (wie DHCP, DNS, NTP, etc..).
Das hoch/runterfahren der VPNs oder das (neu-)starten von Diensten kann dadurch unabhängig voneinander geschehen.

Hier eine Brücke mit IPv4 und IPv6 am Beispiel von Wiesbaden::

    auto wiBR
    iface wiBR inet static
        bridge-ports none
        address 10.56.0.X
        netmask 255.255.192.0
        # add all marked traffic to the appropriate rt_table
        post-up         /sbin/ip rule add iif $IFACE table wi priority 5600
        pre-down        /sbin/ip rule del iif $IFACE table wi priority 5600
        # default route is unreachable
        post-up         /sbin/ip route add unreachable default table wi
        post-down       /sbin/ip route del unreachable default table wi
        # local reachable subnet
        post-up         /sbin/ip route add 10.56.0.0/18 src 10.56.0.X dev $IFACE table wi
        post-down       /sbin/ip route del 10.56.0.0/18 src 10.56.0.X dev $IFACE table wi
        # insert ic-vpn light route for wi network into rt_table mz
        post-up         /sbin/ip route add 10.56.0.0/18 src 10.56.0.X dev $IFACE table mz
        post-down       /sbin/ip route del 10.56.0.0/18 src 10.56.0.X dev $IFACE table mz

    iface wiBR inet6 static
        address fd56:b4dc:4b1e::a38:X
        netmask 64
        # add all marked traffic to the appropriate rt_table
        post-up         /sbin/ip -6 rule add iif $IFACE table wi priority 5600
        pre-down        /sbin/ip -6 rule del iif $IFACE table wi priority 5600
        # default route is unreachable
        post-up         /sbin/ip -6 route add unreachable default table wi
        post-down       /sbin/ip -6 route del unreachable default table wi
        # link-local subnet
        post-up         /sbin/ip -6 route add fe80::/64 dev $IFACE table wi
        post-down       /sbin/ip -6 route del fe80::/64 dev $IFACE table wi
        post-up         /sbin/ip -6 route add fd56:b4dc:4b1e::/64 dev $IFACE table wi
        post-down       /sbin/ip -6 route del fd56:b4dc:4b1e::/64 dev $IFACE table wi
        # insert ic-vpn light route for wi network into rt_table mz
        post-up         /sbin/ip -6 route add fd56:b4dc:4b1e::/64 dev $IFACE table mz
        post-down       /sbin/ip -6 route del fd56:b4dc:4b1e::/64 dev $IFACE table mz

.. TODO: Warum wird unter *inet* bridge-ports none definiert, unter *inet6* aber nicht?

:see:
    - :ref:`routing_table`
    - :ref:`gateway_schema`
    - :ref:`interface_bezeichnung`

Wir haben uns dazu entschieden jeglichs Up- & Downscript in der /etc/network/interfaces zu verwalten.
Dies gestaltet alles übersichtlicher.

Scripte für :ref:`fastd`::

    allow-hotplug wiVPN
    iface wiVPN inet6 manual
        pre-up          /sbin/modprobe batman_adv
        pre-up          /sbin/ip link set address 02:00:0a:38:00:X dev $IFACE
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

.. _self_dns:

DNS-Eintrag für das System selbst
---------------------------------

Nach dem die Konfiguration von Bind abgeschlossen wird ein DNS-Eintrag auf sich gesetzt.

Dies kommt in die inet Section des Internet Interfaces, i.d.R. eth0.

Dadurch wird der Nameserver-Eintrag durch **resolvconf** beim Hochkommen des Interfaces nach ``/etc/resolv.conf`` geschrieben

in die /etc/network/interfaces kommt also folgendes::

    iface eth0 inet static
        [...]
        dns-nameservers 127.0.0.1

:see:
    - :ref:`bind`
