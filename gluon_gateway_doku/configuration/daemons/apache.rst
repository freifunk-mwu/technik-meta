.. _apache:

Apache
======

Wir wollen keine Versionsdetails über den Apache Server preisgeben.

/etc/apache2/conf-available/security.conf::

    ...
    # ServerTokens
    # This directive configures what you return as the Server HTTP response
    # Header. The default is 'Full' which sends information about the OS-Type
    # and compiled in modules.
    # Set to one of:  Full | OS | Minimal | Minor | Major | Prod
    # where Full conveys the most information, and Prod the least.
    #ServerTokens Minimal
    ServerTokens Prod
    #ServerTokens Full

.. _firmware_repo:

Firmware Repositories
---------------------

Jedes Gateway dient als Firmware Repository für die Knoten.

Verzeichnisstruktur:

* /var/www/html/firmware

  * mainz

    * stable
    * beta
    * experimental

  * wiesbaden

    * stable
    * beta
    * experimental

Apache Konfiguration:

/etc/apache2/sites-available/firmware-mainz.conf::

    <VirtualHost *:80>
            ServerName firmware.freifunk-mainz.de
            ServerAlias firmware.ffmz.org

            ServerAdmin webmaster@freifunk-mwu.de
            DocumentRoot /var/www/html/firmware/mainz

            <Directory /var/www/html/firmware/mainz>
                    Options Indexes FollowSymlinks
                    IndexOptions FancyIndexing +FoldersFirst +HTMLTable +NameWidth=*
                    AllowOverride None
                    Order allow,deny
                    allow from all
            </Directory>
    </VirtualHost>
    # vim: syntax=apache ts=4 sw=4 sts=4 sr noet

/etc/apache2/sites-available/firmware-wiesbaden.conf::

    <VirtualHost *:80>
            ServerName firmware.freifunk-wiesbaden.de
            ServerAlias firmware.ffwi.org

            ServerAdmin webmaster@freifunk-mwu.de
            DocumentRoot /var/www/html/firmware/wiesbaden

            <Directory /var/www/html/firmware/wiesbaden>
                    Options Indexes FollowSymlinks
                    IndexOptions FancyIndexing +FoldersFirst +HTMLTable +NameWidth=*
                    AllowOverride None
                    Order allow,deny
                    allow from all
            </Directory>
    </VirtualHost>
    # vim: syntax=apache ts=4 sw=4 sts=4 sr noet

/etc/apache2/sites-available/firmware-mwu.conf::

    <VirtualHost *:80>
            ServerName firmware.freifunk-mwu.de

            ServerAdmin webmaster@freifunk-mwu.de
            DocumentRoot /var/www/html/firmware

            <Directory /var/www/html/firmware>
                    Options Indexes FollowSymlinks
                    IndexOptions FancyIndexing +FoldersFirst +HTMLTable +NameWidth=*
                    AllowOverride None
                    Order allow,deny
                    allow from all
            </Directory>
    </VirtualHost>
    # vim: syntax=apache ts=4 sw=4 sts=4 sr noet

Anschließend die vhosts aktivieren und den apache daemon neuladen::

    a2ensite firmware-mainz.conf
    a2ensite firmware-wiesbaden.conf
    a2ensite firmware-mwu.conf
    apachectl -t
    apachectl graceful
