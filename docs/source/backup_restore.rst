Elasticsearch
=============

Snapshot and Restore
--------------------
We stored snapshots of individual indices in a local file system based repository. These snapshots can be restored relatively.

Repositories
ˆˆˆˆˆˆˆˆˆˆˆˆ
Our snapshots repository is stored /etc/elasticsearch/backup. And we registered a snapshot repository in /etc/elasticsearch/elasticsearch.yml before perform snapshot and restore operations.We created a new snapshot repository for each version we used.

.. code-block:: yaml
   
   path.repo: /etc/elasticsearch/backup


.. code-block:: bash
   
   curl -XPUT 'localhost:9200/_snapshot/backup_2.4?pretty' -H 'Content-Type: application/json' -d'
   {
     "type": "fs",
     "settings": {
     "location": "/etc/elasticsearch/backup"
    }
   }
   '

Snapshot
ˆˆˆˆˆˆˆˆ
Created a snapshot with the name snapshot_1 in our repository my_backup

.. code-block:: bash

  curl -XPUT 'localhost:9200/_snapshot/backup_2.4/snapshot_1?wait_for_completion=true&pretty' -H 'Content-Type: application/json' -d'
  {
  "indices": "audit_log2",
  "ignore_unavailable": true,
  "include_global_state": false
  }
  '



