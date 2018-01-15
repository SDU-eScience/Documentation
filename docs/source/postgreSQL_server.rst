.. _PostgreSQL:

PostgreSQL
===========

PostgreSQL Hot Standby & Streaming Replication Settings
-------------------------------------------------------
We have three PostgreSQL database instances-one primary and two hot standbys. These two hot standbys are replicas of the primary so that we can run read-only queries on them. PostgreSQL provides streaming replication for the capability to continuously ship and apply the WAL XLOG records to our standby servers in order to keep them current. To set up hot standby and enable streaming replication, our configurations are shown as below.

1. create an user named replication with REPLICATION privileges

.. code-block:: psql

   $ CREATE ROLE replication WITH REPLICATION PASSWORD 'ourpassword' LOGIN


2. Set up connections and authentication on the primary server

Edit ``postgresql.conf``

.. code-block:: text-only

   listen_addresses = '*'


Edit ``pg_hba.conf``

.. code-block:: text-only

   host  replication  replication  127.0.0.1/32  trust


3. Set up the streaming replication related parameters on the primary server

Edit ``postgresql.conf``

.. code-block:: text-only

   # To enable read-only queries on a standby server
   # wal_level must be set to hot standby
   wal_level = hot_standby

   # Specifies the maximum number of concurrent connections
   # from standby servers or streaming base backup clients
   max_wal_senders = 5

  # Enable WAL archiving on the primary to an archive directory accessible from

   # the standby. If wal_keep_segments is a high enough number to retain the WAL

   # segments required for the standby server, this is not necessary

   archive_mode    = on

   archive_command = 'cp %p /var/lib/pgsql/archive/%f'



4. Make a base backup by copying the primary server's data directory to the standby servers



We use pg_basebackup to fetch the entire data directory of our PostgreSQL installation from the primary and placing it onto the standby server. Run pg_basebackup command asthe database superuser, in our case is postgres, to make sure the permissions are preserved. -R option creates a minimal recovery command file which is ``recovery.conf`` for standbys within their data directories in order for streaming replication.



.. code-block:: psql



   pg_basebackup -U replication -h 172.22.240.11 -p 5432 -D /var/lib/pgsql/9.6/data1 -R

   pg_basebackup -U replication -h 172.22.240.11 -p 5432 -D /var/lib/pgsql/9.6/data2 -R





5. Set up replication-related parameters, connections and authentication in the standby servers like the primary, so that the standby might work as a primary after failover



6. Enable read-only queries on the standby servers



   Edit ``postgresql.conf``



   .. code-block:: text-only



      hot_standby = on





7. Start postgreSQL in the standby servers. It will start streaming replication.
