Elastic Stack
==============

Elastic Stack Components
------------------------

* There are four main components from the Elastic Stack:

  * Logstash - Processes logs
  * Elasticsearch - Stores logs
  * Kibana - Web interface for searching and visualizing logs
  * Filebeat - Installed on iRODS servers that will send audit logs to Logstash

Elastic Stack Installation
--------------------------

* Clone repo and setup our ansible hosts file

.. code-block:: bash

   git clone https://github.com/sadsfae/ansible-elk
   cd ansible-elk

* Edit the Ansible Inventory hosts file with our target Elastic Server (Index Server) and Elastic Client (iRODS Servers) information
  
  * ELK Server
  
  .. code-block:: yml
  
     [index]
     index.esciencecloud.sdu.dk

  
  * ELK Client
  
  .. code-block:: yml
  
     [irods]
     irods[1:2].esciencecloud.sdu.dk
  

* Run elk.yml playbook against our ELK server.
 
.. code-block:: yml
  
   ansible-playbook -i hosts install/elk.yml


Elastic Stack Configuration
----------------------------

Filebeat configuration
^^^^^^^^^^^^^^^^^^^^^^

Filebeat configuration file is in YAML format, which locates at ``/etc/filebeat/filebeat.yml``. Under paths sub section which belongs to the Filebeat prospectors section, we commented out the default and added new entries to specify the path for the iRODS's log file.



.. code-block:: yml


   # Paths that should be crawled and fetched. Glob based paths.
     paths:
       - /var/lib/irods/log/audit.log*
       - c:\programdata\elasticsearch\logs\*


Under Logstash output sub section which belongs to the Outputs section, we defined to use Logstash as the outputs when sending the iRODS's log file as data collection by the filebeat.


.. code-block:: yml

   output.logstash:
   # The Logstash hosts
     hosts: ["unit03.esciencecloud.sdu.dk:5044”]

Logstash configuration
^^^^^^^^^^^^^^^^^^^^^^^

Logstash configuration file is in the JSON format. It is in our case called ``audit.conf`` and  locates at ``/etc/logstash/conf.d``. It has three defined sections - ``ínput``, ``filter`` and ``output``.

* The input section configures Logstash to read the messages from the "beats" queue.
* The date filter parses dates from [msg][ts] fields, and then timestamp as UNIX_MS which is one of the logstash accepted timestamp.
* The output writes the resulting information to Elasticsearch under the "audit_log2" index.
* The stdout writes the resulting output in an easily readable format to the stdout. This can be commented out once debugging is finished.

The Logstash configuration file - ``audit.conf`` is shown as below.

.. code-block:: yml

   input {
     beats {
       port => 5044
       codec => "json"
           }
   }

   filter {
     date  {
       match => ["[msg][ts]", "UNIX_MS"]
           }
   }

   output {
     elasticsearch {
       hosts => "localhost:9200"
       manage_template => false
       index => "audit_log2"
     }

     stdout {
       codec => rubydebug {
     }
   }
  }


Kibana configuration
^^^^^^^^^^^^^^^^^^^^^

Forward the port 5601 from your local terminal if you want to access Kibana web portal with ``http://localhost:5601`` through your local browser.

.. code-block:: bash


   ssh -L 5601:172.22.240.12:5601 username@130.225.164.200 -N


Access Kibana web portal with ``http://localhost:5601`` and click the ``audit_log2`` index on the left side. The Kibana dashboard for monitoring our iRODS grid looks like the following.


.. figure::  images/kibana.png

   :align:   center


Log shipment diagram
---------------------

The following diagram illustrates how our iRODS audit log is shipped, processed, stored and visualized by using Elastic Stack.

.. figure::  images/ELK-workflow.png

   :align:   center
