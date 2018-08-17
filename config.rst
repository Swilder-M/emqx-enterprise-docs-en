
.. _config_guide:

=============
Configuration
=============

------------
Config Files
------------

Linux: If EMQ X is installed through a RPM or DEB package, or through an cloud Image, the config files are located in '/etc/emqx/':

+----------------------------+-----------------------------------------------------+
| Config file                | Description                                         |
+============================+=====================================================+
| /etc/emqx/emqx.conf        | EMQ X server configuration                          |
+----------------------------+-----------------------------------------------------+
| /etc/emqx/acl.conf         | EMQ X default ACL file                              |
+----------------------------+-----------------------------------------------------+
| /etc/emqx/plugins/\*.conf  | EMQ X plugins, persistence and bridge configuration |
+----------------------------+-----------------------------------------------------+

Linux: If EMQ X is installed using binary package, the config files are located in 'etc/':

+----------------------------+----------------------------------------------------+
| Config file                | Description                                        |
+============================+====================================================+
| etc/emqx.conf              | EMQ X server configuration                         |
+----------------------------+----------------------------------------------------+
| etc/acl.conf               | EMQ X default ACL file                             |
+----------------------------+----------------------------------------------------+
| etc/plugins/\*.conf        | EMQ X plugins, persistence and bridge configuration|
+----------------------------+----------------------------------------------------+

---------------------
Environment Variables
---------------------

EMQ X allows setting system parameters by environment variables when it starts up:

+--------------------+-------------------------------------------------+
| EMQX_NODE_NAME     | Erlang node name, e.g. emqx@192.168.0.6         |
+--------------------+-------------------------------------------------+
| EMQX_NODE_COOKIE   | Cookie for distributed erlang node              |
+--------------------+-------------------------------------------------+
| EMQX_MAX_PORTS     | Maximum number of opened sockets                |
+--------------------+-------------------------------------------------+
| EMQX_TCP_PORT      | MQTT/TCP listener port, default: 1883           |
+--------------------+-------------------------------------------------+
| EMQX_SSL_PORT      | MQTT/SSL listener port, default: 8883           |
+--------------------+-------------------------------------------------+
| EMQX_WS_PORT       | MQTT/WebSocket listener port, default: 8083     |
+--------------------+-------------------------------------------------+
| EMQX_WSS_PORT      | MQTT/WebSocket/SSL listener port, default: 8084 |
+--------------------+-------------------------------------------------+

---------------
Node and Cookie
---------------

The node name and cookie of EMQ X should be configured in a cluster setup:

.. code-block:: properties

    ## Node name
    node.name = emqx@127.0.0.1

    ## Cookie for distributed node
    node.cookie = emqx_dist_cookie

.. NOTE::

    Erlang/OTP platform application consists of Erlang nodes(processes). Each node(process) is assigned with a node name for communication between nodes. All the connected nodes share the same cookie to authenticate each other.

--------------------
Erlang VM Parameters
--------------------

Erlang VM parameters. By default 100,000 concurrent connections are allowed:

.. code-block:: properties

    ## SMP support: enable, auto, disable
    node.smp = auto

    ## vm.args: -heart
    ## Heartbeat monitoring of an Erlang runtime system
    ## Value should be 'on' or comment the line
    ## node.heartbeat = on

    ## Enable kernel poll
    node.kernel_poll = on

    ## async thread pool
    node.async_threads = 32

    ## Erlang Process Limit
    node.process_limit = 256000

    ## Sets the maximum number of simultaneously existing ports for this system
    node.max_ports = 256000

    ## Set the distribution buffer busy limit (dist_buf_busy_limit)
    node.dist_buffer_size = 32MB

    ## Max ETS Tables.
    ## Note that mnesia and SSL will create temporary ets tables.
    node.max_ets_tables = 256000

    ## Tweak GC to run more often
    node.fullsweep_after = 1000

    ## Crash dump
    node.crash_dump = {{ platform_log_dir }}/crash.dump

    ## Distributed node ticktime
    node.dist_net_ticktime = 60

    ## Distributed node port range
    node.dist_listen_min = 6369
    node.dist_listen_max = 6369

Description of most important parameters of Erlang VM:

