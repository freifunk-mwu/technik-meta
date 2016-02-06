.. _scripts:

Backend Scripte
===============

Um sich das Leben einfacher zu machen, werden immer wiederkehrende Probleme mittels Scripte gelöst.

Diese werden in einem Repository gesammelt und nutzen selbst Photon_ als Backend.

:Repository: `freifunk-mwu/backend-scripts <https://github.com/freifunk-mwu/backend-scripts>`_
:Backend: `Photon <https://github.com/spookey/photon>`_

Installation
------------

Photon liegt im neuesten Release im `Python Package Index <https://pypi.python.org/pypi/photon_core/>`_ und wird wiefolgt installiert::

    sudo pip3 install -U photon_core --pre

Die eigentlichen Backend Scripte werden normal geklont::

    mkdir -p ~/clones/backend-scripts
    git clone https://github.com/freifunk-mwu/backend-scripts.git ~/clones/backend-scripts

Einrichten
----------

Um den Backend Scripten (Schreib-) Zugriff auf die benötigten Repos unter GitHub zu gewähren müssen diese erst eingerichtet werden. Dazu dient das Script :file:`bootstrap_git_all.py`::

    python3 ~/clones/backend-scripts/bootstrap_git_all.py

Dieses erzeugt ein SSH-Schlüsselpaar und legt es unter :file:`~/.ssh/%hostname%_rsa` ab sofern dieses noch nicht existiert. Die :file:`~/.ssh/config` wird um einen Eintrag erweitert (sofern dieser noch nicht existiert), so dass ein Zugriff auf GitHub mittels frisch erzeugtem Schlüssel möglich ist.

Momentan: ``github_mwu``

Den Öffentlichen Teil des Schlüssels bekommt ein extra erstellter GitHub Account hinterlegt, der duch eine Gruppen Zugriff auf die entsprechenden Repos erhält.

Für uns ist dies der Nutzer `freifunkmwu <https://github.com/freifunkmwu>`_ der innerhalb der Gruppe `machines <https://github.com/orgs/freifunk-mwu/teams/machines>`_ verweilt.

Ob alles korrekt funktioniert lässt sich durch einen einfachen Aufruf zeigen::

    ssh github_mwu

Die Ausgabe sollte folgendes enthalten::

    Hi freifunkmwu! You've successfully authenticated, but GitHub does not provide shell access.
    Connection to github.com closed.

Die restlichen Scripte sind in der Readme des Repos beschrieben.

.. _cron:

Cronjobs
---------

Damit verschiedene Datenquellen atuell bleiben, sind eine ganze Reihe von Cronjobs notwendig. Die outputs werden im Fehlerfall auch direkt auf die admin@ liste geleitet.
Zunächst ruft man seinen Crontab editor auf::

 crontab -e

Da ein Teil der neu installieren binaries unter /home/admin/ liegen, müssen wir den Pfad setzen::

    PATH=/home/admin/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

Jede Minute annouced das gateway per Alfred seine Daten::

    # alfred announce
    * * * * * $HOME/clones/ffnord-alfred-announce/announce.sh -i mzBR -b mzBAT -f mzVPN -u /var/run/alfred-mz.sock -s ffmz > /dev/null 2>&1
    * * * * * $HOME/clones/ffnord-alfred-announce/announce.sh -i wiBR -b wiBAT -f wiVPN -u /var/run/alfred-wi.sock -s ffwi > /dev/null 2>&1

Die aktuellste Firmware wird dreimal pro Tag vom Buildserver geholt::

    # firmware sync
    23 */8 * * * /usr/bin/rsync -avh --delete rsync://pudding.freifunk-mwu.de:2873/firmware /var/www/html/firmware > $HOME/.cronlog/firmware_rsync.log 2>&1

Die Backend Scripte werden ebenfalls aufgerufen::

    # backend scripts
    */5 * * * * /usr/bin/python3 $HOME/clones/backend-scripts/draw_traffic_all.py > $HOME/.cronlog/draw_traffic.log
    */5 * * * * /usr/bin/python3 $HOME/clones/backend-scripts/check_bind_gw.py > $HOME/.cronlog/check_bind.log
    */5 * * * * /usr/bin/python3 $HOME/clones/backend-scripts/check_exitvpn_gw.py > $HOME/.cronlog/check_exitvpn.log
    */5 * * * * /usr/bin/python3 $HOME/clones/backend-scripts/check_radvd_gw.py > $HOME/.cronlog/check_radvd.log
    23 * * * * /usr/bin/python3 $HOME/clones/backend-scripts/gen_website_all.py > $HOME/.cronlog/gen_website.log
    */15 * * * * /usr/bin/python3 $HOME/clones/backend-scripts/sync_meshkeys_gw.py > $HOME/.cronlog/sync_meshkeys.log
    23 5,23 * * * /usr/bin/python3 $HOME/clones/backend-scripts/snapshot_configs_all.py > $HOME/.cronlog/snapshot_configs.log
    42 19 * * 2 /usr/local/bin/photon-dangerous-selfupgrade.py --sudo --repos $HOME/clones/backend-scripts

