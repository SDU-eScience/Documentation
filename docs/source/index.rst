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
* `Java JSF/Primefaces`_
* `iRODS`_
* `elasticstack`_
* `ceph`_
* `singularity`_
* `postgres`_

ansible
'''''''
We installed components against HPC nodes by ansible, which is a radically simple IT automation engine. Key components which we installed are shown as below.
   
+-----------------+------------------------------+------------------+--------------+
|Node             |Packages                      |Version           |Provider      |
+=================+==============================+==================+==============+
|Ceph Monitor     |ceph-mon                      |2:12.1.4-0        |@cep_stable   |
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


Java JSF/Primefaces
'''''''''''''''''''
IRODS
'''''

* `Functionality of IRODS`_
* `Postgres database instance setup`_
* `IRODS installation and setup`_
* `IRODS deployment overview`_

Functionality of IRODS
''''''''''''''''''''''

IRODS is an open source data management software used by research organizations and government agencies worldwide. It is a middleware which in our case sits above the Ceph filesystem and our application.
We use IRODS mainly in three ways:

* Manage data objects and metadata
* Configure resource
* Secure collaboration

Postgres database instance setup
''''''''''''''''''''''''''''''''
A database instance should be created and configured before installing iRods. The following PSQL is used for setting up our database.

.. code-block:: sql
   :linenos:

   $ (sudo) su - postgres
   postgres$ psql
   psql> CREATE USER irods WITH PASSWORD 'testpassword';
   psql> CREATE DATABASE "ICAT";
   psql> GRANT ALL PRIVILEGES ON DATABASE "ICAT" TO irods;


View permissions

.. code-block:: sql
   :linenos:

    postgres=# \l
                                      List of databases
       Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
    -----------+----------+----------+-------------+-------------+-----------------------
     ICAT      | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =Tc/postgres     +
               |          |          |             |             | postgres=CTc/postgres+
               |          |          |             |             | irods=CTc/postgres



IRODS installation and setup
''''''''''''''''''''''''''''

1. install the public key and add YUM repository 

.. code-block:: bash
   :linenos:

   sudo rpm --import https://packages.irods.org/irods-signing-key.asc
   wget -qO - https://packages.irods.org/renci-irods.yum.repo | sudo tee
   /etc/yum.repos.d/renci-irods.yum.repo

2. install Extra Packages for Enterprise Linux (EPEL), IRODS and IRODS database plugin for postgres

.. code-block:: bash
   :linenos: 

   sudo yum install epel-release
   sudo yum install irods-server irods-database-plugin-postgres

3. upgrade all installed packages

.. code-block:: bash
   :linenos:

   sudo yum update irods-server irods-database-plugin-postgres


IRODS deployment overview
'''''''''''''''''''''''''

Our iRODS deployment includes:

* an IRODS Metadata Catalog(iCAT) database
* a Catalog Provider
* a Catalog Consumers

PostgreSQL is the database that is used to implement the iCAT database.

elasticstack
'''''''''''''
* logstash
* elasticsearch
* kibana

ceph
''''
singularity
'''''''''''
postgres
''''''''

.. toctree::
   :maxdepth: 2
   :caption: Contents:
   
Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`

