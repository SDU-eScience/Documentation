.. eScienceCloud documentation master file, created by
   sphinx-quickstart on Fri Aug 18 12:10:07 2017.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Welcome to SDU eScienceCloud's documentation!
=============================================
This project was proposed by SDU eScience Center, which is structured as a competent research-based organization that has the ownership of the common research infrastructure for eScience, provides user support across the faculties and develops future e-infrastructures andservices.

The key components for this project are listed below as well as their usage descriptions, installations and deployments.

Components
==========
* `ansible`_
* `iRODS`_
* `ceph`_
* `irods-re-audit plugin and elastic stack`_
* `singularity`_
* `postgreSQL`_
* `Java JSF/Primefaces`_
* `HPC`_

ansible
========
We installed components against HPC nodes by ansible, which is a radically simple IT automation engine. Keycomponents which we installed by ansible are shown as below.
   
+-------------+-------------+--------------+-----------+
|Node         |Version      |Provider      |Playbooks  |
+=============+=============+==============+===========+
|Ceph Monitor |2:12.1.4-0   |@ceph_stable  |all.yml    |
|Ceph OSDs    |2:12.1.4-0   |@ceph_stable  |osds.yml   |
|             |             |              |mons.yml   |
+-------------+-------------+--------------+-----------+
|Index        |2.4.6-1      |@elasticsearch|           |
+-------------+-------------+--------------+-----------+
|Database     |9.6.4-1      |@pgdg96       |           |          
+-------------+-------------+--------------+-----------+
|iRods        |4.2.1-1      |@renci-irods  |irods.yml  | 
+-------------+-------------+--------------+-----------+


For more information on our ansible playbooks please refer to `<https://github.com/SDU-eScience/eScienceCloud/tree/master/ansible/playbooks>`_


iRODS
=====

iRODS usage description
-----------------------

iRODS is an open source data management software used by research organizations and government agencies worldwide. It is a middleware which in our case sits above the Ceph filesystem and our application.

We use iRODS mainly in three ways

* Manage data objects and metadata
* Configure resource
* Secure collaboration

iRODS deployment overview
-------------------------

Our iRODS deployment includes three key components

* an iRODS Metadata Catalog(iCAT) database
* a Catalog Provider
* a Catalog Consumer

iCAT database instance setup
----------------------------

iRODS neither creates nor manages a database instance itself, just the tables within the database. Therefore, the database instance should be created and configured before installing iRODS. PostgreSQL is the database for us which is used to implement the iCAT database. The following PSQL is used for setting up our database.

.. code-block:: sql

   $ (sudo) su - postgres
   postgres$ psql
   psql> CREATE USER irods WITH PASSWORD 'testpassword';
   psql> CREATE DATABASE "ICAT";
   psql> GRANT ALL PRIVILEGES ON DATABASE "ICAT" TO irods;


Run ``\l`` to view the permissions

.. code-block:: sql

    postgres=# \l
                                      List of databases
    Name   |Owner    |Encoding |Collate    |Ctype      |Access privileges   
    -------+---------+---------+-----------+-----------+-----------------------
    ICAT   |postgres |UTF8     |en_US.UTF-8|en_US.UTF-8|=Tc/postgres+
           |         |         |           |           |postgres=CTc/
           |         |         |           |           |postgres+
           |         |         |           |           |irods=CTc/p

iRODS Catalog Provider installation
-----------------------------------
We use ansible to install iRODS and the ``irods.yml`` is the playbook for our iRODS installation. It locates at the root of ansible-irods folder. There are several dependencies for installing iRODS-such dependencies as Extra Packages for Enterprise Linux (EPEL) and iRODS database plugin for future connecting iRODS withpostgreSQL database. Basically to finish the installation, you have to complete the following three steps

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

iRODS Catalog Provider configuration 
------------------------------------

After installation, run ``setup_irods.py`` script to fullfil information of the iRODS Catalog Provider.

.. code-block:: bash

   $ (sudo) python /var/lib/irods/scripts/setup_irods.py


The asked information is shown as below

.. code-block:: console

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

For more information on our iRODS ansible playbooks please refer to `<https://github.com/SDU-eScience/eScienceCloud/tree/master/ansible/playbooks/ansible_irods>`_

Ceph
====

Ceph overview
-------------

Ceph is an open-source, massively scalable, software-defined storage system which provides object, block and file system storage from a single clustered platform. Our Ceph storage cluster includes three monitor nodes and two OSD nodes.  

* OSD: an Object Storage Daemon (OSD) stores data, handles data replication, recovery, backfilling, rebalancing, and provides some monitoring information to Ceph Monitors by checking other Ceph OSD Daemons for a heartbeat.


* Monitor: a Ceph Monitor maintains maps of the cluster state, including the monitor map, the OSD map, the Placement Group (PG) map, and the CRUSH map.

Ceph installation
-----------------
We use ansible to install Ceph through repository channels. That means we get Ceph installed through a new repository. It is managed by the ``ceph_origin`` variable. If ``ceph_origin`` is set to ``repository``, you have to choose which repository you want to download Ceph. It is controlled by the ``ceph_origin`` variable. In our case we use ``community`` option, which fetches packages from `the official community Ceph repositories <http://download.ceph.com>`_.

Ceph configuration
----------------------------
Before installing ceph, we create our inventory file, playbook and configuration for our Ceph storage cluster.

Inventory
^^^^^^^^^
The ansible inventory file defines the hosts in our cluster and what roles each host plays in our Ceph cluster. Inventory file related to Ceph installation looks like:

