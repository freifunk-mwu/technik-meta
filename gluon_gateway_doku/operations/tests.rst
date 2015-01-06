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

:ref:`exitvpn`::

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
für eine Range ``ip a show to 10.37.0.0/16`` bzw. ``ip a show to fd56:b4dc:4b1e::/64``.

Interessant ist in diesem Kontext auch ``ip a show scope global``.


Routing
-------

.. seealso::

    - :ref:`routing_tables`
    - :ref:`self_dns`

Die IP Rules?
::

    ip rule
     0:      from all lookup local
     3700:   from all iif mzBR lookup mz
     5600:   from all iif wiBR lookup wi
     9937:   from 10.8.0.9 lookup mz
     9956:   from 10.8.0.9 lookup wi
     10042:  from all lookup icvpn
     10042:  from all lookup icvpn
     32766:  from all lookup main
     32767:  from all lookup default

    ip -6 rule
     0:      from all lookup local
     3700:   from all iif mzBR lookup mz
     5600:   from all iif wiBR lookup wi
     10042:  from all lookup icvpn
     10042:  from all lookup icvpn
     32766:  from all lookup main

Die Routing-Tables?
::

    ip route show table mz
     default via 10.8.0.125 dev exitVPN
     10.37.0.0/18 dev mzBR  scope link
     10.56.0.0/18 dev wiBR  scope link  src 10.56.0.23
    ip -6 route show table mz
     fd37:b4dc:4b1e::/64 dev mzBR  metric 1024
     fd56:b4dc:4b1e::/64 dev wiBR  metric 1024
     fe80::/64 dev mzBR  metric 1024

    ip route show table wi
     default via 10.8.0.125 dev exitVPN
     10.37.0.0/18 dev mzBR  scope link  src 10.37.0.23
     10.56.0.0/18 dev wiBR  scope link
    ip -6 route show table wi
     fd37:b4dc:4b1e::/64 dev mzBR  metric 1024
     fd56:b4dc:4b1e::/64 dev wiBR  metric 1024
     fe80::/64 dev wiBR  metric 1024



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
