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
* apache
* nodejs

  * npm
  * grunt
  * bower

Installation der Abhängigkeiten::

    apt-get install npm apache2
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

Wir clonen das ffmap-backend Repository einmal pro Community und checken auch hier wieder den Branch mwu aus::

    cd ~/clones
    git clone https://github.com/freifunk-mwu/ffmap-backend.git ffmap-backend-mz
    git clone https://github.com/freifunk-mwu/ffmap-backend.git ffmap-backend-wi
    cd ffmap-backend-mz && git checkout mwu
    cd ffmap-backend-wi && git checkout mwu

Der Aufruf von bat2nodes.py in der Datei **mkmap.sh** muss im jeweiligen Verzeichnis der Community entsprechend angepasst werden, hier am Beispiel von Wiesbaden::

    ./bat2nodes.py -a aliases-wi.json -m wiBAT -s /var/run/alfred-wi.sock -f $FIRMWARE -d $DEST

Nun kann die crontab gefüllt werden::

    * * * * * /home/admin/clones/ffmap-backend-mz/mkmap.sh /var/www/mapmz/build 0.1
    * * * * * /home/admin/clones/ffmap-backend-wi/mkmap.sh /var/www/mapwi/build 0.1
    * * * * * /home/admin/clones/ffmap-backend-mz/mkmap-mwu.sh

Die letzte Option hier ist die momentan stabile Firmware Version. Diese muss mit angegeben werden, damit die Karte Knoten mit veralteter Firmware entspechend markieren kann.

Das Script **mkmap-mwu.sh** merged die nodes.json Dateien aller Communities für die gemeinsame Karte. Die Pfade sind aktuell noch hardcodiert::

    /usr/bin/python3 mergenodesjson.py /var/www/mapmz/build/nodes.json /var/www/mapwi/build/nodes.json /var/www/mapmwu/build/nodes.json
    cp /var/www/mapmz/build/nodes/*.png /var/www/mapmwu/build/nodes/
    cp /var/www/mapwi/build/nodes/*.png /var/www/mapmwu/build/nodes/

Sollte es Abweichungen geben, sind diese entsprechend anzupassen.

Nun fehlen noch die vhosts für den Webserver::

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

