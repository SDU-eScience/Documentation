Pgpool-II
==========

pgpool-II is a middleware that works between PostgreSQL servers and a PostgreSQL database client. It takes an advantage of the replication feature to reducethe load on each PostgreSQL server by distributing SELECT queries among multiple servers, improving system's overall throughput.


Pgpool Configuration
---------------------

The pgpool-II user account
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Add a Unix user account to run pgpool-II. The user name is pgpool.


.. code-block:: bash



   $ (sudo) useradd pgpool



Configuring pcp.conf
^^^^^^^^^^^^^^^^^^^^

PCP commands are UNIX commands which manipulate pgpool-II via the network. A PCP user and password for us has been declared in ``pcp.conf`` in ``/etc`` directory. A user postgres and its associated password has been written as one line using the following format:



.. code-block:: text-only

 

   postgres:[md5 encrypted password]



[md5 encrypted password] can be produced with the pg_md5 command. The following command used to generate the md5 encrypted password for postgres user for us.



.. code-block:: bash


   $ pg_md5 your_password
   1060b7b46a3bd36b3a0d66e0127d0517


Configuring pgpool.conf
^^^^^^^^^^^^^^^^^^^^^^^
In pgpool-II we use streaming replication mode which means that PostgreSQL servers operates streaming replication. And load balancing is possible in this mode. We reuse the sample configuration file for streaming replication mode from pgpool-II installation. The file is located in ``/etc/pgpool.conf.sample-stream``. The main part of our pgpool configrations is shown as below.

* connection settings

  .. code-block:: text-only

     # '*' accepts all incoming connections
     listen_addresses = '*'

     # The port number used by pgpool-II to listen for connections
     port = 9999

     # The directory where the UNIX domain socket accepting connections
     # for pgpool-II will be created
     socket_dir = '/var/run/postgresql/'

     # '*' accepts all incoming connections
     pcp_listen_addresses = '*'

     # The port number used by PCP process to listen for connections
     pcp_port = 9898

     # The directory where the UNIX domain socket accepting connections
     # for pgpool-II will be created
     pcp_socket_dir = '/var/run/postgresql/'


* running mode settings

  To enable ``streaming replication mode`` in pgpool-II, we have to firstly turn on ``master_slave_mode`` which is used to couple pgpool-II with another master/slave replication

software in our case is ``streaming replication``, which PostgreSQL server is responsible for doing the actual data replication.


  .. code-block:: text-only



     # Setting to on enables the master/slave mode

     master_slave_mode = on



     # Suitable for PostgreSQL's built-in streaming replication function

     master_slave_sub_mode = 'stream'



* backend settings


  We have three backends which pgpool communicates with. And they all needs to be specified by some parameters.


  .. code-block:: text-only



     # Host name or IP address to connect to for backend 0

     backend_hostname0 = 'localhost'



     # Port number for backend 0

     backend_port0 = 5432



     # Weight for backend 0 (only in load balancing mode)

     backend_weight0 = 1



     # Data directory for backend 0

     backend_data_directory0 = '/data'



     # Controls various backend behavior

     # ALLOW_TO_FAILOVER or DISALLOW_TO_FAILOVER

     backend_flag0 = 'ALLOW_TO_FAILOVER'



     backend_hostname1 = 'localhost'

     backend_port1 = 5433

     backend_weight1 = 1

     backend_data_directory1 = '/data1'

     backend_flag1 = 'ALLOW_TO_FAILOVER'



     backend_hostname2 = 'localhost'

     backend_port2 = 5434

     backend_weight2 = 1

     backend_data_directory2 = '/data2'

     backend_flag2 = ‘ALLOW_TO_FAILOVER'


* load balancing settings
 
We enabled load balancing so that pgpool-II could send the writing queries to the primay node, and other queries got load balanced among all backend nodes. To which node the load balancing mechanism sends read queries is decided at the session start time and will not be changed until the session ends. For more information on which query should be sent to which node in load balancing in streaming replication mode, please refer to `<http://www.pgpool.net/docs/latest/en/html/runtime-config-load-balancing.html>`_.
