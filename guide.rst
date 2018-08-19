
.. _guide:

===========
User Guide
===========

---------------------------
MQTT PUBLISH & SUBSCRIPTION
---------------------------

MQTT is light weight publish/subscribe message transport protocol. EMQ X is a message broker implementing MQTT protocol:

.. image:: ./_static/images/guide_1.png

When EMQ X is started, any clients that support MQTT protocol can connect to the EMQ X broker and then publish / subscribe messages.

MQTT client library: https://github.com/mqtt/mqtt.github.io/wiki/libraries

E.g., using mosquitto_sub/pub CLI to subscribe to topics and publish messages.

.. code-block:: properties

    mosquitto_sub -t topic -q 2
    mosquitto_pub -t topic -q 1 -m "Hello, MQTT!"

MQTT V3.1.1 Standard: http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/mqtt-v3.1.1.html

Configure MQTT/TCP listener in 'emqx.conf':

.. code-block:: properties

    ## TCP Listener: 1883, 127.0.0.1:1883, ::1:1883
    listener.tcp.external = 1883

    ## Size of acceptor pool
    listener.tcp.external.acceptors = 8

    ## Maximum number of concurrent clients
    listener.tcp.external.max_clients = 1024

The default port MQTT/SSL listener is 8883:

.. code-block:: properties

    ## SSL Listener: 8883, 127.0.0.1:8883, ::1:8883
    listener.ssl.external = 8883

    ## Size of acceptor pool
    listener.ssl.external.acceptors = 4

    ## Maximum number of concurrent clients
    listener.ssl.external.max_clients = 512

.. _shared_subscription:

-------------------
Shared Subscription
-------------------

Shared Subscription supports Load balancing to distribute MQTT messages between multiple subscribers in the same group:

.. image:: ./_static/images/guide_2.png

Two ways to create a shared subscription:

+----------------------+-------------------------------------------+
|  Subscription prefix | Example                                   |
+----------------------+-------------------------------------------+
| $queue/              | mosquitto_sub -t '$queue/topic'           |
+----------------------+-------------------------------------------+
| $share/<group>/      | mosquitto_sub -t '$share/group/topic'     |
+----------------------+-------------------------------------------+

.. _local_subscription:

------------------
Local Subscription
------------------

The *EMQ* broker will not create global routes for `Local Subscription`, and only dispatch MQTT messages on local node.

.. code-block:: shell

    mosquitto_sub -t '$local/topic'

    mosquitto_pub -t 'topic'

Usage: subscribe a topic with `$local/` prefix.

.. _fastlane_subscription:

---------------------
Fastlane Subscription
---------------------

EMQ X supports Fastlane subcription, it improves the message dispatching efficiency:

.. image:: _static/images/guide_3.png

