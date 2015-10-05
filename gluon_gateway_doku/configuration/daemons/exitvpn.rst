.. _exitvpn:

ExitVPN
=======

Als erstes entledigt man sich aller alter Konfiguration::

    rm /etc/openvpn/*

Eine angepasste und lauffähige Konfiguration bekommt man meist vom Anbieter.

Darin muss man die Pfade zu den ca, crt, und so Files anpassen (relativ nach absolut)

Weiterhin muss man alle Stellen entfernen, die in das Routing eingreifen.

Dazu nutzen wir unser eigenes Hook-Script. /etc/openvpn/openvpn-updown::

    #!/bin/sh

    # http://manpages.ubuntu.com/manpages/maverick/en/man8/openvpn.8.html#contenttoc5

    set -x

    ENVFILE="/tmp/ovpn-env-up"
    echo "$@"  > "$ENVFILE"
    env >> "$ENVFILE"

    manage_vpn_peer_routes() {
            ACTION="$1"
            ip route $ACTION "$route_vpn_gateway" dev "$dev" src "$ifconfig_local" table ffinetexit
    }

    manage_vpn_default_routes() {
            ACTION="$1"
            ip route $ACTION 0.0.0.0/1 via "$route_vpn_gateway" src "$ifconfig_local" table ffinetexit
            ip route $ACTION 128.0.0.0/1 via "$route_vpn_gateway" src "$ifconfig_local" table ffinetexit
    }

    if [ "$script_type" = "up" ]; then
            manage_vpn_peer_routes add
            manage_vpn_default_routes replace
    elif [ "$script_type" = "down" ]; then
            manage_vpn_peer_routes del
    fi

    exit 0

Die Routing-Tabelle **ffinetexit** enthält per default eine unreachable default route. Durch das updown-Script werden die beiden spezifischeren Routen **0.0.0.0/1** und **128.0.0.0/1** über das **exitVPN**-Interface ergänzt, wenn der OpenVPN-Daemon gestartet wird und wieder entfernt wenn dieser gestoppt wird.

Das updown-Script auch in die /etc/openvpn/ANBIETER.conf schreiben!

Firewall
^^^^^^^^
Um das Netz des ExitVPN Anbieters via NAT mit unserem zu verbinden muss Masquerading aktiviert werden::
   
  iptables -t nat -A POSTROUTING -s 10.37.0.0/18,10.56.0.0/18 -o exitVPN -j MASQUERADE

um dieses persistent zu machen installiert man noch folgende pakete::

  apt-get install iptables-save iptables-persistent

und speichert dann die iptables config einmalig::

  iptables-save > /etc/iptables/rules.v4


