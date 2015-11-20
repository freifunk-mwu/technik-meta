.. _tests:

Funktionstests
==============

Dienste
-------

Laufen alle Dienste?

:ref:`bind`::

    service bind9 status
     * bind9 is running

:ref:`dhcp`::

    service isc-dhcp-server status
     * isc-dhcp-server start/running, process 1337

:ref:`fastd`::

    service fastd status
     * fastd 'mzVPN' is running
     * fastd 'wiVPN' is running

:ref:`vpnprovider`::

    service openvpn status
     * VPN 'mullvad' is running

:ref:`bird`::

    service bird status
     bird start/running, process 1337

    service bird6 status
     bird6 start/running, process 1337

:ref:`radvd`::

    service radvd status
     * radvd is running

:ref:`ntp`::

    service ntp status
     * NTP server is running


Interfaces
----------

Sind die :ref:`interfaces` in Ordnung?

Ein ``ip a`` sollte folgende Interfaces auflisten:

    * lo
    * eth0
    * mzBR
    * wiBR
    * mzVPN
    * wiVPN
    * mzBAT
    * wiBAT
    * exitVPN
    * icVPN

Für ein spezifisches Interface nutzt man ``ip a show dev wiBR``,
Für eine Range ``ip a show to 10.37.0.0/16`` bzw. ``ip a show to fd56:b4dc:4b1e::/64``.

Interessant ist in diesem Kontext ist auch ``ip a show scope global``.


Routing
-------

.. seealso::

    - :ref:`routing_tables`
    - :ref:`policyrouting`
    - :ref:`self_dns`

Die IP Rules?
::

    ip rule
     0:      from all lookup local
     7:      from all iif mzBR lookup mwu 
     7:      from all iif wiBR lookup mwu 
     7:      from all iif icVPN lookup mwu 
     7:      from all iif exitVPN lookup mwu 
     23:     from all iif mzBR lookup icvpn 
     23:     from all iif wiBR lookup icvpn 
     41:     from all iif mzBR lookup ffinetexit 
     41:     from all iif wiBR lookup ffinetexit 
     61:     from all iif mzBR unreachable
     61:     from all iif wiBR unreachable
     61:     from all iif exitVPN unreachable
     61:     from all iif icVPN unreachable
     61:     from all iif eth0 unreachable
     107:    from all lookup mwu 
     107:    from all lookup icvpn
     32766:  from all lookup main
     32767:  from all lookup default

    ip -6 rule
     0:      from all lookup local
     7:      from all iif mzBR lookup mwu 
     7:      from all iif wiBR lookup mwu 
     7:      from all iif icVPN lookup mwu 
     7:      from all iif exitVPN lookup mwu 
     23:     from all iif mzBR lookup icvpn 
     23:     from all iif wiBR lookup icvpn 
     41:     from all iif mzBR lookup ffinetexit 
     41:     from all iif wiBR lookup ffinetexit 
     61:     from all iif mzBR unreachable
     61:     from all iif wiBR unreachable
     61:     from all iif exitVPN unreachable
     61:     from all iif icVPN unreachable
     61:     from all iif eth0 unreachable
     107:    from all lookup mwu 
     107:    from all lookup icvpn 
     32766:  from all lookup main

Die Routing-Tables?
::

    ip route show table mwu
     10.37.0.0/18 dev mzBR  proto static  scope link
     10.56.0.0/18 dev wiBR  proto static  scope link
    ip -6 route show table mwu
     fd37:b4dc:4b1e::/64 dev mzBR  proto static  metric 1024
     fd56:b4dc:4b1e::/64 dev wiBR  proto static  metric 1024

    ip route show table ffinetexit
     0.0.0.0/1 via 10.3.18.136 dev exitVPN  src 10.3.18.136 
     unreachable default 
     10.3.18.136 dev exitVPN  scope link  src 10.3.18.136 
     128.0.0.0/1 via 10.3.18.136 dev exitVPN  src 10.3.18.136
    ip -6 route show table ffinetexit
     unreachable default dev lo  metric 1024  error -101

    ip route show table icvpn
     10.0.0.0/24 via 10.207.0.59 dev icVPN  proto bird  src 10.207.0.56
     10.0.1.0/24 via 10.207.0.79 dev icVPN  proto bird  src 10.207.0.56
     10.5.0.0/16 via 10.207.0.114 dev icVPN  proto bird  src 10.207.0.56
     10.7.0.0/16 via 10.207.0.11 dev icVPN  proto bird  src 10.207.0.56
     10.8.0.0/16 via 10.207.0.36 dev icVPN  proto bird  src 10.207.0.56
     10.11.0.0/18 via 10.207.0.17 dev icVPN  proto bird  src 10.207.0.56
     ... u.v.m. ...
    ip -6 route show table icvpn
     2001:67c:2d50::/48 via fec0::a:cf:0:82 dev icVPN  proto bird  src fec0::a:cf:0:38  metric 1024
     2001:bf7:20::/48 via fec0::a:cf:0:ba dev icVPN  proto bird  src fec0::a:cf:0:38  metric 1024
     2001:bf7:380::/64 via fec0::a:cf:0:28 dev icVPN  proto bird  src fec0::a:cf:0:38  metric 1024
     2001:bf7:380::/44 via fec0::a:cf:0:28 dev icVPN  proto bird  src fec0::a:cf:0:38  metric 1024
     2001:bf7:540::/44 via fec0::a:cf:1:c4 dev icVPN  proto bird  src fec0::a:cf:0:38  metric 1024
     2001:bf7:550::/44 via fec0::a:cf:1:c4 dev icVPN  proto bird  src fec0::a:cf:0:38  metric 1024
     ... u.v.m. ...