+-------------------------+---------------------------------------------------------------------+
| node.process_limit      | Max Erlang VM processes. A MQTT connection consumes 2 processes.    |
|                         | It should be larger than max_clients * 2.                           |
+-------------------------+---------------------------------------------------------------------+
| node.max_ports          | Max port number of a node. A MQTT connection consumes 1 port.       |
|                         | It should be larger than max_clients.                               |
+-------------------------+---------------------------------------------------------------------+
| node.dist_listen_min    | Min TCP port for nodes internal communication.                      |
|                         | If firewall presents, it should be configured accordingly.          |
+-------------------------+---------------------------------------------------------------------+
| node.dist_listen_max    | Max TCP port for nodes internal communication.                      |
|                         | If firewall presents, it should be configured accordingly.          |
+-------------------------+---------------------------------------------------------------------+

---------------------
Cluster Communication
---------------------

EMQ X adopts Scalable RPC architecture, the data channel and the cluster control channel are separated to improve the cluster’s reliability and performance:

.. code-block:: properties

    ## TCP server port.
    rpc.tcp_server_port = 5369

    ## Default TCP port for outgoing connections
    rpc.tcp_client_port = 5369

    ## Client connect timeout
    rpc.connect_timeout = 5000

    ## Client and Server send timeout
    rpc.send_timeout = 5000

    ## Authentication timeout
    rpc.authentication_timeout = 5000

    ## Default receive timeout for call() functions
    rpc.call_receive_timeout = 15000

    ## Socket keepalive configuration
    rpc.socket_keepalive_idle = 7200

    ## Seconds between probes
    rpc.socket_keepalive_interval = 75

    ## Probes lost to close the connection
    rpc.socket_keepalive_count = 9

-----------------
Log Level & Files
-----------------

Console Log
-----------

.. code-block:: properties

    ## Console log. Enum: off, file, console, both
    log.console = console

    ## Console log level. Enum: debug, info, notice, warning, error, critical, alert, emergency
    log.console.level = error

Error Log
---------

.. code-block:: properties

    ## Error log file
    log.error.file = {{ platform_log_dir }}/error.log

Crash Log
---------

.. code-block:: properties

    ## Enable the crash log. Enum: on, off
    log.crash = on

    log.crash.file = {{ platform_log_dir }}/crash.log

Syslog
-------

.. code-block:: properties

    ## Syslog. Enum: on, off
    log.syslog = on

    ##  syslog level. Enum: debug, info, notice, warning, error, critical, alert, emergency
    log.syslog.level = error

-------------------------
Anonymous Auth & ACL File
-------------------------

By default, EMQ X enables Anonymous Auth, any client can connect to the server:

.. code-block:: properties

    ## Allow Anonymous authentication
    mqtt.allow_anonymous = true

Access Control List (ACL) File
------------------------------

Default ACL is based on 'acl.conf'. If other Auth plugin(s), e.g. MySQL and PostgreSQL Auth, is(are) loaded, this config file is then ignored.

.. code-block:: properties

    ## ACL nomatch
    mqtt.acl_nomatch = allow

    ## Default ACL File
    mqtt.acl_file = etc/acl.conf

Defining ACL rules in 'acl.conf'::

    allow|deny user|IP_Address|ClientID PUBLISH|SUBSCRIBE TOPICS

ACL rules are Erlang Tuples, which are matched one by one:

.. image:: _static/images/6.png

Setting default rules in 'acl.conf':

.. code-block:: erlang

    %% allow user 'dashboard' to subscribe to topic '$SYS/#'
    {allow, {user, "dashboard"}, subscribe, ["$SYS/#"]}.

    %% allow local users to subscribe to all topics
    {allow, {ipaddr, "127.0.0.1"}, pubsub, ["$SYS/#", "#"]}.

    %% Deny all user to subscribe to topic '$SYS#' and '#'
    {deny, all, subscribe, ["$SYS/#", {eq, "#"}]}.

.. NOTE:: default rules allow only local user to subscribe to '$SYS/#' and '#'

After EMQ X receives MQTT clients' PUBLISH or SUBSCRIBE packets, it matches the ACL rules one by one till it hits, and return 'allow' or 'deny'.

Cache of ACL Rule
-----------------

Enable Cache of ACL rule for PUBLISH messages:

.. code-block:: properties

    ## Cache ACL for PUBLISH
    mqtt.cache_acl = true

.. WARNING:: If a client cached too much ACLs, it causes high memory occupancy.

