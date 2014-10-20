.. _bird:

bird für IC-VPN einrichten
==========================

Das IC-VPN beruht darauf, dass zwischen den teilnehmenden communities ein
IP-Routing (layer 3) stattfindet. Dafür wird über :ref:`tinc` ein Transfernetz
zwischen einigen Gates aufgebaut. Über dieses Transfernetz wird dann
ge-route-t.

Die Routing-Einträge auf diesen vielen Routern (Gates) werden nicht
manuell, sondern per `BGP`_ gepflegt (die `ASN`_ für Wiesbaden und Mainz sind 65036 und 65037).
Es gibt zwei etablierte BGP-Implmentationen:
quagga_ und bird_; wir haben uns für letztere entschieden. Auch hier folgen
wir grob der zentralen `Dokumentation`_ und es sei auf das im Aufbau befindliche
`IC-VPN-Meta-repository`_ für die Metainformationen sowie auf das `IC-VPN-scripts-repository`_ für die Erzeugung der bgp peers sowie DNS Config verwiesen.

bird config
^^^^^^^^^^^

bird wird für IPv4 und IPv6 gesondert konfiguriert, wobei sich die config files allerdings sehr
ähneln. Da die Einträge für die Nachbarrouter im IC-VPN (*peers*) in Kürze halbautomatisch
gepflegt werden sollen und die bird-Konfiguration das Einbinden von config files in config
files erlaubt, werden diese schon jetzt ausgelagert. Damit ergibt sich diese Dateistruktur::

  /etc/bird/
  /etc/bird/bird.conf
  /etc/bird/ebgp_peers_v4.inc
  /etc/bird/bird6.conf
  /etc/bird/ebgp_peers_v6.inc

Im Großen und Ganzen handelt es sich bei uns um eine recht normale bird-BGP-Konfiguration
(nachdem der Versuch, in bird eine gleichberechtigte config für zwei AS hinzubekommen
gescheitert war). Die Routen zu den anderen communities werden über BGP abgeglichen. Die eigenen
Netze, die ins IC-VPN bekannt zu geben sind, werden über einen "protocol direkt"-Eintrag
bestimmt.




tbc...










.. _BGP: http://de.wikipedia.org/wiki/Border_Gateway_Protocol
.. _ASN: http://wiki.freifunk.net/AS-Nummern
.. _quagga: http://www.nongnu.org/quagga/
.. _bird: http://bird.network.cz/
.. _Dokumentation: http://wiki.freifunk.net/IC-VPN#BGP_Einrichten
.. _IC-VPN-Meta-repository: https://github.com/freifunk/icvpn-meta
.. _IC-VPN-Scripts-repository: https://github.com/freifunk/icvpn-scripts




Kellerabteil1 zum Durchsuchen für's Weiterschreiben::

  define wi_addr_ic = 10.207.0.56;  # lotuswurzel = wiesbaden1
  define wi_addr_wi = 10.56.0.23;
  define mz_addr_mz = 10.37.0.23;
  router id 10.207.0.56;
  #
  table ic; # BGP Peerings 4 wi (ICVPN) and local wimz nets
  table ic_mz; # BGP Peerings 4 mz (ICVPN) - copy of ic
  #
  function is_freifunk_dn42() {
      return (net ~ [
          10.0.0.0/8{12,32},
          172.22.0.0/15+,
          172.31.0.0/16
          ]);
  }
  #
  function is_wimz_self_nets() {
      return (net ~ [10.56.0.0/16+,
                     10.37.0.0/16+]);
  }
  # 
  function is_wi_self_net() {
      return (net ~ [10.56.0.0/16+]);
  }
  # 
  function is_mz_self_net() {
      return (net ~ [10.37.0.0/16+]);
  }
  # 
  # necessary to inform bird about devices
  protocol device {
      scan time 30;
  };
  # learn about directly connected community subnets
  protocol direct wi_subnets {
      interface 10.37.0.0/16;
      interface 10.56.0.0/16;
      table ic;
  };
  # 
  protocol kernel kernel_wi {
      scan time 30;
      import none;
      export filter {
        if is_wimz_self_nets() then
                reject;
          krt_prefsrc = wi_addr_wi;
          accept;
      };
      table ic;
      kernel table 56;
  };
  #
  protocol pipe wi2mz {
      import all;
      export none;
      table ic_mz;
      peer table ic;
  };
  #    
  protocol kernel kernel_mz {
      scan time 30;
      import none;
      export filter {
          if is_wimz_self_nets() then
              reject;
          krt_prefsrc = mz_addr_mz;
          accept;
      };
      table ic_mz;
      kernel table 37;
  };
  #
  # templates for iBGP
    template bgp bgp_ibgp_wi {
      local wi_addr_wi as 65036;
      table ic;
      import all;  # EXPERIMENT !!!!!
      export where source = RTS_BGP;
      direct;
      gateway direct;
  };
  #
  # templates for eBGP
  template bgp ebgp_ic {
      local wi_addr_ic as 65036;
      table ic;
      import where (is_freifunk_dn42() && !is_wimz_self_nets());
      export filter {
          if is_wi_self_net() then {  # own nets
  #            bgp_path.delete(65036);
  #            bgp_path.prepend(65036);
              bgp_path.prepend(65036);
              accept;
          }
          if is_mz_self_net() then {  # foreign mz nets
              bgp_path.delete(65036);
              bgp_path.prepend(65037);
              bgp_path.prepend(65036);
              accept;
          }
          if source = RTS_BGP then {
              accept;
          }
          reject;
      };
    direct;
  };
  #
  # P E E R I N G S
  # iBGP 
  #
  #protocol bgp wiesbaden2 from bgp_ibgp_wi { # hinterschinken ???
  #    neighbor 10.56.0.5 as 65036;
  #};
  #
  # P E E R I N G S
  # eBGP (siehe IPv6)
  #
  protocol bgp Augsburg1 from ebgp_ic {
      neighbor 10.207.0.17 as 65050;
  };