.. code-block:: yml

   [ceph-mon]
   cephmon[1:3].esciencecloud.sdu.dk

   [ceph-osd]
   cephosd[1:2].esciencecloud.sdu.dk

Playbook
^^^^^^^^
We have our ``site.yml`` playbook to pass to the ``ansible-playbook`` command when deploying our Ceph storage cluster. It locates at the root of ``ansible-ceph`` folder. This playbook installs dependencies like ``python2``, defines deployment design and assigns roleto server groups. The roles assigned to mons server looks like:


.. code-block:: yml

   - hosts: mons
   gather_facts: false
   become: True
   roles:
     - ceph-defaults
     - ceph-common
     - ceph-mon

The configuration for our Ceph storage cluster will be set by the use of ansible variable. All of these variables are defined in ``group_vars/`` directory. Part of our configuration that deploys luminous version of Ceph with OSDs looks like this:

.. code-block:: yml

   ceph_origin = repository
   ceph_repository = community
   ceph_stable_release: luminous
   monitor_interface: eno49
   osd_scenario: non-collocated


For more information on our Ceph ansible playbooks please refer to `<https://github.com/SDU-eScience/eScienceCloud/tree/master/ansible/playbooks/ansible_ceph>`_

irods-re-audit plugin and elastic stack
========================================

* irods-re-audit plugin

We have rewritten an iRODs audit plugin for auditing iRODS grid. And our iRODS audit plugin improves from the `official iRODS audit plugin <https://irods.org/2016/12/auditing-irods-with-the-audit-plugin-and-elastic-stack/>`_. The official iRODS audit plugin generates messages whenever a dynamic policy enforcement point (PEP) is fired, and our iRODS audit plugin filters the messages from the messages which is generated by the official iRODS audit plugin. The outputs is stored in an rotating log file which is placed at ``/var/lib/irods/log/audit.log``. Each line of this log file contains a JSON message which is corresponding to a user's event. For example, the following JSON message is generated after executes an ``iput`` command which means that uploads a file to iRODS collection.

.. code-block:: json

   {  
   "pid":4491,
   "level":"I",
   "msg":{  
      "ts":1505731837906,
      "completed":true,
      "type":"putObject",
      "accessedBy":{  
         "username":"rods",
         "zone":"tempZone",
         "type":""
      },
      "path":"/tempZone/home/rods/imada/irods_dashboard.json"
     }
   }


The following table lists the users' actions that can be logged by our iRODS audit plugin. 

+-------------------+--------------------+--------------------------------------------------------------------------+
|iCommands          |Message type        |Description                                                               |
+===================+====================+==========================================================================+
|iput               |putObject           |Store a file into iRODS.                                                  |
+-------------------+--------------------+--------------------------------------------------------------------------+      
|iget               |objectAccessed      |Get data-objects or collections from iRODS space, either to the specified |
|                   |                    |local area or to the current working directory.                           |            
+-------------------+--------------------+--------------------------------------------------------------------------+
|iadmin mkuser      |userCreation        |Create a new iRODS user in the ICAT database.                             |           
+-------------------+--------------------+--------------------------------------------------------------------------+
|igroupadmin mkgroup|userCreation        |Create a user group.                                                      |
+-------------------+--------------------+--------------------------------------------------------------------------+
|icp                |objectAccess        |Copies an irods data-object (file) or collection (directory) to another   |
|                   |dataObjCopy         |data-object or collection.                                                |
+-------------------+--------------------+--------------------------------------------------------------------------+
|irm                |dataObjRemove       |Remove one or more data-object or collection from iRODS space.            |
+-------------------+--------------------+--------------------------------------------------------------------------+
|imv                |dataObjRename       |Moves/renames an irods data-object (file) or collection (directory) to    |
|                   |                    |another, data-object or collection.                                       |
+-------------------+--------------------+--------------------------------------------------------------------------+
|ichmod             |objectModeModified  |Modify access to dataObjects (iRODS files) and collections (directories). |       
+-------------------+--------------------+--------------------------------------------------------------------------+
|iadmin mkresc      |createResource      |Create (register) a new coordinating or storage resource.                 |
+-------------------+--------------------+--------------------------------------------------------------------------+
|imeta add          |metadataModification|Modify iRODS metadata.                                                    |
+-------------------+--------------------+--------------------------------------------------------------------------+
|imeta rm           |metadataModification|Delete iRODS metadata.                                                    |
+-------------------+--------------------+--------------------------------------------------------------------------+

For more information on our iRODS audit plugin please refer to `<https://github.com/SDU-eScience/irods-re-audit>`_

* elastic stack

``Elastic stack`` is an overall solutions which aims to reliably and securely take data from any source, in any format, and 
search, analyze, and visualize it in real time. It provides a collection of open source software tools and in our case we use        ``Filebeat``, ``Logstash``, ``Elasticsearch`` and ``Kibana``. ``Filebeat`` sends data from ``/var/lib/irods/log/audit.log`` to ``Logstash``,which then transforms and stores them in ``Elasticsearch``. ``Elasticsearch`` stores and indexes all the data. Finally the data canbe queried and displayed graphically from ``Elasticsearch`` to ``Kibana``.

filebeat
^^^^^^^^

logstash
^^^^^^^^^
elasticsearch
^^^^^^^^^
kibana
^^^^^^^^^

singularity
===========


postgreSQL
==========


Java JSF/Primefaces
===================


HPC
====
.. toctree::
   :maxdepth: 2
   :caption: Contents:
   
Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`

