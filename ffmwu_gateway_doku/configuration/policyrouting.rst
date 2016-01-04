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

    # Priority 7 - lookup rt_table mwu for all incoming traffic of freifunk related interfaces
    ip -4 rule add from all iif mzBR lookup mwu priority 7
    ip -4 rule add from all iif wiBR lookup mwu priority 7
    ...
    <IPv4 rules für IC-VPN>
    <IPv4 rules für Internet Exit>
    ...
    ip -6 rule add from all iif mzBR lookup mwu priority 7
    ip -6 rule add from all iif wiBR lookup mwu priority 7
    ...
    <IPv6 rules für IC-VPN>
    <IPv6 rules für Internet Exit>
    ...

    # Priortiy 23 - lookup rt_table icvpn for all incoming traffic of freifunk bridges
    ip -4 rule add from all iif mzBR lookup icvpn priority 23
    ip -4 rule add from all iif wiBR lookup icvpn priority 23
    ip -6 rule add from all iif mzBR lookup icvpn priority 23
    ip -6 rule add from all iif wiBR lookup icvpn priority 23

    # Priority 41 - lookup rt_table ffinetexit for all incoming traffic of freifunk bridges
    ip -4 rule add from all iif mzBR lookup ffinetexit priority 41
    ip -4 rule add from all iif wiBR lookup ffinetexit priority 41
    ...
    <IPv4 rules für Internet Exit>
    <IPv6 rules für Internet Exit>
    ...

    # Priority 61 - at this point this is the end of policy routing for freifunk related routes 
    ip -4 rule add from all iif mzBR type unreachable priority 61
    ip -4 rule add from all iif wiBR type unreachable priority 61
    ...
    <IPv4 rules für Internet Exit>
    ...
    ip -4 rule add from all iif icVPN type unreachable priority 61
    ip -4 rule add from all iif eth0 type unreachable priority 61
    ip -6 rule add from all iif mzBR type unreachable priority 61
    ip -6 rule add from all iif wiBR type unreachable priority 61
    ip -6 rule add from all iif icVPN type unreachable priority 61
    ...
    <IPv6 rules für Internet Exit>
    ...
    ip -6 rule add from all iif eth0 type unreachable priority 61

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

    # static route for icvpn transfer-net
    /sbin/ip -4 route add 10.207.0.0/16 proto static dev icVPN table icvpn
    /sbin/ip -6 route add fec0::/96 proto static dev icVPN table icvpn

    # static mainz routes for rt_table mwu
    /sbin/ip -4 route add 10.37.0.0/18 proto static dev mzBR table mwu
    /sbin/ip -6 route add fd37:b4dc:4b1e::/64 proto static dev mzBR table mwu

    # static wiesbaden routes for rt_table mwu
    /sbin/ip -4 route add 10.56.0.0/18 proto static dev wiBR table mwu
    /sbin/ip -6 route add fd56:b4dc:4b1e::/64 proto static dev wiBR table mwu


.. _LARTC: http://lartc.org/howto/
.. _Policy-Routing: http://www.policyrouting.org/PolicyRoutingBook/ONLINE/TOC.html