------------------------
MQTT Protocol Parameters
------------------------

Max Length of ClientId
----------------------

.. code-block:: properties

    ## Max ClientId Length Allowed.
    mqtt.max_clientid_len = 1024

Max Length of MQTT Packet
-------------------------

.. code-block:: properties

    ## Max Packet Size Allowed, 64K by default.
    mqtt.max_packet_size = 64KB

MQTT Client Idle Timeout
------------------------

Max time interval from Socket connection to arrival of CONNECT packet:

.. code-block:: properties

    ## Client Idle Timeout (Second)
    mqtt.client.idle_timeout = 30

Force GC Client Connection
--------------------------

This parameter is used to optimize the CPU / memory occupancy of MQTT connection. When certain amount of messages are transferred, the connection is forced to GC:

.. code-block:: properties

    ## Force GC: integer. Value 0 disabled the Force GC.
    mqtt.conn.force_gc_count = 100

Enable Per Client Statistics
----------------------------

Enable per client stats:

.. code-block:: properties

    ## Enable client Stats: on | off
    mqtt.client.enable_stats = off

-----------------------
MQTT Session Parameters
-----------------------

EMQ X creates a session for each MQTT connection:

.. code-block:: properties

    ## Max Number of Subscriptions, 0 means no limit.
    mqtt.session.max_subscriptions = 0

    ## Upgrade QoS?
    mqtt.session.upgrade_qos = off

    ## Max Size of the Inflight Window for QoS1 and QoS2 messages
    ## 0 means no limit
    mqtt.session.max_inflight = 32

    ## Retry Interval for redelivering QoS1/2 messages.
    mqtt.session.retry_interval = 20s

    ## Client -> Broker: Max Packets Awaiting PUBREL, 0 means no limit
    mqtt.session.max_awaiting_rel = 100

    ## Awaiting PUBREL Timeout
    mqtt.session.await_rel_timeout = 20s

    ## Enable Statistics: on | off
    mqtt.session.enable_stats = off

    ## Expired after 1 day:
    ## w - week
    ## d - day
    ## h - hour
    ## m - minute
    ## s - second
    mqtt.session.expiry_interval = 2h

..
    +----------------------------+------------------------------------------------------------+
    | session.max_subscriptions  | Maximum allowed subscriptions                              |
    +----------------------------+------------------------------------------------------------+
    | session.upgrade_qos        | Upgrade QoS according to subscription                      |
    +----------------------------+------------------------------------------------------------+
    | session.max_inflight       | Inflight window.                                           |
    |                            |                                                            |
    |                            | Maximum allowed simultaneous QoS1/2 packet.                |
    |                            |                                                            |
    |                            | 0 means unlimited. Higher value means higher throughput    |
    |                            |                                                            |
    |                            | while lower value means stricter packet transmission order.|
    +----------------------------+------------------------------------------------------------+
    | session.retry_interval     | Retry interval between QoS1/2 messages and PUBACK messages |
    +----------------------------+------------------------------------------------------------+
    | session.max_awaiting_rel   | Maximum number of packets awaiting PUBREL packet           |
    +----------------------------+------------------------------------------------------------+
    | session.await_rel_timeout  | Timeout for awaiting PUBREL                                |
    +----------------------------+------------------------------------------------------------+
    | session.enable_stats       | Enable session stats                                       |
    +----------------------------+------------------------------------------------------------+
    | session.expiry_interval    | Session expiry time.                                       |
    +----------------------------+------------------------------------------------------------+

.. list-table::
    :widths: 50 100
    :header-rows: 0

    * - session.max_subscriptions
      - Maximum allowed subscriptions
    * - session.upgrade_qos
      - Upgrade QoS according to subscription
    * - session.max_inflight
      - Inflight window. Maximum allowed simultaneous QoS1/2 packet. 0 means unlimited. Higher value means higher throughput while lower value means stricter packet transmission order.
    * - session.retry_interval
      - Retry interval between QoS1/2 messages and PUBACK messages
    * - session.max_awaiting_rel
      - Maximum number of packets awaiting PUBREL packet
    * - session.await_rel_timeout
      - Timeout for awaiting PUBREL
    * - session.enable_stats
      - Enable session stats
    * - session.expiry_interval
      - Session expiry time.


------------------
MQTT Message Queue
------------------

