.. _alfred:

A.L.F.R.E.D. konfigurieren
==========================

A.L.F.R.E.D. soll auf allen Gateways im Master Modus laufen, A.L.F.R.E.D. daemons im Master Modus replizieren ihre Daten gegenseitig.

Dadurch wird sichergestellt, dass jedes Gateway immer alle A.L.F.R.E.D. Daten hat.

Darüber hinaus muss auf allen Gateways zusätzlich ``batadv-vis`` im Server Modus laufen.

Dies sorgt dafür, dass jedes Gate seine ``batman_adv`` neighbors und local client table über A.L.F.R.E.D. bekannt gibt.

Ubuntu liefert keine A.L.F.R.E.D. Pakete mit, daher verwenden wir selbstgebaute :ref:`pakete`.

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
