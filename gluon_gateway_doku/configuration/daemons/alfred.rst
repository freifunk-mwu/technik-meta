.. _alfred:

A.L.F.R.E.D.
============

A.L.F.R.E.D. soll auf allen Gateways im Master Modus laufen, A.L.F.R.E.D. daemons im Master Modus replizieren ihre Daten gegenseitig.

Dadurch wird sichergestellt, dass jedes Gateway immer alle A.L.F.R.E.D. Daten hat.

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
in unserem Repository im Branch **mwu** vor: `ffmwu-alfred-announce`_

Nun clonen wir pro Community dieses Repo und wechseln in den Branch **mwu**::

    cd ~/clones
    git clone https://github.com/freifunk-mwu/ffnord-alfred-announce.git ffnord-alfred-announce-mz
    git clone https://github.com/freifunk-mwu/ffnord-alfred-announce.git ffnord-alfred-announce-wi
    cd ~/clones/ffnord-alfred-announce-mz && git checkout mwu
    cd ~/clones/ffnord-alfred-announce-wi && git checkout mwu

Konfiguration
`````````````

Nun müssen die Daten, die announced werden sollen noch dem jeweiligen Gateway angepasst werden, hier am Beispiel von Wiesbaden und dem Gateway **Spinat**::

    cd ~/clones/ffnord-alfred-announce-wi
    echo "\"x86\"" > nodeinfo.d/hardware/model
    echo "\"Spinat (WI)\"" > nodeinfo.d/hostname
    echo "8.249702453613281" > nodeinfo.d/location/longitude
    echo "50.0845343600232" > nodeinfo.d/location/latitude

Die momentan announcten Daten sind::


* Nodeinfo

  * Hardware
  * Hostname
  * Geodaten
  * Software

    * batman_adv version
    * fastd version
    * firmware (OS Release)

  * VPN Status

* Statistics

  * idletime
  * loadavg
  * memory
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

Nun kann die crontab gefüllt werden::

    * * * * * /home/admin/clones/ffnord-alfred-announce-mz/announce.sh -i mzBR -b mzBAT -u /var/run/alfred-mz.sock
    * * * * * /home/admin/clones/ffnord-alfred-announce-wi/announce.sh -i wiBR -b wiBAT -u /var/run/alfred-wi.sock


.. _ffnord: https://github.com/ffnord
.. _ffnord-alfred-announce: https://github.com/ffnord/ffnord-alfred-announce
.. _ffmwu-alfred-announce: https://github.com/freifunk-mwu/ffnord-alfred-announce/tree/mwu
