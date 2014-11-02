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
