o Recovery time depends on these factors:
-----------------------------------------

1. Your transaction log size - So each replica in a collection writes to a transaction log. This gets rolled over on a (hard) commit. So if you restart between commits then that portion needs to be replayed.

2. When a replica is in recovery, it starts writing all updates sent by the leader in this period to the transaction log. So if the node was in recovery when you restarted then this could be a reason why it took long.

Thus if your transaction log on disk is large then that is probably the reason for a slow recovery time. A short auto (hard) commit setting will ensure a small transaction log and faster recovery, but be careful not to make it too short as that can effect your indexing performance

3. When a replica tries to come up after the restart, it asks the shard leader if any document updates have taken place. If the number is less than 100 then SolrCloud does a PeerSync and gets all the updated documents, but when it's more than 100 it does an index replication from the leader. Full replication will kick in if the index has changed because of merges, optimize, expungeDeletes etc). If a full replication is kicking in then this could be a major reason for recovery time. You could grep for "fullCopy=true" to see if that is happening

4. At the time of restarting a node if your overseer queue already has a lot of pending requests queued up and with the node restart theOverseer would have to process these state changes in addition, causing a pile up in the Overseer queue and thus it might take time to process the events - hence the slow recovery. You can look at your overseer queue size here - http://host:port/solr/#/~cloud?view=tree and click on /overseer -> queue and see it's children_count number.

5. As a general advice for restarting nodes in a SolrCloud cluster, its advised to restart one node at a time, waiting for a node to completely come up ( all shards show up as active in the Cloud dashboard ) . Also you should bounce the Overseer as the last node as this causes minimum switching of the overseer

o Update:
---------

Starting with Solr 5.1 the 100 count mentioned on point 3 is configurable .  To take that count higher , you need to modify the <updateLog> section within your solrconfig.xml file

```
<updateLog>
    <str name="dir">${solr.ulog.dir:}</str>
    <str name="numRecordsToKeep">${solr.ulog.numRecordsToKeep:100}</str>
</updateLog>
```

#### ***Note the documents that are are sent via the "PeerSync" are not streamed. They are sent as a POST request. So setting a number too high will cause the request to fail. The default POST size limit for Jetty is 20MB ( http://wiki.eclipse.org/Jetty/Howto/Configure_Form_Size#Changing_the_Maximum_Form_Size_for_All_Apps_in_the_JVM ) , so one needs to make sure the numRecordsToKeep count multiplied by document size doesn't exceed that.
