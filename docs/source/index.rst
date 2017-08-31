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
We installed components against HPC nodes by ansile, which is a radically simple IT automation engine. Key components which we installed are shown as below.
   
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

IRODS is an open source data management software used by research organizations and government agencies worldwide. It is a middleware which in our case sits above the Ceph filesystem and our application.
We use IRODS mainly in three ways:

* Manage data objects and metadata
* Configure resource
* Secure collaboration

Our iRods deployment includes:

* an IRODS Metadata Catalog(iCAT) database
* a Catalog Provider

PostgreSQL is the database that is used to implement the iCAT database.

elasticstack
'''''''''''''
ceph
''''
singularity
'''''''''''
postgres
''''''''

System overview
===============

Mauris rhoncus condimentum leo a vulputate. Morbi placerat tristique quam, ac facilisis felis ultricies a. Maecenas fermentum in eros eget condimentum. Sed in iaculis est. Fusce aliquam felis ut convallis dapibus. Sed massa metus, facilisis ut eleifend sed, accumsan ut mauris. Phasellus sollicitudin, odio in pulvinar fringilla, dolor sapien fringilla ipsum, nec bibendum felis dolor quis ipsum 


.. toctree::
   :maxdepth: 2
   :caption: Contents:
   
Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`

