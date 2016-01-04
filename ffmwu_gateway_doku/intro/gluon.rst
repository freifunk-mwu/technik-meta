.. _gluon:

Gluon
=====

Gluon entstand in Lübeck aus dem Bedarf heraus, einheitliche OpenWRT-Images für Freifunk-Nutzer herauszugeben, die bereits vorkonfiguriert und mit allen Packages versehen waren.

Da es nicht nur in Lübeck Freifunk gibt, wurden alle Elemente zusammen getragen, modularisiert, und für alle Freifunk-Communities veröffentlicht. *Vielen Dank!*

Es ist ein Aufsatz, der den normalen OpenWRT Source Code hernimmt, patcht, und anhand der Konfiguration alles zurechtrückt um den OpenWRT-Bauprozess für einen Satz unterstützter Router-Modelle zu starten. Dabei können dem OpenWRT noch weitere Pakete untergeschoben werden.

Heraus fallen fertige Images, vorkonfiguriert, nur noch flashen ist nötig. Bonus: Die Images werden noch fein mit elliptischen Kurven signiert, um ein automatisches Updaten zu ermöglichen (Autoupdater ist ein Standard OpenWRT-Paket für Gluon)

Hintergrund zu Gluon
--------------------

Ein bisschen zu den Anfängen von Gluon und viel mehr zum technischen Hintergrund kann man unter folgenden Links erfahren:

* http://luebeck.freifunk.net/wiki/gluon (http://luebeck.freifunk.net/wiki/Firmware)
* http://luebeck.freifunk.net/wiki/fastd-schlusselverwaltung
* http://nilsschneider.net/2012/12/24/openwrt-autoupdater.html
* http://nilsschneider.net/2013/02/17/fastd-tutorial.html

Links & Dokumentation
---------------------

* Gluon Code: https://github.com/freifunk-gluon/gluon
* Gluon Pakete:

  * https://github.com/freifunk-gluon/gluon/tree/master/package
  * https://github.com/freifunk-gluon/packages

* Gluon Konfiguration (site.conf) https://github.com/freifunk-gluon/gluon/tree/master/docs/site-example
* Dokumentation: http://gluon.readthedocs.org/
* Github Wiki: https://github.com/freifunk-gluon/gluon/wiki
* fastd: http://fastd.readthedocs.org/

* Doku aus Hamburg: https://wiki.freifunk.net/Freifunk_Hamburg/Firmware#Gluon

Gluon in MWU
------------

Für Mainz und Wiesbaden nutzen wir den unveränderten Quellcode, frisch aus Lübeck.
Unter den folgenden Links gelangt man zu unserer Gluon Konfiguration sowie unser Packages-Repo.

* site.conf: https://github.com/freifunk-mwu/site-ffmwu
* packages: https://github.com/freifunk-mwu/packages-ffmwu

Gebaut wird mit gluon-builder-ffmwu_.

.. _gluon-builder-ffmwu: http://gluon-builder-doku.readthedocs.org