EMQ X creates a message queue to cache QoS1/2 messages in each session. Two types of messages are put into this queue:

1. Offline messages for persistent session.

2. Messages which should be pended if inflight window is full.

Queue Parameters:

.. code-block:: properties

    ## Type: simple | priority
    mqtt.mqueue.type = simple

    ## Topic Priority: 0~255, Default is 0
    ## mqtt.mqueue.priority = topic/1=10,topic/2=8

    ## Max queue length. Enqueued messages when persistent client disconnected,
    ## or inflight window is full. 0 means no limit.
    mqtt.mqueue.max_length = 0

    ## Low-water mark of queued messages
    mqtt.mqueue.low_watermark = 20%

    ## High-water mark of queued messages
    mqtt.mqueue.high_watermark = 60%

    ## Queue Qos0 messages?
    mqtt.mqueue.store_qos0 = true

Description of queue parameters:

+-----------------------------+-------------------------------------------------------------+
| mqueue.type                 | Queue type. simple: simple queue, priority: priority queue  |
+-----------------------------+-------------------------------------------------------------+
| mqueue.priority             | Topic priority                                              |
+-----------------------------+-------------------------------------------------------------+
| mqueue.max_length           | Max queue size, infinity means no limit                     |
+-----------------------------+-------------------------------------------------------------+
| mqueue.low_watermark        | Low watermark                                               |
+-----------------------------+-------------------------------------------------------------+
| mqueue.high_watermark       | High watermark                                              |
+-----------------------------+-------------------------------------------------------------+
| mqueue.store_qos0           | Maintain Queue for QoS0 messages                            |
+-----------------------------+-------------------------------------------------------------+

----------------------
Sys Interval of Broker
----------------------

System interval of publishing $SYS/# message:

.. code-block:: properties

    ## System Interval of publishing broker $SYS Messages
    mqtt.broker.sys_interval = 60

-----------------
PubSub Parameters
-----------------

.. code-block:: properties

    ## PubSub Pool Size. Default should be scheduler numbers.
    mqtt.pubsub.pool_size = 8

    mqtt.pubsub.by_clientid = true

    ## Subscribe Asynchronously
    mqtt.pubsub.async = true

----------------------
MQTT Bridge Parameters
----------------------

EMQ X nodes can be bridged:

.. code-block:: properties

    ## Bridge Queue Size
    mqtt.bridge.max_queue_len = 10000

    ## Ping Interval of bridge node. Unit: Second
    mqtt.bridge.ping_down_interval = 1

---------------------------
Plugin Config File Location
---------------------------

EMQ X plugin config file location:

.. code-block:: properties

    ## Dir of plugins' config
    mqtt.plugins.etc_dir ={{ platform_etc_dir }}/plugins/

    ## File to store loaded plugin names.
    mqtt.plugins.loaded_file = {{ platform_data_dir }}/loaded_plugins

--------------
MQTT Listeners
--------------

Listeners enabled by default are: MQTT, MQTT/SSL, MQTT/WS and MQTT/WS/SSL:

+-----------+-----------------------------------+
| 1883      | MQTT/TCP port                     |
+-----------+-----------------------------------+
| 8883      | MQTT/SSL port                     |
+-----------+-----------------------------------+
| 8083      | MQTT/WebSocket port               |
+-----------+-----------------------------------+
| 8084      | MQTT/WebSocket/SSL port           |
+-----------+-----------------------------------+

EMQ X allows enabling multiple listeners on a single server, and the most important listener parameters are listed below:

+-----------------------------------+--------------------------------------------------+
| listener.tcp.${name}.acceptors    | TCP Acceptor pool                                |
+-----------------------------------+--------------------------------------------------+
| listener.tcp.${name}.max_clients  | Max concurrent TCP connections                   |
+-----------------------------------+--------------------------------------------------+
| listener.tcp.${name}.rate_limit   | max TCP connection speed rate. 10KB/s: "100,10"  |
+-----------------------------------+--------------------------------------------------+
| listener.tcp.${name}.access.${id} | limitation on client IP Address                  |
+-----------------------------------+--------------------------------------------------+

-------------------------
MQTT/TCP Listener - 1883
-------------------------

