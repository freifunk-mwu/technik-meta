.. _repositories

Benötigte Repositories
======================

* FFMZWI Repository einbinden (B.A.T.M.A.M Advanced Kernel Modul + batctl)::

    add-apt-repository ppa:ffmzwi/batman-adv-2014

* ALFRED Repository einbinden (momentan noch im launchpad account von tobias)::

    add-apt-repository ppa:kokelnet/a.l.f.r.e.d

* Neoraiders Repository einbinden (für fastd)::

    echo "deb http://repo.universe-factory.net/debian/ sid main" > /etc/apt/sources.list.d/freifunk.list
    apt-key adv --keyserver keyserver.ubuntu.com --recv 16EF3F64CB201D9C

* Repo für aktuelle BIRD Version::

    add-apt-repository ppa:cz.nic-labs/bird

Zum Schluss::

    apt-get update
    apt-get dist-upgrade