Die Logs der Cronjobs werden im admin Homeverzeichnis unter .cronlog (versteckt) abgelegt.
Damit das funktioniert, müssen wir es anlegen::

    mkdir  $HOME/.cronlog

Als letztes müssen noch die ICVPN Daten aktuell gehalten werden::

    # icvpn prototypes
    0 3 * * U /usr/bin/python3 $HOME/clones/backend-scripts/update_tinc_conf_gw.py > $HOME/.cronlog/update_tinc_conf.log
    0 4 * * U /usr/bin/python3 $HOME/clones/backend-scripts/update_bird_conf_gw.py > $HOME/.cronlog/update_bird_conf.log
    0 5 * * U /usr/bin/python3 $HOME/clones/backend-scripts/update_bind_conf_gw.py > $HOME/.cronlog/update_bind_conf.log

Dabei sollte darauf geachtet werden, dass U so gewählt wird, dass diese sich möglichst nicht mit den Tagen der anderen Gateways überlappen. Bei Änderungen der Anzahl der Gateways können hier Verschiebungen auf allen Gateways nötig werden; U kann (und wird) mehr als einen Tag enthalten, an jedem Tag sollte auf einem Gateway das Update laufen. Sollte sich dort eine Fehlkonfiguration einschleichen, so werden auf diese Weise nicht alle ICVPN Verbindungen gleichzeitig disfunktional.


.. note::

    Um das oben genannte zu erreichen hier eine Übersicht der momentanen Konfiguration.

    Wir müssen verhindern, dass wir uns durch ein fehlerhaftes Update gleichzeitig alle Gateways zersägen, zumindest sollte dies zeitversetzt geschehen.

    Weiterhin sollte der Zugriff auf die Git Repos auch nicht gleichzeitig stattfinden um Merge-Conflicts vorsorglich zu verhindern.

    Nächste Schritte:
        * Nur Scripte nach oben gennanten Kriterien in der Tabelle auflisten
        * Schema ausknobeln, wie man (z.B. anhand der Gateway-Nummer) eindeutige Zeitpunkte festlegen kann:

            - Wochentag, Stunde, Minute
        * Schema auf die Tabelle anwenden
        * Cronjobs auf den Gateways anpassen
        * ???
        * Profit!

+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+
| Script/Gate           | Kaschu                | Lotuswurzel           | Spinat                | Wasserfloh            | Aubergine             | Pudding               | Linse                 |
+=======================+=======================+=======================+=======================+=======================+=======================+=======================+=======================+
| alfred announce       | ``* * * * *``                                                                                                         | x                                             |
+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+
| firmare sync          | ``23 */8 * * *``                                                                              | x                                                                     |
+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+
| draw traffic          | ``*/5 * * * *``                                                                                                       | x                                             |
+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+
| check bind            | ``*/5 * * * *``                                                                                                       | x                                             |
+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+
| check exitvpn         | ``*/5 * * * *``                                                                               | x                                                                     |
+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+
| check radvd           | ``*/5 * * * *``                                                                               | x                                                                     |
+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+
| gen website           | ``23 * * * *``                                                                                                                                | x                     |
+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+
| sync meshkeys         | ``*/15 * * * *``                                                                              | x                                                                     |
+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+
| snapshot configs      | ``23 5,23 * * *``     | ``42 5,23 * * *``     | ``23 5,23 * * *``     | ``23 5,23 * * *``     | ``42 5,23 * * *``     | ``42 5,23 * * *``     | ``42 5,23 * * *``     |
+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+
| selfupgrade           | ``42 19 * * 2``       | ``42 17 * * 2``       | ``42 19 * * 2``       | ``42 19 * * 2``       | ``42 17 * * 2``       | ``42 17 * * 2``       | ``42 17 * * 2``       |
+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+
| update tinc conf      | ``0 3 * * 3,6``       | ``0 3 * * 3,6,7``     | ``0 3 * * 2,5``       | ``0 3 * * 2,5``       | x                                                                     |
+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+
| update bird conf      | ``0 4 * * 3,6``       | ``0 4 * * 3,6,7``     | ``0 4 * * 2,5``       | ``0 4 * * 2,5``       | x                                                                     |
+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+
| update bind conf      | ``0 5 * * 3,6``       | ``0 5 * * 3,6,7``     | ``0 5 * * 2,5``       | ``0 5 * * 2,5``       | ``0 5 * * 0,1,5``     | x                                             |
+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+
| alfred to zone        | x                                                                                             | ``*/15 * * * *``      | x                                             |
+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+
| ffmap-d3              | x                                                                                             | ``* * * * *``         | x                                             |
+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+
| meshviewer            | x                                                                                             | ``* * * * *``         | x                                             |
+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+
| cleanup node states   | x                                                                                             | ``*/15 * * * *``      | x                                             |
+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+
| mirror openwrt repo   | x                                                                                             | ``19 1 * * * *``      | x                                             |
+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+
| nagg exitvpn accouts  | x                                                                                             | ``23 19 * * *``       | ``23 17 * * *``       | x                     |
+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+
| autobuild gluon       | x                                                                                                                     | ``#42 5 * * 0,4``     | x                     |
+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+