B.A.T.M.A.N.
------------

.. seealso::

    - :ref:`packages`
    - :ref:`fastd`

Die momentan genutzte B.A.T.M.A.N.-Version ermittelt man mit::

    modinfo -F version /lib/modules/$(uname -r)/updates/dkms/batman-adv.ko
     2014.3.0

bzw mit::

    batctl -v
     batctl 2014.3.0 [batman-adv: 2014.3.0]

Gateway Status überprüfen::

    batctl -m mzBAT gw
     server (announced bw: 96.0/96.0 MBit)
    batctl -m wiBAT gw
     server (announced bw: 96.0/96.0 MBit)

Schauen, was die Kollegen so treiben::

    batctl -m wiBAT gwl
     Gateway          (#/255)          Nexthop [outgoingIF]: advertised uplink bandwidth ... [B.A.T.M.A.N. adv 2014.3.0, MainIF/MAC: wiVPN/02:00:0a:38:00:17 (wiBAT)]
     02:00:0a:38:00:05 (255) 02:00:0a:38:00:05 [     wiVPN]: 96.0/96.0 MBit
     02:00:0a:38:00:07 (255) 02:00:0a:38:00:07 [     wiVPN]: 96.0/96.0 MBit

A.L.F.R.E.D.
------------

.. seealso::

    - :ref:`packages`
    - :ref:`alfred`

Wie geht's Alfred?
::

    service alfred-wi status
     alfred-wi start/running, process 1337

    service alfred-mz status
     alfred-mz start/running, process 1337

Sind Nodes online, die ``gluon-announce`` installiert und am laufen haben, sollte man json/gzip Daten erhalten::

    alfred -r 158 -u /var/run/alfred-wi.sock
     [...]
     { "xx:xx:xx:xx:xx:xx", "\xxx\xxx [...] \xxx\xxx" },
     [...]

Ist `alfred-json <https://github.com/tcatm/alfred-json>`_ installiert kann man sich die Daten gleich mit entpacken lassen::

    alfred -r 158 -s /var/run/alfred-wi.sock -z
     [...]
     {
         "xx:xx:xx:xx:xx:xx": {
             "location": {
               "longitude": 0.0,
               "latitude": 0.0
             },
             "network": {
               "mac": "xx:xx:xx:xx:xx:xx",
               [...]
            },
            [...]
         },
         [...]
     }
     [...]

Hier nervt: Zur Angabe des Sockets nutzt ``alfred-json`` den Flag ``-s``, ``alfred`` hingegen ``-u``.
::

    batadv-vis -u /var/run/alfred-mz.sock -f jsondoc
     {
       "source_version" : "2014.4.0",
       "algorithm" : 4,
       "vis" : [
         { "primary" : "xx:xx:xx:xx:xx:xx",
           "neighbors" : [
              { "router" : "xx:xx:xx:xx:xx:xx",
                "neighbor" : "xx:xx:xx:xx:xx:yy",
                "metric" : "1.0" },
              { "router" : "xx:xx:xx:xx:xx:xx",
                "neighbor" : "xx:xx:xx:xx:yy:xx",
                "metric" : "1.1" }
           ],
           "clients" : [
             "xx:xx:xx:xx:xx:xx",
             "xx:xx:xx:xx:yy:yy"
           ]
         },
         [...]
     }
