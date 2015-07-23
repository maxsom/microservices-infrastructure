Calico
======

.. versionadded:: 0.4

`Calico <http://www.projectcalico.org>`_ is used in the project to add the IP
per container functionality. Calico connects Docker containers via IP no matter
which worker node they are on. Calico uses :doc:`etcd` to distribute information
about workloads, endpoints, and policy to each worker node. Endpoints are
network interfaces associated with workloads. Calico is deployed in a Docker
container on each worker node and managed by systemd. Any workload managed by
Calico is registered as a service in Consul.

Modes
^^^^^

Calico can run a public cloud environment that doesn't allow either L3 peering
or L2 connectivity between Calico hosts. Calico will then route traffic between
Calico hosts using IP in IP. At this time, the full node-to-node BGP mesh is
supported and configured in CiscoCloud only. Other cloud environments are set up
with the IP in IP mode.

Mesos
^^^^^

systemd unit file for Mesos slave is updated with the new ``DOCKER_HOST``
environment variable:

.. code-block:: shell

   DOCKER_HOST=localhost:2377

This allows Calico to set up networking automatically routing Docker API
requests through the `Powerstrip <https://github.com/clusterhq/powerstrip>`_
proxy which is running on port ``2377`` on each Mesos slave host. It sets
a dependency meaning if Calico Docker container is not running, Mesos slave
will refuse to start accordingly.

Marathon
^^^^^^^^

When you start containers on top of Marathon, you will need to add two
environment variables to your JSON file: ``CALICO_IP`` and ``CALICO_PROFILE``.
You can assign IP address to ``CALICO_IP`` explicitly or set ``auto`` and it
will be allocated automatically. If profile set with ``CALICO_PROFILE`` doesn't
exist, it will be created automatically. If you don't provide the two variables,
the Docker default network settings will be applied.

Example:

.. code-block:: json

   {
     "container": {
       "type": "DOCKER",
       "docker": {
         "image": "busybox",
         "parameters":[{"key": "env", "value": "CALICO_IP=auto"},
                       {"key": "env", "value": "CALICO_PROFILE=dev"}]
       }
     },
     "id": "testapp",
     "instances": 1,
     "cpus": 0.1,
     "mem": 32,
     "uris": [],
     "cmd": "while sleep 10; do date -u +%T; done"
   }

Consul
^^^^^^

When you start a workload on Marathon with the proper environment variables
as ``CALICO_IP`` and ``CALICO_PROFILE``, the workload is getting registered
in Consul as a service. The Powerstrip logic was extended in this case.
The registered name is constructed in this way: ``MARATHON_APP_ID`` plus
``-calico`` suffix. For example, if you create a workload with the name of
``testapp``, then the ``testapp-calico`` service will be registered in Consul.

Thus, you have the option to query Consul in two ways. In order to obtain Docker
host IP addresses where your workload is running:

.. code-block:: shell

   dig @localhost -p 8600 testapp.service.consul

To resolve IP addresses from Calico network:

.. code-block:: shell

   dig @localhost -p 8600 testapp-calico.service.consul

calicoctl
^^^^^^^^^

You can use the command line tool ``calicoctl`` to manually configure and start
Calico services, interact with etcd datastore, define and apply network and
security policies, etc.

Examples:

.. code-block:: shell

   calicoctl status
   calicoctl profile show --detailed
   calicoctl endpoint show --detailed
   calicoctl pool show

Logging
^^^^^^^

All components log to directories under ``/var/log/calico`` inside
the calico-docker container. By default this is mapped to
the ``/var/log/calico`` directory on the host. Files are automatically rotated,
and by default 10 files of 1MB each are kept.

Variables
---------

You can use these variables to customize your Calico installation. Please refer
to :doc:`etcd` configuration for more details.

.. data:: etcd_service_name

   Set ``ETCD_AUTHORITY`` environment variable that is used by Calico Docker
   container and the CLI tool ``calicoctl``. The value of this variable is
   a Consul service that must be resolved via DNS

   Default: ``etcd.service.consul``

.. data:: etcd_client_port

   Port for etcd client communication

   Default: ``2379``

.. data:: calico_network

   Containers are assigned IPs from this network range

   Default: ``192.168.0.0/16``

.. data:: calico_profile

   Endpoints are added to this profile for interconnectivity

   Default: ``dev``
