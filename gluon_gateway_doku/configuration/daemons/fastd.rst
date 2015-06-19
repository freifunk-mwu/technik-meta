.. _fastd:

fastd
=====

Ordnerstruktur::

    /etc/fastd/mzVPN - Config für Mainz
    /etc/fastd/mzVPN/peers - Peers für Mainz
    /etc/fastd/wiVPN - Config für Wiesbaden
    /etc/fastd/wiVPN/peers - Peers für Wiesbaden

Die peers aus dem git clonen::

    git clone https://github.com/freifunk-mwu/peers-ffmz /etc/fastd/mzVPN/peers
    git clone https://github.com/freifunk-mwu/peers-ffwi /etc/fastd/wiVPN/peers

Für die :ref:`scripts` muss der Nutzer darin schreiben dürfen::

    sudo chown -R admin:admin /etc/fastd/*/peers/


Konfiguration
-------------

/etc/fastd/mzVPN/fastd.conf::

    log level warn;
    hide ip addresses yes;
    hide mac addresses yes;

    interface "mzVPN";

    method "salsa2012+umac";    # new method (faster)

    # Bind von v4 and v6 interfaces
    bind $eure_externe_ip4:10037;
    bind [$eure_externe_ip6]:10037;

    include "secret.conf";
    mtu 1406; # 1492 - IPv4/IPv6 Header - fastd Header...

    include peers from "peers";

    status socket "/var/run/fastd-mainz.status";

/etc/fastd/wiVPN/fastd.conf:

Analog zu Mainz (Ports anpassen 10037 -> 10056)

.. _fastd_key:

Schlüsselpaar generieren
------------------------

Das Schlüsselpaar schreibt man sich am besten in ein Tempfile::

     fastd --generate-key >> /etc/fastd/wiVPN/MEINTEMPFILE

dauert manchmal ein bisschen :) keep calm :) 
Wenn euch das zu langsam ist, installiert euch den haveged daemon um mehr Entropie zu generieren.

Das ganze sieht dann so aus:

/etc/fastd/wiVPN/MEINTEMPFILE::

    Secret: "0000..ffff"
    Public: "ffff..0000"

daraus die passenden Files erstellen (auf das ``;``-Zeichen achten, Groß-/Kleinschreibung beachten! ):

/etc/fastd/wiVPN/secret.conf::

    secret "0000..ffff";

/etc/fastd/wiVPN/peers/``GW_Nickname`` (z.B. Lotuswurzel, Spinat, Popcorn)::

    key "ffff...0000";

Achtet auf die korrekten Schreibweisen in der Gw_Nickname und der secret.conf! Groß- und Kleinschreibung sowie das Semikolon am Ende sind wichtig, ansonsten kann es dazu kommen, dass die Bridge, Batman und VPN interfaces nicht hochkommen.
