# <b>Hadoop Distributed File System</b>
A comprehensive guide to understanding HDFS and it’s inner workings.
<br></br>
From a computing perspective, there are essentially 2 types of scaling:
- <b>Vertical Scaling</b>: We simply add more RAM and storage to a single computer/machine aka “node”.
- <b> Horizontal Scaling</b>:  We add more nodes connected through a common network, thereby increasing the overall capacity of the system. With that in mind, let’s dive right in.

<br></br>
<b>Block Size</b>

![Alt text](https://miro.medium.com/max/443/0*mtdCvQf2EMWP5rME.gif)
<br></br>

When a file is saved in HDFS, the file is broken into smaller chunks or “blocks”, as can be seen in the GIF above. The number of blocks is dependent on the “Block Size”. The default is 128 MB but can be changed/configured easily.

In our example, a 500 MB file needs to be broken into blocks of 128 MB. 500/128 = 3 blocks of 128 MB and 1 block of 116 MB. The residual block space of 12 MB is returned back to the name node for usage elsewhere, thus preventing any wastage. This is true of any file system really, for example, Windows NTFS has a block size between 4 KB and 64 KB depending on file size (up to 256 TB). Considering petabytes and above for Big Data processing, KBs would be highly inefficient, as you can imagine. This is why HDFS has a block size of 128 MB.

<br></br>
<b>Replication Factor</b>

![Alt text](https://miro.medium.com/max/750/0*n2BT4I4Nipi_m6Qk.gif)
<br></br>

HDFS is a fault-tolerant and resilient system, meaning it prevents a failure in a node from affecting the overall system’s health and allows for recovery from failure too. In order to achieve this, data stored in HDFS is automatically replicated across different nodes.

How many copies are made? This depends on the “replication factor. By default, it is set to 3 i.e. 1 original and 2 copies. This is also easily configurable.

In the GIF just above, we see a file broken into blocks and each block replicated across other data nodes for redundancy.

<br></br>
## <b>Storage and Replication Architecture</b>
![Alt Text](Images/Storage_and_Replication_Architecture.png?raw=true "Storage and Replication Architecture")
<br></br>

Hadoop Distributed File System (HDFS) follows a Master — Slave architecture, wherein, the ‘Name Node’ is the master and the ‘Data Nodes’ are the slaves/workers. This simply means that the name node monitors the health and activities of the data node. The data node is where the file is actually stored in blocks.

Let's continue with the same example of a file of size = 500 MB in the images above. With HDFS’ default block size of 128 MB, this file is broken into 4 blocks B1 — B4. Please note that A — E are our Data Nodes. With HDFS’ default replication factor of 3, the blocks are replicated across our 5 node cluster. Block B1 (in yellow) is replicated across Nodes A, B and D and so on and so forth (follow the coloured lines).
Here, the Name Node maintains the metadata, i.e. data about data. Which replica of which block of which file is stored in which node is maintained in NN — replica 2 of block B1 of file xyz.csv is stored in node B.

So a file of size 500 MB requires a total storage capacity in HDFS of 1500 MB due to its replication. This is abstracted from the end users’ perspective and the user can only see 1 file of size 500 MB stored within HDFS.

<br></br>

---
Now’s a good time as any to get some hands on:

[HDFS Commands](Data_Storage/HDFS/Commands.md)

---

<br></br>

<b>The Block Replication Algorithm</b>

![Alt text](/Images/Block_Replication_Algorithm.png?raw=true "Block Replication Algorithm")
<br></br>

The algorithm starts by searching for the topology.map file under HDFS’ default configuration folder. This .map file contains metadata information on all the available racks and nodes it contains. In our example case in the image above, we have 2 racks and 10 data nodes.

Once the file is divided into blocks, the first copy of the first block is inserted into the rack and data node which is nearest to the client i.e. end-user. The copy of this first block is created and moved onto the next available rack i.e. Rack 2 through TCP/IP and stored in any available data node. Another copy is created here and moved onto the next available rack through TCP/IP and so on. But, since we have only 2 racks, the algorithm looks for the next available data node on the same rack (i.e. Rack 2) and stores the third copy there. This redundancy is put in place such that even if one rack fails, we still have the second rack to retrieve the data, thus enabling fault-tolerance and resilience.

<br></br>
## <b>High Availability Architecture</b>

In Hadoop 1.x, the ecosystem was shipped with 1 Name Node only, resulting in a single point of failure. There was a Secondary or Backup Name Node which took over an hour of manual intervention to bring up. Subsequently, any data lost was irrecoverable.

In Hadoop 2.x, High Availability as an alternative to standard mode, was provided. In the standard mode, you still have a primary and secondary name node. In the high availability mode, you have an active and passive name node.

![Alt text](/Images/HDFS_High_Availability_Architecture.png?raw=true "HDFS High Availability Architecture")
<br></br>

The data nodes send an activity update to the “Active” Name Node (every 5 seconds at minimum — configurable). This metadata is synced to the “Stand by” Name Node in real-time. Thus, when the “Active” fails, the “Stand by” has all the necessary metadata to switch over.

The Zookeeper, through its Fail-over Controller, monitors the health of the Active and Stand by Name Nodes through a heart beat or instant notification it receives from each NN (every 5 seconds, again configurable). It also has the information of all the stand by name nodes available (Hadoop 3.x allows for multiple stand by name nodes).

Thus, a connectivity between the data nodes, name nodes and zookeeper is established. The moment an active name node fails, the Zookeeper elects an appropriate stand by name node and facilitates the automatic switch over. The Stand by becomes the new Active Name Node and broadcasts this election to all the data nodes. The data nodes now send their activity updates to the newly elected Active Name Node within a few minutes.

<br></br>
<b>What is NameNode Metadata?</b>

![Alt text](/Images/NameNode_Metadata.png?raw=true "NameNode Metadata")
<br></br>

The name node (NN) metadata consists of two persistent files, namely, FsImage — namespace and Edit logs — transaction logs (insert, append)

<br></br>
<b>Namespace & FsImage</b>

In every file system, there is a path to the required files — `On Windows: C:\Users\username\learning\BigData\namenode.txt and on Unix: /usr/username/learning/BigData/namenode.txt.`
HDFS follows the Unix way of namespace. This namespace is stored as part of the FsImage. Every detail of the file i.e. who, what, when, etc. is also stored in the FsImage snapshot. The FsImage is stored on the disk for consistency, durability and security.

<br></br>
<b>Edit logs</b>

Any real-time changes to all files are logged in what is known as “Edit logs”. These are recorded in-memory (RAM) and contain every little detail of the change and the respective file/block.

On HDFS startup, the metadata is read from the FsImage and the changes are written to Edit Logs. Once the data is recorded for the day in Edit Logs, it is flushed down onto the FsImage. This is how the two work in tandem.

As an aside, the FsImage and Edit Logs are not human readable. They are binary-compressed (serialized) and stored in the file system. However, for debugging purposes it can be converted into an xml format to be read using [Offline Image Viewer](https://hadoop.apache.org/docs/r1.2.1/hdfs_imageviewer.html).

<br></br>
## <b>How does NameNode Metadata Sync?</b>

As you can imagine or see in the image ‘HDFS High Availability Architecture’, the name node metadata is a single point of failure, hence this metadata is replicated to introduce redundancy and enable high-availability (HA).

<br></br>
<b>Shared Storage</b>

![Alt text](/Images/Shared_Storage_Sync.png?raw=true "Shared Storage Sync")
<br></br>

We now know that there exists an Active Name Node and a Standby Name Node. Any change in the active is synced in real-time to the shared folder/storage i.e. network file system (NFS). This NFS is accessible to the standby, which downloads all of the relevant incremental information in real-time to maintain the sync between the Namenodes. Thus, in the event of a failure of the active, the standby name node already has all the relevant information to continue “business as usual” post fail-over. This is not used in the Production environment.

<br></br>
<b>Quorum Journal Node (QJN)</b>

![Alt text](/Images/Quorum_Journal_Node.png?raw=true "Quorum Journal Node")
<br></br>

“Quorum” means minimum required to facilitate an event. The term is generally used in politics; it is the minimum number of representatives required to conduct proceedings in the house.

Here, we use this concept to determine the minimum number of journal nodes aka quorum that is needed to establish a majority and maintain metadata sync.

The image shows three (always odd) journal nodes (process threads not physical nodes) that help to establish metadata sync. When an Active NN receives a change, it pushes it to majority of the QJ Nodes (follow a single colour). The Standby NN, in real-time, requests the majority number of QJ Nodes for the required metadata to establish the sync.

The minimum number for QJN to function is 3 and the quorum/majority is determined by the following formula:

```
Q = (N+1)/2
where N = total number of Journal Nodes
For example, if we have N=5, the quorum/majority would be established by (5+1)/2 i.e. 3. The metadata change would be written to 3 journal nodes.
```

The QJN is the preferred Production method of metadata sync as it is also “highly available”. In the event of a failure of any of the QJ nodes, any of the remaining nodes are available to provide the required data to maintain metadata sync. Thus, the standby already has all the relevant information to continue “business as usual” post fail-over.

This brings us to the end of my comprehensive guide on HDFS and it’s inner workings.
<br></br>

---
References:

1. [HDFS Architecture](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html) (2019), Apache Hadoop, ASF
2. [Managing HDFS](https://docs.cloudera.com/documentation/enterprise/6/6.3/topics/admin_hdfs_config.html), Cloudera
---

<br></br>
Lastly:

- Should you have any feedback/suggestions, I would love to hear them.
- If you'd like to contribute, feel free to submit a pull request.
- If you'd like for me to address a specific topic in further detail, do not hesitate to connect with me.
<br></br>

Originally published on my Medium account [here](https://towardsdatascience.com/hadoop-distributed-file-system-b09946738555)