Kellerabteil1 zum Durchsuchen für's Weiterschreiben::

  _addr_ic     = fec0::a:cf:0:38;        # lotuswurzel = wiesbaden1
  define wi_addr_wi     = fd56:b4dc:4b1e::a38:17;
  define mz_addr_mz     = fd37:b4dc:4b1e::a25:17;
  # prefixes repeated in functions -> see there
  define wi_prefix      = fd56:b4dc:4b1e::/48;
  define mz_prefix      = fd37:b4dc:4b1e::/48;
  #
  router id 10.207.0.56;   # traditionally v4-addr as router id
  #
  # routing tables
  table ic;    # BGP Peerings 4 wi (ICVPN) and local wimz nets
  table ic_mz; # BGP Peerings 4 mz (ICVPN) - copy of ic
  #
  # filter to check ulas
  function is_ula() {
      return (net ~ [ fc00::/7{48,64} ]);
  }
  #
  function is_wimz_self_nets() {
      return (net ~ [fd56:b4dc:4b1e::/48+,
                     fd37:b4dc:4b1e::/48+]);
  }
  #
  function is_wi_self_net() {
      return (net ~ [fd56:b4dc:4b1e::/48+]);
  }
  #
  function is_mz_self_net() {
    return (net ~ [fd37:b4dc:4b1e::/48+]);
  }
  #
  # necessary to inform bird about devices
  protocol device {
      scan time 30;
  };
  # learn about directly connected community subnets
  protocol direct wimz_subnets {
      interface fd56:b4dc:4b1e::/48;
      interface fd37:b4dc:4b1e::/48;
      table ic;
  };
  #
  protocol kernel kernel_wi {
      scan time 30;
      import none;
      export filter {
          if is_wimz_self_nets() then
              reject;
          krt_prefsrc = wi_addr_wi;
          accept;
      };
      table ic;
      kernel table 56;
  };
  #
  protocol pipe wi2mz {
      import all;
      export none;
      table ic_mz;
      peer table ic;
  };
  #
  protocol kernel kernel_mz {
      scan time 30;
      import none;
      export filter {
          if is_wimz_self_nets() then
              reject;
          krt_prefsrc = mz_addr_mz;
          accept;
      };
      table ic_mz;
      kernel table 37;
  };
  #
  # template for iBGP
  template bgp ibgp_wi {
      local wi_addr_wi as 65036;
      table ic;
      import all;  # EXPERIMENT !!!!!
      export where source = RTS_BGP;
      direct;
      gateway direct;
  };
  #
  # template for eBGP
  template bgp ebgp_ic {
      local wi_addr_ic as 65036;
      table ic;
      import where (is_ula() && !is_wimz_self_nets());
      export filter {
          if is_wi_self_net() then {  # own nets
  #            bgp_path.delete(65036);
  #            bgp_path.prepend(65036);
              bgp_path.prepend(65036);
              accept;
          }
        if is_mz_self_net() then {  # foreign mz nets
                bgp_path.delete(65036);
              bgp_path.prepend(65037);
            bgp_path.prepend(65036);
              accept;
          }
          if source = RTS_BGP then {
              accept;
          }
          reject;
      };
    direct;
  };
  #
    # P E E R I N G S
  # #### iBGP #####
  #
  #protocol bgp wiesbaden2 from ibgp_wi { # hinterschinken ???
  #    neighbor fd56:b4dc:4b1e::a38:5 as 65036;
  #};
  #
  # P E E R I N G S
  # #### eBGP #####
  #
  # following the pattern, a load of 'em:
  # protocol bgp <PeerName> from ebgp_ic {
    #    neighbor <PeerAddrV6> as <PeerAS>;
  #};
  include "ebgp_peers_v6.inc";
  #
  protocol bgp Augsburg1 from ebgp_ic {
      neighbor fec0::a:cf:0:a as 65050;
  };

