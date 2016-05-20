.. _policyrouting:

Policy Routing
==============

Wir setzen auf unseren Gateways Policy Routing ein, um den Netzwerkverkehr in die richtige Routing-Tabelle zu schubsen.

Wie Policy Routing unter Linux funktioniert kann hier nachgelesen werden:

* `LARTC`_
* `Policy-Routing`_

Unter Linux konfiguriert man Routing Policies über sogenannte **IP Rules**. Damit diese immer vorhanden sind lassen wir diese über das **rc.local**-Script setzen.

/etc/rc.local::

    #
    # IP rules
    #

    # Priority 7 - lookup rt_table mwu for all traffic to and from own freifunk networks
    ip -4 rule add from 10.37.0.0/16 lookup mwu priority 7
    ip -4 rule add to 10.37.0.0/16 lookup mwu priority 7
    ip -4 rule add from 10.56.0.0/16 lookup mwu priority 7
    ip -4 rule add to 10.56.0.0/16 lookup mwu priority 7
    ip -4 rule add from all oif mzBR lookup mwu priority 7
    ip -4 rule add from all oif wiBR lookup mwu priority 7
    ip -4 rule add from all oif icVPN lookup mwu priority 7
    ip -6 rule add from fd37:b4dc:4b1e::/48 lookup mwu priority 7
    ip -6 rule add to fd37:b4dc:4b1e::/48 lookup mwu priority 7
    ip -6 rule add from fd56:b4dc:4b1e::/48 lookup mwu priority 7
    ip -6 rule add to fd56:b4dc:4b1e::/48 lookup mwu priority 7
    ip -6 rule add from all oif mzBR lookup mwu priority 7
    ip -6 rule add from all oif wiBR lookup mwu priority 7
    ip -6 rule add from all oif icVPN lookup mwu priority 7
    ...
    <IPv6 rules für öffentliche IPv6-Präfixe / Internet Exit>
    ...

    # Priortiy 23 - lookup rt_table icvpn for all traffic to and from ICVPN
    ip -4 rule add from 10.37.0.0/16 lookup icvpn priority 23
    ip -4 rule add to 10.37.0.0/16 lookup icvpn priority 23
    ip -4 rule add from 10.56.0.0/16 lookup icvpn priority 23
    ip -4 rule add to 10.56.0.0/16 lookup icvpn priority 23
    ip -4 rule add from all oif mzBR lookup icvpn priority 23
    ip -4 rule add from all oif wiBR lookup icvpn priority 23
    ip -4 rule add from all oif icVPN lookup icvpn priority 23
    ip -6 rule add from fd37:b4dc:4b1e::/48 lookup icvpn priority 23
    ip -6 rule add to fd37:b4dc:4b1e::/48 lookup icvpn priority 23
    ip -6 rule add from fd56:b4dc:4b1e::/48 lookup icvpn priority 23
    ip -6 rule add to fd56:b4dc:4b1e::/48 lookup icvpn priority 23
    ip -6 rule add from all oif mzBR lookup icvpn priority 23
    ip -6 rule add from all oif wiBR lookup icvpn priority 23
    ip -6 rule add from all oif icVPN lookup icvpn priority 23
    ...
    <IPv6 rules für öffentliche IPv6-Präfixe / Internet Exit>
    ...

    # Priority 41 - lookup rt_table ffinetexit for all traffic related own freifunk networks to and from public internet
    ip -4 rule add from 10.37.0.0/16 lookup ffinetexit priority 41
    ip -4 rule add from 10.56.0.0/16 lookup ffinetexit priority 41
    ...
    <IPv4 rules für Internet Exit>
    <IPv6 rules für Internet Exit>
    ...

    # Priority 61 - at this point this is the end of policy routing for freifunk related traffic
    ip -4 rule add to 10.0.0.0/8 type unreachable priority 61
    ip -4 rule add from 10.0.0.0/8 type unreachable priority 61
    ip -4 rule add to 192.168.0.0/16 type unreachable priority 61
    ip -4 rule add from 192.168.0.0/16 type unreachable priority 61
    ip -4 rule add to 172.16.0.0/12 type unreachable priority 61
    ip -4 rule add from 172.16.0.0/12 type unreachable priority 61
    ip -4 rule add from all iif mzBR type unreachable priority 61
    ip -4 rule add from all iif wiBR type unreachable priority 61
    ip -4 rule add from all iif icVPN type unreachable priority 61
    ip -4 rule add from all iif eth0 type unreachable priority 61
    ...
    <IPv4 rules für Internet Exit>
    ...
    ip -6 rule add from fc00::/7 type unreachable priority 61
    ip -6 rule add to fc00::/7 type unreachable priority 61
    ip -6 rule add from all iif mzBR type unreachable priority 61
    ip -6 rule add from all iif wiBR type unreachable priority 61
    ip -6 rule add from all iif icVPN type unreachable priority 61
    ip -6 rule add from all iif eth0 type unreachable priority 61
    ...
    <IPv6 rules für Internet Exit>
    ...

    # Priority 107 - lookup policies for the gateway host self originating traffic
    ip -4 rule add from all lookup mwu priority 107
    ip -4 rule add from all lookup icvpn priority 107
    ip -6 rule add from all lookup mwu priority 107
    ip -6 rule add from all lookup icvpn priority 107

