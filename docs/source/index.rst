.. eScienceCloud documentation master file, created by
   sphinx-quickstart on Fri Aug 18 12:10:07 2017.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Welcome to SDU eScienceCloud's documentation!
=============================================
This project was proposed by SDU eSciene Center, which is structured as a competent research-based organization that has the ownership ofthe common research infrastructure for eScience, provides user support across the faculties and develops future e-infrastructures and services.

Components
==========
* `ansible`_
* `iRODS`_
* `ceph`_
* `elasticstack`_
* `singularity`_
* `postgreSQL`_
* `Java JSF/Primefaces`_

ansible
========
We installed components against HPC nodes by ansible, which is a radically simple IT automation engine. Key components which we installed are shown as below.
   
+-----------------+------------------------------+------------------+--------------+
|Node             |Packages                      |Version           |Provider      |
+=================+==============================+==================+==============+
|Ceph Monitor     |ceph-mon                      |2:12.1.4-0        |@ceph_stable  |
+-----------------+------------------------------+------------------+--------------+
|Ceph OSDs        |ceph-osd                      |2:12.1.4-0        |@ceph_stable  |
+-----------------+------------------------------+------------------+--------------+
|Index            |elasticsearch.noarch          |2.4.6-1           |@elasticsearch|
+-----------------+------------------------------+------------------+--------------+
|Database         |postgresql96                  |9.6.4-1           |@pgdg96       |                       
+-----------------+------------------------------+------------------+--------------+
|iRods            |irods-server                  |4.2.1-1           |@renci-irods  |
+-----------------+------------------------------+------------------+--------------+
|                 |irods-database-plugin-postgres|4.2.1-1           |@renci-irods  |
+-----------------+------------------------------+------------------+--------------+




iRODS
=====

iRODS usage description
-----------------------

iRODS is an open source data management software used by research organizations and government agencies worldwide. It is a middleware which in our case sits above the Ceph filesystem and our application.
We use iRODS mainly in three ways-

* Manage data objects and metadata
* Configure resource
* Secure collaboration

iRODS deployment overview
-------------------------

Our iRODS deployment includes three key components

* an iRODS Metadata Catalog(iCAT) database
* a Catalog Provider
* a Catalog Consumers

iCAT database instance setup
----------------------------
iRODS neither creates nor manages a database instance itself, just the tables within the database. Therefore, the database instance should be created and configured before installing iRODS. PostgreSQL is the database that is used to implement the iCAT database.The following PSQL is used for setting up our database.

.. code-block:: sql
   :linenos:

   $ (sudo) su - postgres
   postgres$ psql
   psql> CREATE USER irods WITH PASSWORD 'testpassword';
   psql> CREATE DATABASE "ICAT";
   psql> GRANT ALL PRIVILEGES ON DATABASE "ICAT" TO irods;


view permissions
^^^^^^^^^^^^^^^^
.. code-block:: sql
   :linenos:

    postgres=# \l
                                      List of databases
       Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
    -----------+----------+----------+-------------+-------------+-----------------------
     ICAT      | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =Tc/postgres     +
               |          |          |             |             | postgres=CTc/postgres+
               |          |          |             |             | irods=CTc/p
iRODS Catalog Provider setup
----------------------------

1. install the public key and add YUM repository 

.. code-block:: bash
   :linenos:

   $ (sudo) rpm --import https://packages.irods.org/irods-signing-key.asc
   $ wget -qO - https://packages.irods.org/renci-irods.yum.repo | sudo tee
   /etc/yum.repos.d/renci-irods.yum.repo

2. install Extra Packages for Enterprise Linux (EPEL), iRODS and iRODS database plugin for postgreSQL

.. code-block:: bash
   :linenos: 

   $ (sudo) yum install epel-release
   $ (sudo) yum install irods-server irods-database-plugin-postgres

3. upgrade all installed packages

.. code-block:: bash
   :linenos:

   $ (sudo) yum update irods-server irods-database-plugin-postgres

4. run `setup_irods.py` script to fullfil information of the iRODS Catalog Provider

.. code-block:: bash
   :linenos:

   $ (sudo) python /var/lib/irods/scripts/setup_irods.py

   
The asked information is shown as below

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
   :linenos:

   $ ienv



ceph
====
Ceph overview
-------------

what is ceph
^^^^^^^^^^^^^
Ceph is an open-source, massively scalable, software-defined storage system which provides object, block and file system storage from a single clustered platform.

ceph components
^^^^^^^^^^^^^^^

* OSD: an Object Storage Daemon (OSD) stores data, handles data replication, recovery, backfilling, rebalancing, and provides some monitoring information to Ceph Monitors by checking other Ceph OSD Daemons for a heartbeat.

* Monitor: a Ceph Monitor maintains maps of the cluster state, including the monitor map, the OSD map, the Placement Group (PG) map, and the CRUSH map.

Ceph installation
-----------------
We use ansible to install ceph, and the installed version is luminous.

Ceph configuration and usage
----------------------------
Before installing ceph, we create our inventory file, playbook and configuration for our ceph cluster.

Inventory
^^^^^^^^^
The ansible inventory file defines the hosts in our cluster and what roles each host plays in our ceph cluster. Inventory file related to ceph installation looks like:

.. code-block:: yml

   [ceph-mon]
   cephmon[1:3].esciencecloud.sdu.dk

   [ceph-osd]
   cephosd[1:2].esciencecloud.sdu.dk

Playbook
^^^^^^^^

Ceph configuration
^^^^^^^^^^^^^^^^^^^
ceph-ansible configuration
""""""""""""""""""""""""""
ceph.conf configuration
"""""""""""""""""""""""
OSD configuration
"""""""""""""""""


elasticstack
=============
* logstash
* elasticsearch
* kibana


singularity
===========


postgreSQL
==========


Java JSF/Primefaces
===================



.. toctree::
   :maxdepth: 2
   :caption: Contents:
   
Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`

