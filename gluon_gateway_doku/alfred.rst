.. _alfred:

A.L.F.R.E.D. konfigurieren
==========================

A.L.F.R.E.D. soll auf allen Gateways im Master Modus laufen, A.L.F.R.E.D. daemons im Master Modus replizieren ihre Daten gegenseitig.

Dadurch wird sichergestellt, dass jedes Gateway immer alle A.L.F.R.E.D. Daten hat.

Darüber hinaus muss auf alles Gateways zusätzlich ``batadv-vis`` im Server Modus laufen.

Dies sorgt dafür, dass jedes Gate seine ``batman_adv`` neighbors und local client table über A.L.F.R.E.D. bekannt gibt.

Leider sind die von der Distribution mitgelieferten Pakete kaputt, wir müssen uns unsere eigenen :ref:`pakete` installieren.

Altes System V Init Zeugs löschen::

    update-rc.d -f alfred remove
    rm /etc/default/alfred
    rm /etc/init.d/alfred

Danach schreiben wir pro Mesh-Wolke ein Configfile (muss man ggf. neu erstellen).

/etc/default/alfred-mz z.B. für Mainz::

    #
    # /etc/default/alfred-mz
    #

    # Additional command line options
    DAEMON_OPTS=""

    # Interface for A.L.F.R.E.D. to listen on. Has to be specified.
    INTERFACE="mzBR"

    # Specify the batman-adv interface configured on the system (default: bat0).
    # Use 'none' to disable the batman-adv based best server selection.
    BATMANIF="mzBAT"

    # Unix socket to use (default: /var/run/alfred.sock)
    UNIX_SOCKET="/var/run/alfred-mz.sock"

Dazu kommt jeweils ein Upstart Script.

/etc/init/alfred-mz.conf z.B. für Mainz::

    # Starts A.L.F.R.E.D
    #

    description     "A.L.F.R.E.D"
    author          "Tobias Hachmer <tobias@hachmer.de>"

    start on started networking
    stop on stopped networking

    respawn

    script
    . /etc/default/alfred-mz
    exec /usr/sbin/alfred -i ${INTERFACE} -b ${BATMANIF:-bat0} -u ${UNIX_SOCKET} -m ${DAEMON_ARGS}
    end script

Danach noch je ein Upstart Script für batadv-vis, damit Alfred funktioniert.

/etc/init/batadv-vis-wi.conf diesmal für Wiesbaden::

    # Starts batadv-vis
    #

    description     "batadv-vis"
    author          "Tobias Hachmer <tobias@hachmer.de>"

    start on started alfred-wi
    stop on stopped alfred-wi

    respawn
    script
    . /etc/default/alfred-wi
    exec /usr/sbin/batadv-vis -i ${BATMANIF} -u ${UNIX_SOCKET} -s
    end script


