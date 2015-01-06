.. _nodemap:

Knoten Karte
============

Der Betrieb der Knoten Karte ist in zwei Bereiche unterteilt:

* Karten Frontend

  * `Original Frontend`_
  * `MWU Frontend`_

* Karten Backend

  * `Original Backend`_
  * `MWU Backend`_

Das Karten Backend liest die Knotendaten (Namen, Clientcount, IP Adressen, Mesh Links, Geodaten, etc.) sowie die Meshverbindungen aus A.L.F.R.E.D.
und bereitet diese im JSON-Format auf. Das Ergebnis wird in der Datei **nodes.json** gespeichert und dem Karten Frontend bereitgestellt.
Dieses zeigt all diese Informationen entweder in einem D3-Graphen, in einer Geokarte oder in einer Liste augenfreundlich an.

.. _Original Frontend: https://github.com/ffnord/ffmap-d3
.. _Original Backend: https://github.com/ffnord/ffmap-backend
.. _MWU Frontend: https://github.com/freifunk-mwu/ffmap-d3/tree/mwu
.. _MWU Backend: https://github.com/freifunk-mwu/ffmap-backend/tree/mwu

Frontend
--------

Voraussetzungen
```````````````
* nodejs

  * npm
  * grunt
  * bower

Installation der Abhängigkeiten::

    apt-get install npm
    sudo npm install -g grunt-cli bower


Konfiguration der Karten
````````````````````````

Wir legen für jede Community eine Karte an. Darüber hinaus wird eine Karte angelegt, die alle Communities zusammenführt::

    cd /var/www
    sudo git clone https://github.com/freifunk-mwu/ffmap-d3.git mapmz
    sudo git clone https://github.com/freifunk-mwu/ffmap-d3.git mapwi
    sudo git clone https://github.com/freifunk-mwu/ffmap-d3.git mapmwu
    sudo chown -R admin:admin map*

Freifunk MWU hat das Karten Frontend angepasst. Dafür muss der Branch mwu ausgecheckt werden::

    cd /var/www/mapmz && git checkout mwu
    cd /var/www/mapwi && git checkout mwu
    cd /var/www/mapmwu && git checkout mwu

Nun muss in jedes Kartenverzeichnis gewechselt werden, um die nodejs Abhängigkeiten zu installieren::

    cd /var/www/mapmz && npm install
    cd /var/www/mapwi && npm install
    cd /var/www/mapmwu && npm install

Der Community Name sowie deren Einstiegshomepage wird in der Datei **config.json** konfiguriert, hier am Beispiel von Wiesbaden::

    {
      "cityname": "Wiesbaden",
      "sitename": "wiesbaden.freifunk.net",
      "url": "/"
    }

Mit Hilfe von grunt werden alle Abhängigkeiten installiert sowie der Karten-Code in das Unterverzeichnis build kompiliert::

    grunt -f

Hier tauchen leider noch ein paar **no-undef** Fehler aufgrund der Statistik Seite auf, die ich nicht beheben konnte.


Backend
-------

Voraussetzungen
```````````````
* Python3

  * jq

* alfred-json
* autoconf
* flex
* bison
* build-essential

Installation der Abhängigkeiten::

    sudo apt-get install alfred-json autoconf flex bison build-essential
    sudo pip3 install jq


Für das Backend muss der Benutzer **admin** ein paar Binaries mit Root Rechten ausführen, deshalb legen wir ein bin Verzeichnis an und legen dort Wrapper-Scripte hinein::

    mkdir ~/bin

    ~/bin/batadv-vis

        #!/bin/sh
        exec sudo /usr/sbin/batadv-vis $*
        EOCAT

    ~/bin/batctl

        #!/bin/sh
        exec sudo /usr/sbin/batctl $*
        EOCAT

    ~/bin/alfred-json

        #!/bin/sh
        exec sudo /usr/bin/alfred-json $*
        EOCAT

