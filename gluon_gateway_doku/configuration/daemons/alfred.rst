.. _alfred:

A.L.F.R.E.D.
============

A.L.F.R.E.D. soll auf allen Gateways im Slave Modus laufen, A.L.F.R.E.D. daemons im Slave Modus senden ihre Daten zum Master.
Auf dem Server, der die Freifunk Knotenkarte hostet, läuft der A.L.F.R.E.D. Master.

Darüber hinaus muss auf allen Gateways zusätzlich ``batadv-vis`` im Server Modus laufen.

Dies sorgt dafür, dass jedes Gate seine ``batman_adv`` neighbors und local client table über A.L.F.R.E.D. bekannt gibt.

Ubuntu liefert keine A.L.F.R.E.D. Pakete mit, daher verwenden wir selbstgebaute :ref:`packages`.

In unseren Paketen liefern wir für Mainz und Wiesbaden schon vorgefertigte Beispiele.

Defaults beseitigen::

    service alfred stop
    service batadv-vis stop
    rm /etc/default/alfred
    rm /etc/init/alfred.conf
    rm /etc/init/batadv-vis.conf

Danach kopieren wir die vorgefertigten Beispiele an die richige Stelle::

    cp /usr/share/doc/alfred/examples/alfred-mz.default /etc/default/alfred-mz
    cp /usr/share/doc/alfred/examples/alfred-wi.default /etc/default/alfred-wi
    cp /usr/share/doc/alfred/examples/alfred-mz.upstart /etc/init/alfred-mz.conf
    cp /usr/share/doc/alfred/examples/alfred-wi.upstart /etc/init/alfred-wi.conf
    cp /usr/share/doc/batadv-vis/examples/batadv-vis-mz.upstart /etc/init/batadv-vis-mz.conf
    cp /usr/share/doc/batadv-vis/examples/batadv-vis-wi.upstart /etc/init/batadv-vis-wi.conf

Und anschließend die beiden A.L.F.R.E.D. Instanzen starten::

    service alfred-mz start
    service alfred-wi start

Durch die Abhängigkeiten in den A.L.F.R.E.D. Upstart Scripts wird dafür gesorgt, dass die batadv-vis Instanzen automatisch mitgestartet und gestoppt werden.

Announcements
-------------

Die Gateways sollen auch ein paar Daten über sich selbst via Alfred preisgeben, damit die Kartendaten rund werden.
Wir danken `ffnord`_ an dieser Stelle für die `ffnord-alfred-announce`_ Scripte. Wir haben diese unseren Bedürfnissen angepasst und halten sie
im Branch **mwu** vor: `ffmwu-alfred-announce-mwu`_

Nun clonen wir dieses Repo und wechseln in den **mwu** Branch::

    cd ~/clones
    git clone https://github.com/freifunk-mwu/ffnord-alfred-announce.git
    cd ~/clones/ffnord-alfred-announce && git checkout mwu

Konfiguration
`````````````

Die momentan announcten Daten sind:

* Nodeinfo

  * Hardware

    * model

  * Network

    * ip addresses
    * primary mac
    * mesh interfaces

  * Software

    * autoupdater (branch/state)
    * batman_adv version
    * batman_adv gw_mode
    * fastd version
    * firmware (OS Release)

  * System

    * role
    * site_code

  * Hostname
  * Node ID
  * VPN Status

* Statistics

  * clients
  * gateway (batman has selected)
  * idletime
  * loadavg
  * memory
  * node_id
  * processes
  * traffic
  * uptime


Da wir die announce scripte mit einem normalen Benutzer ausführen, das Announcen an sich aber root Rechte erfordern benötigen wir die folgenden Wrapper-Sripts::

    mkdir ~/bin

    ~/bin/alfred

        #!/bin/sh
        exec sudo /usr/sbin/alfred $*
        EOCAT

    ~/bin/alfred-json

        #!/bin/sh
        exec sudo /usr/bin/alfred-json $*
        EOCAT

    ~/bin/batctl

        #!/bin/sh
        exec sudo /usr/bin/batctl $*
        EOCAT

Nun kann die crontab des Benutzers gefüllt werden (crontab -e). Die Alfred Daten sollten minütlich announced werden::

    * * * * * /home/admin/clones/ffnord-alfred-announce-mz/announce.sh -i mzBR -b mzBAT -f mzVPN -s ffmz -u /var/run/alfred-mz.sock > /dev/null 2>&1
    * * * * * /home/admin/clones/ffnord-alfred-announce-wi/announce.sh -i wiBR -b wiBAT -f wiVPN -s ffwi -u /var/run/alfred-wi.sock > /dev/null 2>&1

.. _ffnord: https://github.com/ffnord
.. _ffnord-alfred-announce: https://github.com/ffnord/ffnord-alfred-announce
.. _ffmwu-alfred-announce-mwu: https://github.com/freifunk-mwu/ffnord-alfred-announce/tree/mwu
