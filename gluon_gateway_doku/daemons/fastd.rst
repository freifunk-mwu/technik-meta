.. _fastd:

fastd einrichten
================

Ordnerstruktur::

    /etc/fastd/mzVPN - Config für Mainz
    /etc/fastd/mzVPN/peers - Peers für Mainz
    /etc/fastd/wiVPN - Config für Wiesbaden
    /etc/fastd/wiVPN/peers - Peers für Wiesbaden

    sudo chmod -R admin:admin /etc/fastd/*/peers/

Als Benutzer admin ausführen (für FFctl)!

git peers clonen::

    git clone https://github.com/freifunk-mwu/peers-ffmz /etc/fastd/mzVPN/peers
    git clone https://github.com/freifunk-mwu/peers-ffwi /etc/fastd/wiVPN/peers


Für Mainz
---------

/etc/fastd/mzVPN/fastd.conf::

    log to syslog level warn;
    interface "mzVPN";
    method "salsa2012+gmac";    # new method, between gateways for the moment (faster)

    # Bind von v4 and v6 interfaces
    bind 1.2.3.4:10037;
    bind [1234:1234:1234:1234::1]:10037;

    include "secret.conf";
    mtu 1406; # 1492 - IPv4/IPv6 Header - fastd Header...

    include peers from "peers";

.. _fastd_key:

Schlüsselpaar generieren
------------------------

Das Schlüsselpaar schreibt man sich in ein Tempfile::

     fastd --generate-key > /etc/fastd/mzVPN/MEINTEMPFILE

dauert manchmal ein bisschen :) keep calm :)
Das ganze sieht dann so aus:

/etc/fastd/mzVPN/MEINTEMPFILE::

    Secret: "0000..ffff"
    Public: "ffff..0000"

daraus die passenden Files erstellen (auf das ``;``-Zeichen achten!):

/etc/fastd/mzVPN/secret.conf::

    secret "0000..ffff";

/etc/fastd/mzVPN/peers/``GW_Nickname`` (z.B. Lotuswurzel, Spinat, Popcorn)::

    key "ffff...0000";

Für Wiesbaden
-------------

/etc/fastd/wiVPN/fastd.conf:

Analog zu Mainz (Ports anpassen 10037 -> 10056)

/etc/fastd/wiVPN/secret.conf,
/etc/fastd/wiVPN/peers/``GW_Nickname``:

Danach wie oben Schlüssel erzeugen
