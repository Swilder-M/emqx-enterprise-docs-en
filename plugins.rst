
.. _plugins:

=======
Plugins
=======

EMQ X messaging broker can be extended by plugins. Ultilizing the module registration and hook mechanism, developers can customize the broker with plugins to extended the authentication, access control, data persistence, bridge and management functions.

+---------------------+-------------------------+----------------+---------------------------+
| Plugin              | Config file             | Load by default| Description               |
+=====================+=========================+================+===========================+
| emqx_dashboard      | emqx_dashboard.conf     | Y              | Web dashboard (default)   |
+---------------------+-------------------------+----------------+---------------------------+
| emqx_modules        | emqx_modules.conf       | Y              | Modules plugins           |
+---------------------+-------------------------+----------------+---------------------------+
| emqx_retainer       | emqx_retainer.conf      | Y              | Retained message          |
+---------------------+-------------------------+----------------+---------------------------+
| emqx_recon          | emqx_recon.conf         | Y              | Recon plugin              |
+---------------------+-------------------------+----------------+---------------------------+
| emqx_reloader       | emqx_reloader.conf      | N              | Reloader plugin           |
+---------------------+-------------------------+----------------+---------------------------+
| emqx_web_hook       | emqx_web_hook.conf      | N              | Web Hook plugin           |
+---------------------+-------------------------+----------------+---------------------------+

----------------
Dashboard Plugin
----------------

EMQ X Web Dashboard, loaded by default. URL: http://host:18083, default username/password: admin/public.

On the dashboard, following information can be queried: Status of EMQ X, statistics and metrics of clients, sessions, topics and subscriptions.

.. image:: ./_static/images/dashboard.png

Dashboard Listener
------------------

Config file: 'emqx_dashboard.conf', default port of listener: 18083.

.. code-block:: properties

    ## HTTP Listener
    dashboard.listener.http = 18083
    dashboard.listener.http.acceptors = 2
    dashboard.listener.http.max_clients = 512

    ## HTTPS Listener
    ## dashboard.listener.https = 18084
    ## dashboard.listener.https.acceptors = 2
    ## dashboard.listener.https.max_clients = 512
    ## dashboard.listener.https.handshake_timeout = 15
    ## dashboard.listener.https.certfile = etc/certs/cert.pem
    ## dashboard.listener.https.keyfile = etc/certs/key.pem
    ## dashboard.listener.https.cacertfile = etc/certs/cacert.pem
    ## dashboard.listener.https.verify = verify_peer
    ## dashboard.listener.https.fail_if_no_peer_cert = true

---------------
Retainer Plugin
---------------

Retainer plugin is responsible for the persistence of MQTT retained messages. Config file: 'emqx_retainer.conf'.

.. code-block:: properties

    ## disc: disc_copies, ram: ram_copies
    ## Notice: retainer's storage_type on each node in a cluster must be the same!
    retainer.storage_type = ram

    ## Max number of retained messages
    retainer.max_message_num = 1000000

    ## Max Payload Size of retained message
    retainer.max_payload_size = 64KB

    ## Expiry interval. Never expired if 0
    ## h - hour
    ## m - minute
    ## s - second
    retainer.expiry_interval = 0

---------------
Modules Plugin
---------------

Consists of modules like Presence, Subscription, Rewrite and etc.

Presence module
---------------

Presence module published presence message to $SYS/ when a client connected or disconnected:

.. code-block:: properties

    ## Enable Presence, Values: on | off
    module.presence = on

    module.presence.qos = 1

Subscription Module
-------------------

    Subscription module forces clients to subscribe to some particular topics when connected to the broker:

.. code-block:: properties

    ## Enable Subscription, Values: on | off
    module.subscription = on

    ## Subscribe the Topics automatically when client connected
    module.subscription.1.topic = $client/%c
    ## Qos of the subscription: 0 | 1 | 2
    module.subscription.1.qos = 1

    ## module.subscription.2.topic = $user/%u
    ## module.subscription.2.qos = 1

Rewrite Module
--------------

Rewrite module supports topic rewrite:

.. code-block:: properties

    ## Enable Rewrite, Values: on | off
    module.rewrite = off

    ## {rewrite, Topic, Re, Dest}
    ## module.rewrite.rule.1 = "x/# ^x/y/(.+)$ z/y/$1"
    ## module.rewrite.rule.2 = "y/+/z/# ^y/(.+)/z/(.+)$ y/z/$2"

------------
Recon Plugin
------------

Recon plugin loads the recon library on a running EMQ X. Recon library helps by debugging and optimizing Erlang applications. It supports periodically global garbage collection. This plugin registers 'recon' command to the './bin/emqx_ctl' CLI tool. Config file: 'emqx_recon.conf'.

Setup the interval of global GC
-------------------------------

.. code-block:: properties

    ## Global GC Interval
    ## h - hour
    ## m - minute
    ## s - second
    recon.gc_interval = 5m

Recon Plugin CLI
----------------

.. code-block:: bash

    ./bin/emqx_ctl recon

    recon memory            #recon_alloc:memory/2
    recon allocated         #recon_alloc:memory(allocated_types, current|max)
    recon bin_leak          #recon:bin_leak(100)
    recon node_stats        #recon:node_stats(10, 1000)
    recon remote_load Mod   #recon:remote_load(Mod)

---------------
Reloader Plugin
---------------

Erlang Module Reloader for development. If this plugin is loaded, EMQ X hot-updates the codes automatically.

Setup Reload Interval
---------------------

Config file: 'emqx_reloader.conf':

.. code-block:: properties

    reloader.interval = 60s

    reloader.logfile = reloader.log

Load Reloader Plugin
--------------------

.. code-block:: bash

    ./bin/emqx_ctl plugins load emqx_reloader

Reloader Plugin CLI
-------------------

.. code-block:: bash

    ./bin/emqx_ctl reload

    reload <Module>             # Reload a Module

---------------
Web Hook Plugin
---------------

Web Hook Plugin is responsible for sending mqtt message via http post to the configured Web Server.

Configuration
-------------

Config fiele emqx_web_hook.conf:

.. code-block:: properties

    ## http post web server
    web.hook.api.url = http://127.0.0.1:8080

    ## hook rule
    web.hook.rule.client.connected.1     = {"action": "on_client_connected"}
    web.hook.rule.client.disconnected.1  = {"action": "on_client_disconnected"}
    web.hook.rule.client.subscribe.1     = {"action": "on_client_subscribe"}
    web.hook.rule.client.unsubscribe.1   = {"action": "on_client_unsubscribe"}
    web.hook.rule.session.created.1      = {"action": "on_session_created"}
    web.hook.rule.session.subscribed.1   = {"action": "on_session_subscribed"}
    web.hook.rule.session.unsubscribed.1 = {"action": "on_session_unsubscribed"}
    web.hook.rule.session.terminated.1   = {"action": "on_session_terminated"}
    web.hook.rule.message.publish.1      = {"action": "on_message_publish"}
    web.hook.rule.message.delivered.1    = {"action": "on_message_delivered"}
    web.hook.rule.message.acked.1        = {"action": "on_message_acked"}



Load Web Hook Plugin
---------------------

.. code-block:: bash

    ./bin/emqx_ctl plugins load emqx_web_hook

.. _recon: http://ferd.github.io/recon/

