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

    # lookup rt_table mwu for all incoming traffic of freifunk related interfaces
    ip -4 rule add from all iif mzBR lookup mwu priority 7
    ip -4 rule add from all iif wiBR lookup mwu priority 7
    ip -4 rule add from all iif icVPN lookup mwu priority 7
    ip -4 rule add from all iif exitVPN lookup mwu priority 7
    ip -6 rule add from all iif mzBR lookup mwu priority 7
    ip -6 rule add from all iif wiBR lookup mwu priority 7
    ip -6 rule add from all iif icVPN lookup mwu priority 7
    ip -6 rule add from all iif exitVPN lookup mwu priority 7

    # lookup rt_table icvpn for all incoming traffic of freifunk bridges
    ip -4 rule add from all iif mzBR lookup icvpn priority 23
    ip -4 rule add from all iif wiBR lookup icvpn priority 23
    ip -6 rule add from all iif mzBR lookup icvpn priority 23
    ip -6 rule add from all iif wiBR lookup icvpn priority 23

    # lookup rt_table ffinetexit for all incoming traffic of freifunk bridges
    ip -4 rule add from all iif mzBR lookup ffinetexit priority 41
    ip -4 rule add from all iif wiBR lookup ffinetexit priority 41
    ip -6 rule add from all iif mzBR lookup ffinetexit priority 41
    ip -6 rule add from all iif wiBR lookup ffinetexit priority 41

    # at this point this is the end of policy routing for freifunk related routes 
    ip -4 rule add from all iif mzBR type unreachable priority 61
    ip -4 rule add from all iif wiBR type unreachable priority 61
    ip -4 rule add from all iif exitVPN type unreachable priority 61
    ip -4 rule add from all iif icVPN type unreachable priority 61
    ip -4 rule add from all iif eth0 type unreachable priority 61
    ip -6 rule add from all iif mzBR type unreachable priority 61
    ip -6 rule add from all iif wiBR type unreachable priority 61
    ip -6 rule add from all iif exitVPN type unreachable priority 61
    ip -6 rule add from all iif icVPN type unreachable priority 61
    ip -6 rule add from all iif eth0 type unreachable priority 61

    # lookup policies for the gateway host self originating traffic
    ip -4 rule add from all lookup mwu priority 107
    ip -4 rule add from all lookup icvpn priority 107
    ip -6 rule add from all lookup mwu priority 107
    ip -6 rule add from all lookup icvpn priority 107

Wie zu erkennen ist verwenden wir 3 Routing Tabellen:

* icvpn (wird dynamisch über BGP gefüllt)
* mwu (enthält statische Routen der Community-Netze)
* ffinetexit (enthält Routen für den Internet-Verkehr)

Zusätzlich zu den **IP Rules** befüllen wir über das **rc.local**-Script auch die Routing-Tabellen **mwu** und **ffinetexit** mit den nötigen statischen Routen::

    #
    # IP routes
    #

    # static mainz routes for rt_table mwu
    /sbin/ip -4 route add 10.37.0.0/18 proto static dev mzBR table mwu
    /sbin/ip -6 route add fd37:b4dc:4b1e::/64 proto static dev mzBR table mwu

    # static wiesbaden routes for rt_table mwu
    /sbin/ip -4 route add 10.56.0.0/18 proto static dev wiBR table mwu
    /sbin/ip -6 route add fd56:b4dc:4b1e::/64 proto static dev wiBR table mwu

    # unreachable routes for rt_table ffinetexit
    /sbin/ip -4 route add unreachable default table ffinetexit
    /sbin/ip -6 route add unreachable default table ffinetexit


.. _LARTC: http://lartc.org/howto/
.. _Policy-Routing: http://www.policyrouting.org/PolicyRoutingBook/ONLINE/TOC.html
