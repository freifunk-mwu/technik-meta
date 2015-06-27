.. _cleanup:

Aufräumarbeiten
===============

.. _self_dns:

Lokaler DNS Resolver
--------------------

Nach dem die Konfiguration von BIND abgeschlossen wird der DNS-Eintrag auf sich selbst gesetzt.

Dies kommt in die inet Section des Internet Interfaces, i.d.R. eth0.

Dadurch wird der Nameserver-Eintrag durch **resolvconf** beim Hochkommen des Interfaces nach ``/etc/resolv.conf`` geschrieben

in die /etc/network/interfaces kommt also folgendes::

    iface eth0 inet static
        [...]
        dns-nameservers 127.0.0.1

Als Fallback DNS Server tragen wir zusätzlich noch 2 weitere freie DNS Resolver ein. Dies erledigen wir über resolvconf. Wir legen die Datei ``/etc/resolvconf/resolv.conf.d/tail`` an und tragen die freien DNS Server ein::

    nameserver 213.73.91.35
    nameserver 85.214.20.141

.. seealso::
    - :ref:`bind`

.. _logging:

Logging
-------

Freifunk steht unter Anderem für Netzneutralität und ist in keinster Weise an irgendwelchen Nutzer-, Meta-, Irgendwasdaten interessiert.
Aus diesem Grund muss den sonst so redseligen Linux Daemonen das Logging abgewöhnt werden.

Hier wird beschrieben, wie man das für die jeweiligen Daemons erreicht.

Rsyslog
```````

Wir legen uns auf eine Log Facility fest, deren Ziel ein schwarzes Loch ist.

* **Black-Hole Log-Facility:** ``local6``

/etc/rsyslog.d/50-default.conf

Die folgende Zeile::

    *.*;auth,authpriv.none              -/var/log/syslog

durch diese ersetzen::

    *.*;auth,authpriv.none;local6.none              -/var/log/syslog

/etc/rsyslog.d/99-freifunk.conf::

    local6.*        /dev/null

Nun kann jeder Daemon, der per default syslog für das Loggen benutzt angewiesen werden, die Log Facility ``local6`` zu nutzen.
Dadurch wird bewirkt, dass jegliche Infos im Nirvana landen.


DHCP Server
```````````

/etc/dhcpd/dhcpd.conf::

    log-facility local6;

fastd
`````

/etc/fastd/xxVPN/fastd.conf::

    log level warn;
    hide ip addresses yes;
    hide mac addresses yes;

BIND
````

/etc/bind/named.conf.logging::

    logging {
        channel null { null; };
        category default { null; };
    };

/etc/bind/named.conf::

    ...
    include "/etc/bind/named.conf.logging";

Apache
``````
ServerTokens in /etc/apache2/conf-available/security.conf weniger aussagekräftig einstellen und ServerSignature auf EMail setzen::

    ServerTokens Prod
    ServerSignature EMail

Um Apache das Mitloggen der Zugriffe abzugewöhnen müssen in jeder aktivierten site die Log Direktiven auskommentiert werden::

    #ErrorLog ${APACHE_LOG_DIR}/error.log
    #CustomLog ${APACHE_LOG_DIR}/access.log combined

Des Weiteren existiert eine Standardkonfiguration, die automatisch für jede site das Logging aktiviert, diese muss unbedingt deaktiviert werden::

    sudo a2disconf other-vhosts-access-log.conf
    sudo apachectl graceful
