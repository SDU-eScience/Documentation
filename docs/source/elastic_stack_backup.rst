Elastic Stack
==============

Elastic Stack Components
------------------------

There are four main components from the Elastic Stack:

* Logstash - Processes logs
* Elasticsearch - Stores logs
* Kibana - Web interface for searching and visualizing logs
* Filebeat - Installed on iRODS servers that will send audit logs to Logstash

Elastic Stack Installation
--------------------------

We installed the filebeat on irods nodes and logstash, elasticsearch and kibana on the index node. We had to add the Elastic repository first before installing all these components. The ansible playbook for adding the Elastic repository for index mode is shown as below.

.. code-block:: yml

   - hosts: index
     become: true
     tasks:
       - name: add Elastic repo
         yum_repository:
          name: Elastic repository for 5.x packages
          description: Elastic Repository
          baseurl: https://artifacts.elastic.co/packages/5.x/yum
          gpgcheck: 1
          gpgkey: https://artifacts.elastic.co/GPG-KEY-elasticsearch
          enabled: 1
          autorefresh: 1
          type: rpm-md

After adding the Elastic repository, using the following YAML to install the filebeat, logstash, elasticsearch and kibana perspectively.

.. code-block:: yml

   - name: install filebeat
     yum:
      name: filebeat
      state: present

   - name: install logstash
     yum:
      name: logstash
      state: present

    - name: install elasticsearch
      yum:
       name: elasticsearch
       state: present
 
    - name: install kibana
      yum:
       name: kibana
       state: present

Updating the yum packahges after each installation

.. code-block:: yml

   - name: upgrade all packages
     yum:
      name: '*'
      state: latest


ELK Configuration
------------------

Filebeat configuration
^^^^^^^^^^^^^^^^^^^^^^

Filebeat configuration file is in YAML format, which locates at ``/etc/filebeat/filebeat.yml``. Under paths sub section which belongs to the Filebeat prospectors section, we commented out the default and added new entries to specify the path for the iRODS's log file.



.. code-block:: yml



   # Paths that should be crawled and fetched. Glob based paths.

   paths:

     - /var/lib/irods/log/audit.log*

     - c:\programdata\elasticsearch\logs\*



Under Logstash output sub section which belongs to the Outputs section, we defined to use Logstash as the outputs when sending the iRODS'slog file as data collection by the filebeat.



.. code-block:: yml



   output.logstash:

     # The Logstash hosts

     hosts: ["unit03.esciencecloud.sdu.dk:5044”]

Logstash configuration
^^^^^^^^^^^^^^^^^^^^^^^

Logstash configuration file is in the JSON format. It is in our case called ``audit.conf`` and  locates at ``/etc/logstash/conf.d``. It has three defined sections-``ínput``, ``filter`` and ``output``.

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

al terminal if you want to access Kibana web portal with ``http://localhost:5601`` throug your local browser.



Forward the port 5601 from your local terminal.



.. code-block:: bash



   ssh -L 5601:172.22.240.12:5601 username@130.225.164.200 -N





Access Kibana web portal with ``http://localhost:5601`` and click the ``audit_log2`` index on the left side. The Kibana dashboard for monitoring our iRODS grid looks like the following.



.. figure::  images/kibana.png

   :align:   center
