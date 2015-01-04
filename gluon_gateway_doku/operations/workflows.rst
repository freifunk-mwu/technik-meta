.. _workflows:

Workflows
=========

.. _fastd_keys:

fastd-keys
----------

Damit die Nodes eine VPN-Verbindung aufbauen können, muss der *öffentliche* **fastd** *Schlüssel* auf die Gateways gelangen.

Wir verwalten diese Keys in Git-Repositories:

* Mainz: peers-ffmz.git_
* Wiesbaden: peers-ffwi.git_

.. _peers-ffmz.git: https://github.com/freifunk-mwu/peers-ffmz
.. _peers-ffwi.git: https://github.com/freifunk-mwu/peers-ffwi

Bei dem ersten Start der Gluon Node wird ein **fastd** *Schlüsselpaar* erzeugt.

Im Beschreibungstext steht, man solle den erzeugten *öffentlichen Schlüssel* per Mail an unsere Listen schicken. Alternativ lässt sich auch ein Link klicken, der das Email-Programm öffnet und eine neue Mail ausfüllt.

* Mainz: ``keys (at) freifunk-mainz.de``
* Wiesbaden: ``keys (at) freifunk-wiesbaden.de``

Die Empfänger dieser Liste sind am Besten auch Mitglieder des `fastd-keys GitHub Teams`_ - Mitglieder dieser Gruppe haben Schreibrechte auf die *peers* Repositories.

.. _fastd-keys GitHub Teams: https://github.com/orgs/freifunk-mwu/teams/fastd-keys

Die neuen Keys von der Liste werden wie unten gezeigt lokal eingetragen, die Änderungen wie gewohnt gepusht.

.. note:: Damit der Nodebesitzer und die anderen Empfänger der Keys-Listen bescheid wissen, muss nach dem Eintragen der Keys die Mail beantwortet werden. Den *Nodebesitzer* ins **To:**-Feld, ``keys@freifunk-...`` ins **CC:** (Oder einfach auf Reply-All klicken..).

Alle 15 Minuten kommt ein Script vorbei und synchronisiert die neuen Keys von GitHub auf die Gateways.

.. seealso::
    - :ref:`fastd`
    - :ref:`scripts`

.. _fastd_key_format:

fastd-keys Format
-----------------

Ein **fastd**-key ist ein 64-Zeichen langer HEX-String::

    0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef

Damit der **fastd**-daemon den Schlüssel lesen kann, muss noch etwas Syntax drum rum::

    key "0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef";

Bei Start oder Neuladen des **fastd**-daemons wird das Config-File neu eingelesen.
In diesem steht, **fastd** möge zusätzlich noch alle Dateien aus dem ``peers``-Ordner mit einlesen (== *peers-... .git* Repository).

**fastd** nimmt den Dateinamen des keys als peer name. Hat die Node z.B. den Namen *Wurstsalat* ist die Struktur denkbar einfach:

* Dateiname: ``Wurstsalat``::

    key "0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef";

Einige Dateinamen werden zur einfachen Zuordnung mit einem Prefix markiert. In Benutzung/Angedacht sind:

:gw: Für Gateways (``gw_Lotuswurzel``, ``gw_Spinat``).
:srv: Für Dienste-Server (``srv_Aubergine``).
:chaos: Für Nodes im cccmz (``chaos_Haggis``).
:peng: Für Nodes im pengland.


.. _exitvpn_accounts:

exitVPN Accounts
----------------

Anfragen aus dem Freifunknetz in das Internet tunneln wir durch das **exitVPN**, um die Störerhaftung zu umgehen.

Dies hat den Vorteil, dass Anfragen in das Internet anonymisiert werden, Anbieter sehen nur dass die Anfrage aus dem Freifunk-Netz kommt.

Hierbei handelt es sich um **OpenVPN**-Angebote, meist in Schweden oder Niederlande.

Diese werden im Vorraus gezahlt, und müssen von Hand aufgeladen werden.

**Problem** - Dies wird all zu gerne verpeilt!

Im `gateway-configs.git`_ findet sich eine ``exitvpn.yaml``

.. _gateway-configs.git: https://github.com/freifunk-mwu/gateway-configs/

Dort wird pro Gateway hinterlegt, welcher VPN-Account hinterlegt ist, und bis zu welchem Datum dieser bezahlt ist.

Einmal tägtlich kommt ein Script vorbei, und schreibt bei nähern des Datums Mails auf die ``admin@``-Listen, ansonsten wird ein mal pro Woche eine Übersicht verschickt.

.. seealso::
    - :ref:`exitvpn`
    - :ref:`scripts`

