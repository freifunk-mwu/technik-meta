.. _firewall:

Firewall
========

Die Freifunk Gateways sollen in erster Linie Netzwerkpakete zwischen Freifunk-Netzen routen.
Deshalb verstehen wir die Gateways primär als Router. Mittels Policy Routing und Blackhole Routes stellen wir sicher,
dass Netzwerk-Pakete keinen falschen Weg laufen können. Dennoch sind ein paar wenige Firewall-Regeln erforderlich,
um z.B. invalide Pakete so früh wie möglich zu verwerfen::

    iptables -A INPUT -m conntrack --ctstate INVALID -j DROP
    iptables -A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
    iptables -A FORWARD -m conntrack --ctstate INVALID -j DROP
    iptables -A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
    iptables -A OUTPUT -m conntrack --ctstate INVALID -j DROP
    iptables -A OUTPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT

    iptables-save > /etc/iptables/rules.v4

    ip6tables -A INPUT -m conntrack --ctstate INVALID -j DROP
    ip6tables -A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
    ip6tables -A FORWARD -m conntrack --ctstate INVALID -j DROP
    ip6tables -A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
    ip6tables -A OUTPUT -m conntrack --ctstate INVALID -j DROP
    ip6tables -A OUTPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT

    ip6tables-save > /etc/iptables/rules.v6

