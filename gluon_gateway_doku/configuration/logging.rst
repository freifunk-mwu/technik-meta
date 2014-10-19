.. _logging:

Logging
===================

Die Freifunk Gateway dürfen keine relevanten Log-Files anlegen. Hier wird beschrieben, wie man das für die jeweiligen Daemons erreicht.

Rsyslog
-------

Wir legen uns auf eine Log Facility fest, deren Ziel ein schwarzes Loch ist.

* **Black-Hole Log-Facility:** ``local6``

/etc/rsyslog.d/50-default.conf

Die Zeile::

    *.*;auth,authpriv.none              -/var/log/syslog

muss durch::

    *.*;auth,authpriv.none;local6.none              -/var/log/syslog

ersetzt werden.

/etc/rsyslog.d/99-freifunk.conf::

    local6.*        /dev/null

DHCP Server
-----------

/etc/dhcpd/dhcpd.conf::

    log-facility local6;

fastd
-----

/etc/fastd/xxVPN/fastd.conf::

    log level fatal;
    hide ip addresses yes;
    hide mac addresses yes;

BIND
----

/etc/bind/named.conf.logging::

    logging {
        channel null { null; };
        category default { null; };
    };

/etc/bind/named.conf::

    ...
    include "/etc/bind/named.conf.logging";

Apache
------

Um Apache das Mitloggen der Zugriffe abzugewöhnen genügt es die Log Direktiven auszukommentieren::

    #ErrorLog ${APACHE_LOG_DIR}/error.log
    #CustomLog ${APACHE_LOG_DIR}/access.log combined