Wie zu erkennen ist verwenden wir 3 Routing Tabellen:

* icvpn (wird dynamisch über BGP gefüllt)
* mwu (enthält statische Routen der Community-Netze)
* ffinetexit (enthält Routen für den Internet-Verkehr)

Die IP Rules für den Internet-Exit hängen von der gewählten Variante ab, mehr Informationen dazu im Abschnitt :ref:`internetexit`

Zusätzlich zu den **IP Rules** befüllen wir über das **rc.local**-Script auch die Routing-Tabellen **mwu** und **icvpn** mit den nötigen statischen Routen::

    #
    # IP routes
    #

    # static mainz routes for rt_table mwu
    /sbin/ip -4 route add 10.37.0.0/18 proto static dev mzBR table mwu
    /sbin/ip -6 route add fd37:b4dc:4b1e::/64 proto static dev mzBR table mwu

    # static wiesbaden routes for rt_table mwu
    /sbin/ip -4 route add 10.56.0.0/18 proto static dev wiBR table mwu
    /sbin/ip -6 route add fd56:b4dc:4b1e::/64 proto static dev wiBR table mwu

    # static blackhole routes for rt_table ffinetexit
    /sbin/ip -4 route add blackhole 0.0.0.0/8 table ffinetexit
    /sbin/ip -4 route add blackhole 10.0.0.0/8 table ffinetexit
    /sbin/ip -4 route add blackhole 100.64.0.0/10 table ffinetexit
    /sbin/ip -4 route add blackhole 127.0.0.0/8 table ffinetexit
    /sbin/ip -4 route add blackhole 169.254.0.0/16 table ffinetexit
    /sbin/ip -4 route add blackhole 172.16.0.0/12 table ffinetexit
    /sbin/ip -4 route add blackhole 192.0.0.0/24 table ffinetexit
    /sbin/ip -4 route add blackhole 192.0.2.0/24 table ffinetexit
    /sbin/ip -4 route add blackhole 192.88.99.0/24 table ffinetexit
    /sbin/ip -4 route add blackhole 192.168.0.0/16 table ffinetexit
    /sbin/ip -4 route add blackhole 198.18.0.0/15 table ffinetexit
    /sbin/ip -4 route add blackhole 198.51.100.0/24 table ffinetexit
    /sbin/ip -4 route add blackhole 203.0.113.0/24 table ffinetexit
    /sbin/ip -4 route add blackhole 224.0.0.0/4 table ffinetexit
    /sbin/ip -4 route add blackhole 240.0.0.0/4 table ffinetexit
    /sbin/ip -4 route add blackhole 255.255.255.255/32 table ffinetexit

    # static route for icvpn transfer-net
    /sbin/ip -4 route add 10.207.0.0/16 proto static dev icVPN table icvpn
    /sbin/ip -6 route add fec0::/96 proto static dev icVPN table icvpn

.. _LARTC: http://lartc.org/howto/
.. _Policy-Routing: http://www.policyrouting.org/PolicyRoutingBook/ONLINE/TOC.html
