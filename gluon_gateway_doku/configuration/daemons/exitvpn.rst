.. _exitvpn:

ExitVPN
=======

Als erstes entledigt man sich aller alter Konfiguration::

    rm /etc/openvpn/*

Eine angepasste und lauffähige Konfiguration bekommt man meist vom Anbieter.

Darin muss man die Pfade zu den ca, crt, und so Files anpassen (relativ nach absolut)

Weiterhin muss man alle Stellen entfernen, die in das Routing eingreifen.

Dazu nutzen wir unsere eigenen Hook-Scripte. /etc/openvpn/openvpn-up::

    #!/bin/sh

    # http://manpages.ubuntu.com/manpages/maverick/en/man8/openvpn.8.html#contenttoc5

    # ip rules for maintenance
    ip rule add from $ifconfig_local table mz priority 9937
    ip rule add from $ifconfig_local table wi priority 9956

    # default route into appropriate rt_table
    ip route replace 0.0.0.0/1 via $route_vpn_gateway dev $dev table mz
    ip route replace 128.0.0.0/1 via $route_vpn_gateway dev $dev table mz
    ip route replace 0.0.0.0/1 via $route_vpn_gateway dev $dev table wi
    ip route replace 128.0.0.0/1 via $route_vpn_gateway dev $dev table wi

    exit 0

Und Down-Script: /etc/openvpn/openvpn-down::

    #!/bin/sh

    # http://manpages.ubuntu.com/manpages/maverick/en/man8/openvpn.8.html#contenttoc5

    # ip rules for maintenance
    ip rule del from $ifconfig_local table mz priority 9937
    ip rule del from $ifconfig_local table wi priority 9956

    exit 0


Die up- und down- Scripte auch in die /etc/openvpn/ANBIETER.conf schreiben!
