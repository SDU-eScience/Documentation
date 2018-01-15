.. _iRODS:

iRODS
=====

iRODS Usages
------------

iRODS is an open source data management software used by research organizations and government agencies worldwide. It is a middleware which in our case sits above the Ceph filesystem and our application.

* We use iRODS mainly in three ways

  * Manage data objects and metadata
  * Configure resource
  * Secure collaboration

iRODS Deployments
-----------------

* Our iRODS deployment includes three key components

  * an iRODS Metadata Catalog(iCAT) database
  * a Catalog Provider
  * a Catalog Consumer

iCAT Database Instance Setups
-----------------------------

iRODS neither creates nor manages a database instance itself, just the tables within the database. Therefore, the database instance should be created and configured before installing iRODS. PostgreSQL is the database for us which is used to implement the iCAT database. The following PSQL was used for setting up our database.

.. code-block:: psql

   $ (sudo) su - postgres
   postgres$ psql
   psql> CREATE USER irods WITH PASSWORD 'testpassword';
   psql> CREATE DATABASE "ICAT";
   psql> GRANT ALL PRIVILEGES ON DATABASE "ICAT" TO irods;


Run ``\l`` to view the permissions.

.. code-block:: psql

    postgres=# \l

                                      List of databases
    Name   |Owner    |Encoding |Collate    |Ctype      |Access privileges
    -------+---------+---------+-----------+-----------+-----------------------
    ICAT   |postgres |UTF8     |en_US.UTF-8|en_US.UTF-8|=Tc/postgres+
           |         |         |           |           |postgres=CTc/
           |         |         |           |           |postgres+
           |         |         |           |           |irods=CTc/p


iRODS Server Installation
-----------------------------------
We used ansible to install iRODS and the ``irods.yml`` is the playbook for our iRODS installation. It locates at the root of ansible-irods folder. There are several dependencies for installing iRODS-such dependencies as Extra Packages for Enterprise Linux (EPEL) and iRODS database plugin for future connecting iRODS with postgreSQL database. 

* Basically to finish the installation, you have to complete the following three steps

  * Install the public key and add the repository
  * Install irods-server irods-database-plugin-postgres
  * Upgrade all the installed packages

The following code which is included in our ``irods.yml`` shows the installation of EPEL.

.. code-block:: yml

   - name: add epel repo
     yum_repository:
      name: epel
      description: Fedora EPEL Repository
      baseurl: https://download.fedoraproject.org/pub/epel/$releasever/
               $basearch/
      gpgcheck: yes
      gpgkey: http://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-7
      enabled: yes

iRODS Server Configuration
------------------------------------

After installation, run ``setup_irods.py`` script to fullfil information of the iRODS Catalog Provider.

.. code-block:: bash

   $ (sudo) python /var/lib/irods/scripts/setup_irods.py


The asked information is shown as below

.. code-block:: text-only

   1. Service Account

   *  Service Account Name
   *  Service Account Group
   *  Catalog Service Role

   2. Database Connection

   *  ODBC Driver
   *  Database Server's Hostname or IP
   *  Database Server's Port
   *  Database Name
   *  Database User
   *  Database Password
   *  Stored Passwords Salt

   3. iRODS Server Options

   *  Zone Name
   *  Zone Port
   *  Parallel Port Range (Begin)
   *  Parallel Port Range (End)
   *  Control Plane Port
   *  Schema Validation Base URI
   *  iRODS Administrator Username

   4. Keys and Passwords

   *  zone_key
   *  negotiation_key
   *  Control Plane Key
   *  iRODS Administrator Password

   5. Vault Directory

Once a server is up and running, you can view the environment settings by running

.. code-block:: bash

   $ ienv