Konfiguration
`````````````
Wir haben das Karten Backend so angepasst, sodass man mit einem Karten Backend mehrere Frontends befüllen kann.
Wir clonen das ffmap-backend Repository einmal und checken auch hier wieder den Branch mwu aus::

    cd ~/clones
    git clone https://github.com/freifunk-mwu/ffmap-backend.git
    cd ffmap-backend && git checkout mwu

Für jede Community und auch für die übergreifende Kartendarstellung muss es ein eigenes Script **mkmap-xx.sh** geben.
Die Aufrufe für Mainz, Wiesbaden und MWU sind schon vorgefertigt, es müssen lediglich die korrekten Zeilen auskommentiert werden::

    cp mkmap.sh mkmap-mz.sh mkmap-wi.sh mkmap-mwu.sh

Der Aufruf für Mainz sieht dann in der mkmap-mz.sh so aus::

    ./bat2nodes.py -c mainz -a aliases-mz.json -m mzBAT -s /var/run/alfred-mz.sock -f $FIRMWARE -d $DEST

Und für die übergreifende Karte so::

    ./bat2nodes.py -c mwu -a aliases-mz.json -a aliases-wi.json -m mzBAT -m wiBAT -s /var/run/alfred-mz.sock -s /var/run/alfred-wi.sock -f $FIRMWARE -d $DEST

Wichtig bei diesem Aufruf ist, dass die Reihenfolge der übergebenden Parameter konsistent ist. Wenn für **-m** an erster Stelle das Mainzer Batman Interface angegeben wird, dann muss für **-s** auch das alfred-socket des Mainzer Meshes an erster Stelle angegeben werden.

An dieser Stelle eine kurze Erläuterung der Parameter:

   =========== =================================================================================================================
    Parameter   Erläuterung
   =========== =================================================================================================================
   -c          Ein Kurzname um die verschiedenen Communities im selben Backend zu unterscheiden (nodedb, state.json, nodes.json)
   -a          Aliases File(s), in denen statische Parameter für Gateways definiert werden können.
   -m          Batman Interface(s)
   -s          Alfred Unix Socket(s)
   -f          Aktuelle Firmware Version (wird über den Aufruf des **mkmap-xx.sh** Scriptes übergeben)
   -d          Pfad zum webroot des jeweiligen Karten Frontends (wird über den Aufruf des **mkmap-xx.sh** Scriptes übergeben)
   =========== =================================================================================================================

Nun kann die crontab gefüllt werden::

    * * * * * /home/admin/clones/ffmap-backend/mkmap-mz.sh /var/www/mapmz/build 0.1
    * * * * * /home/admin/clones/ffmap-backend/mkmap-wi.sh /var/www/mapwi/build 0.1
    * * * * * /home/admin/clones/ffmap-backend/mkmap-mwu.sh /var/www/mapmwu/build 0.1

Sollte es Abweichungen geben, sind diese entsprechend anzupassen.

Nun fehlen noch die vhosts für :ref:`apache`::

/etc/apache2/sites-available/ffmap-mz.conf
/etc/apache2/sites-available/ffmap-wi.conf
/etc/apache2/sites-available/ffmap-mwu.conf

Hier am Beispiel der Freifunk MWU Karte, die die Knoten aller Communities zusammen anzeigt. Für die anderen Communities sind die Pfade entsprechend anzupassen::

    <VirtualHost *:80>
            ServerName map.freifunk-mwu.de

            ServerAdmin admin@freifunk-mainz.de
            DocumentRoot /var/www/mapmwu/build

            <Directory /var/www/mapmwu/build>
                    Options +Indexes +FollowSymlinks +MultiViews
                    DirectoryIndex geomap.html graph.html list.html
                    AllowOverride None
                    Order allow,deny
                    allow from all
            </Directory>
    </VirtualHost>
    # vim: syntax=apache ts=4 sw=4 sts=4 sr noet

Anschließend müssen diese vhosts noch aktiviert und der Webserver neu geladen werden::

    a2ensite ffmap-mz.conf
    a2ensite ffmap-wi.conf
    a2ensite ffmap-mwu.conf
    apachectl -t
    apachectl graceful