Usage of Fastlane: *$fastlane/* prefix + topic

Limitation on Fastlane:

1. CleanSession = true
2. Qos = 0

Fastlane is suitable for IoT sensor data collection:

.. image:: _static/images/guide_4.png

.. _http_publish:

----------------
HTTP Publish API
----------------

EMQ X provides a HTTP publish API. Application server or Web server can publish MQTT messages through this API::

    HTTP POST http://host:8080/mqtt/publish

Web servers (PHP/Java/Python/NodeJS/Ruby on Rails) publish MQTT messages using HTTP POST:

.. code-block:: bash

    curl -v --basic -u user:passwd -d "qos=1&retain=0&topic=/a/b/c&message=hello from http..." -k http://localhost:8080/mqtt/publish

Parameters of the HTTP API:

+---------+----------------+
| Name    | Description    |
+=========+================+
| client  | clientid       |
+---------+----------------+
| qos     | QoS(0, 1, 2)   |
+---------+----------------+
| retain  | Retain(0, 1)   |
+---------+----------------+
| topic   | Topic          |
+---------+----------------+
| message | Payload        |
+---------+----------------+

.. NOTE:: The API uses HTTP Basic Authentication.

--------------
MQTT WebSocket
--------------

EMQ X supports MQTT WebSocket connection, web browsers can directly connect to broker through MQTT protocol:

+-------------------------+----------------------------+
| WebSocket URI:          | ws(s)://host:8083/mqtt     |
+-------------------------+----------------------------+
| Sec-WebSocket-Protocol: | 'mqttv3.1' or 'mqttv3.1.1' |
+-------------------------+----------------------------+

Dashboard plugin provides a test page for MQTT WebSocket connection::

    http://127.0.0.1:18083/websocket.html

EMQ X uses an embedded HTTP server to implement MQTT WebSocket and HTTP publish interface. Configurable in file 'etc/emqx.conf':

.. code-block:: properties

    ## HTTP and WebSocket Listener
    mqtt.listener.http.external = 8083
    mqtt.listener.http.external.acceptors = 4
    mqtt.listener.http.external.max_clients = 64

.. _sys_topic:

--------------------
$SYS -- System Topic
--------------------

EMQ X periodically publishes its server status, MQTT protocol statistics, client connection status to topics starting with '$SYS/'.

$SYS topic path starts with "$SYS/brokers/{node}/", where '${node}' is the Erlang node name::

    $SYS/brokers/emqx@127.0.0.1/version

    $SYS/brokers/emqx@host2/uptime

.. NOTE:: By default, only clients on localhost are allowed to subscribe to $SYS topics, this can be changed in 'etc/acl.config'.

$SYS publish interval can be changed in 'etc/emq.conf':

.. code-block:: properties

    ## System Interval of publishing broker $SYS Messages
    mqtt.broker.sys_interval = 60

.. _sys_brokers:

Broker Version, Uptime and Description
---------------------------------------

+--------------------------------+---------------------------+
| Topic                          | Description               |
+================================+===========================+
| $SYS/brokers                   | Node list in cluster      |
+--------------------------------+---------------------------+
| $SYS/brokers/${node}/version   | EMQ X version             |
+--------------------------------+---------------------------+
| $SYS/brokers/${node}/uptime    | EMQ X Up-Time             |
+--------------------------------+---------------------------+
| $SYS/brokers/${node}/datetime  | EMQ X system time         |
+--------------------------------+---------------------------+
| $SYS/brokers/${node}/sysdescr  | EMQ X version description |
+--------------------------------+---------------------------+

.. _sys_clients:

MQTT Client Connection status
-----------------------------

$SYS topic prefix: $SYS/brokers/${node}/clients/

+--------------------------+--------------------------------------------+------------------------------------+
| Topic                    | Data(JSON)                                 | Description                        |
+==========================+============================================+====================================+
| ${clientid}/connected    | {ipaddress: "127.0.0.1", username: "test", | Publish when a client connected    |
|                          |  session: false, version: 3, connack: 0,   |                                    |
|                          |  ts: 1432648482}                           |                                    |
+--------------------------+--------------------------------------------+------------------------------------+
| ${clientid}/disconnected | {reason: "keepalive_timeout",              | Publish when a client disconnected |
|                          |  ts: 1432749431}                           |                                    |
+--------------------------+--------------------------------------------+------------------------------------+

'connected' message (JSON Data):

.. code-block:: json

    {
        ipaddress: "127.0.0.1",
        username:  "test",
        session:   false,
        protocol:  3,
        connack:   0,
        ts:        1432648482
    }

'disconnected' message (JSON Data):

.. code-block:: json

    {
        reason: normal,
        ts:     1432648486
    }

.. _sys_stats:

Statistics -- System Statistics
-------------------------------

$SYS prefix: $SYS/brokers/${node}/stats/

Clients -- Client Statistics
............................

+---------------------+---------------------------------------------+
| Topic               | Description                                 |
+---------------------+---------------------------------------------+
| clients/count       | Current client count                        |
+---------------------+---------------------------------------------+
| clients/max         | Maximum concurrent clients allowed          |
+---------------------+---------------------------------------------+

Sessions -- Session Statistics
...............................

+---------------------+---------------------------------------------+
| Topic               | Description                                 |
+---------------------+---------------------------------------------+
| sessions/count      | Current session count                       |
+---------------------+---------------------------------------------+
| sessions/max        | Maximum concurrent session allowed          |
+---------------------+---------------------------------------------+

Subscriptions -- Subscription Statistics
........................................

+---------------------+---------------------------------------------+
| Topic               | Description                                 |
+---------------------+---------------------------------------------+
| subscriptions/count | Current subscription count                  |
+---------------------+---------------------------------------------+
| subscriptions/max   | Maximum subscription allowed                |
+---------------------+---------------------------------------------+

Topics -- Topic Statistics
...........................

+---------------------+---------------------------------------------+
| Topic               | Description                                 |
+---------------------+---------------------------------------------+
| topics/count        | Current topic count (cross-node)            |
+---------------------+---------------------------------------------+
| topics/max          | Max number of topics                        |
+---------------------+---------------------------------------------+

Metrics -- Traffic/Packet/Message Statistics
----------------------------------------------

Topic prefix: $SYS/brokers/${node}/metrics/

Traffic
............

+---------------------+---------------------------------------------+
| Topic               | Description                                 |
+---------------------+---------------------------------------------+
| bytes/received      | Traffic received in bytes                   |
+---------------------+---------------------------------------------+
| bytes/sent          | Traffic sent in bytes                       |
+---------------------+---------------------------------------------+

MQTT Packet Statistics
......................

+--------------------------+---------------------------------------+
| Topic                    | Description                           |
+--------------------------+---------------------------------------+
| packets/received         | Count of received MQTT packets        |
+--------------------------+---------------------------------------+
| packets/sent             | Count of sent MQTT packets            |
+--------------------------+---------------------------------------+
| packets/connect          | Count of received CONNECT packets     |
+--------------------------+---------------------------------------+
| packets/connack          | Count of sent CONNECT packets         |
+--------------------------+---------------------------------------+
| packets/publish/received | Count of received PUBLISH packets     |
+--------------------------+---------------------------------------+
| packets/publish/sent     | Count of sent PUBLISH packets         |
+--------------------------+---------------------------------------+
| packets/subscribe        | Count of received SUBSCRIBE packets   |
+--------------------------+---------------------------------------+
| packets/suback           | Count of sent SUBACK packets          |
+--------------------------+---------------------------------------+
| packets/unsubscribe      | Count of received UNSUBSCRIBE packets |
+--------------------------+---------------------------------------+
| packets/unsuback         | Count of sent UNSUBACK packets        |
+--------------------------+---------------------------------------+
| packets/pingreq          | Count of received PINGREQ packets     |
+--------------------------+---------------------------------------+
| packets/pingresp         | Count of sent PINGRESP packets        |
+--------------------------+---------------------------------------+
| packets/disconnect       | Count of received DISCONNECT packets  |
+--------------------------+---------------------------------------+

MQTT Message Statistic
......................

+--------------------------+--------------------------------+
| Topic                    | Description                    |
+--------------------------+--------------------------------+
| messages/received        | Count of received messages     |
+--------------------------+--------------------------------+
| messages/sent            | Count of sent messages         |
+--------------------------+--------------------------------+
| messages/retained        | Count of retained messages     |
+--------------------------+--------------------------------+
| messages/dropped         | Count of dropped message       |
+--------------------------+--------------------------------+

.. _sys_alarms:

Alarms -- System Alarms
------------------------

$SYS prefix: $SYS/brokers/${node}/alarms/

+------------------+------------------+
| Topic            | Description      |
+------------------+------------------+
| ${alarmId}/alert | New alarm        |
+------------------+------------------+
| ${alarmId}/clear | Clear alarm      |
+------------------+------------------+

.. _sys_sysmon:

Sysmon -- System Monitor
------------------------

$SYS prefix: $SYS/brokers/${node}/sysmon/

+------------------+----------------------+
| Topic            | Description          |
+------------------+----------------------+
| long_gc          | Long GC Time         |
+------------------+----------------------+
| long_schedule    | Long Scheduling time |
+------------------+----------------------+
| large_heap       | Large Heap           |
+------------------+----------------------+
| busy_port        | Port busy            |
+------------------+----------------------+
| busy_dist_port   | Dist Port busy       |
+------------------+----------------------+

.. _trace:

-----
Trace
-----

EMQ X supports tracing of packets from a particular client or messages published to a particular topic.

Tracing by client:

.. code-block:: bash

    ./bin/emqx_ctl trace client "clientid" "trace_clientid.log"

Tracing by topic:

.. code-block:: bash

    ./bin/emqx_ctl trace topic "topic" "trace_topic.log"

Query trace:

.. code-block:: bash

    ./bin/emqx_ctl trace list

Stop tracing:

.. code-block:: bash

    ./bin/emqx_ctl trace client "clientid" off

    ./bin/emqx_ctl trace topic "topic" off