.. code-block:: properties

    ## External TCP Listener: 1883, 127.0.0.1:1883, ::1:1883
    listener.tcp.external = 0.0.0.0:1883

    ## Size of acceptor pool
    listener.tcp.external.acceptors = 8

    ## Maximum number of concurrent clients
    listener.tcp.external.max_clients = 1024

    #listener.tcp.external.mountpoint = external/

    ## Rate Limit. Format is 'burst,rate', Unit is KB/Sec
    #listener.tcp.external.rate_limit = 100,10

    #listener.tcp.external.access.1 = allow 192.168.0.0/24

    listener.tcp.external.access.2 = allow all

    ## TCP Socket Options
    listener.tcp.external.backlog = 1024

    #listener.tcp.external.recbuf = 4KB

    #listener.tcp.external.sndbuf = 4KB

    listener.tcp.external.buffer = 4KB

    listener.tcp.external.nodelay = true

------------------------
MQTT/SSL Listener - 8883
------------------------

One way SSL authentication by default:

.. code-block:: properties

    ## SSL Listener: 8883, 127.0.0.1:8883, ::1:8883
    listener.ssl.external = 8883

    ## Size of acceptor pool
    listener.ssl.external.acceptors = 4

    ## Size of acceptor pool
    listener.ssl.external.acceptors = 16

    ## Maximum number of concurrent clients
    listener.ssl.external.max_clients = 102400

    ## Authentication Zone
    ## listener.ssl.external.zone = external

    ## listener.ssl.external.mountpoint = inbound/

    ## Rate Limit. Format is 'burst,rate', Unit is KB/Sec
    ## listener.ssl.external.rate_limit = 100,10

    listener.ssl.external.access.1 = allow all

    ### TLS only for POODLE attack
    ## listener.ssl.external.tls_versions = tlsv1.2,tlsv1.1,tlsv1

    listener.ssl.external.handshake_timeout = 15s

    listener.ssl.external.handshake_timeout = 15s

    listener.ssl.external.keyfile = {{ platform_etc_dir }}/certs/key.pem

    listener.ssl.external.certfile = {{ platform_etc_dir }}/certs/cert.pem

    ## listener.ssl.external.cacertfile = {{ platform_etc_dir }}/certs/cacert.pem

    ## listener.ssl.external.verify = verify_peer

    ## listener.ssl.external.fail_if_no_peer_cert = true

    ## listener.ssl.external.secure_renegotiate = off

    ### A performance optimization setting, it allows clients to reuse
    ### pre-existing sessions, instead of initializing new ones.
    ### Read more about it here.
    listener.ssl.external.reuse_sessions = on

    ### Use the CN or DN value from the client certificate as a username.
    ### Notice: 'verify' should be configured as 'verify_peer'
    ## listener.ssl.external.peer_cert_as_username = cn

------------------------------
MQTT/WebSocket Listener - 8083
------------------------------

.. code-block:: properties

    ## HTTP and WebSocket Listener
    listener.http.external = 8083

    listener.http.external.acceptors = 4

    listener.http.external.max_clients = 64

    ## listener.http.external.zone = external

    listener.http.external.access.1 = allow all

----------------------------------
MQTT/WebSocket/SSL Listener - 8084
----------------------------------

By default one way SSL authentication:

.. code-block:: properties

    ## External HTTPS and WSS Listener

    listener.https.external = 8084

    listener.https.external.acceptors = 4

    listener.https.external.max_clients = 64

    ## listener.https.external.zone = external

    listener.https.external.access.1 = allow all

    ## SSL Options
    listener.https.external.handshake_timeout = 15s

    listener.https.external.keyfile = {{ platform_etc_dir }}/certs/key.pem

    listener.https.external.certfile = {{ platform_etc_dir }}/certs/cert.pem

    ## listener.https.external.cacertfile = {{ platform_etc_dir }}/certs/cacert.pem

    ## listener.https.external.verify = verify_peer

    ## listener.https.external.fail_if_no_peer_cert = true

-----------------
Erlang VM Monitor
-----------------

.. code-block:: properties

    ## Long GC, don't monitor in production mode for:
    sysmon.long_gc = false

    ## Long Schedule(ms)
    sysmon.long_schedule = 240

    ## 8M words. 32MB on 32-bit VM, 64MB on 64-bit VM.
    sysmon.large_heap = 8MB

    ## Busy Port
    sysmon.busy_port = false

    ## Busy Dist Port
    sysmon.busy_dist_port = true

