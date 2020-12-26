![Alt Text](/Images/HDFS_Commands.png?raw=true "HDFS Commands")
<br></br>

# <b>HDFS Commands</b>

Assumptions:

1. You’ve successfully installed [Cloudera Manager](/Supporting_Docs/Cloudera_Manager_on_Google_Cloud.md).
2. You know how to startup/shutdown your environment, if not please take a look at how to do so [here](/Supporting_Docs/Cloudera_Manager_Startup_and_Shutdown.md).
3. The project’s idea is to learn how to use our pipeline [here](/Data_Storage/HDFS/Comprehensive_Guide.md).

You want the GUI-way? :( sigh. Do the `initial prep` work and then scroll to the end of post. Please note, most functionality would be unavailable in the GUI way.

<br></br>
## <b>Initial prep</b>

Enter into `instance-1` now through the ssh option on your Google VM. Note, you can download the data into any instance really.

```
# Login into instance-1 using root
sudo su -
# Good time to check the amount of space allocated and available in your VM
df -h
Filesystem Size Used Avail Use% Mounted on
devtmpfs 5.8G 0 5.8G 0% /dev
tmpfs 5.8G 0 5.8G 0% /dev/shm
tmpfs 5.8G 8.5M 5.8G 1% /run
tmpfs 5.8G 0 5.8G 0% /sys/fs/cgroup
/dev/sda1 100G 13G 88G 13% /
cm_processes 5.8G 1.9M 5.8G 1% /run/cloudera-scm-agent/process
tmpfs 1.2G 0 1.2G 0% /run/user/1000
```

If you’ve diligently followed my installation post, you’d land up with the same. While not required, I would suggest to ensure you have upwards of 50 GB configured on each VM.

Details on the data set are in the footer of this post. Lets begin...

```
# Create a folder
mkdir /root/flightdatasets
# cd to download the dataset in this newly created folder
cd flightdatasets/
wget https://history.adsbexchange.com/downloads/samples/2020-01-01.zip
# Check the download
ls -lrth
# Unzip the dataset here itself — bunch of json files
unzip 2020–01–01.zip # Or alternatively extract just 10–20 files
```

Now that we have the files, lets push one of them into HDFS. Ensure you have brought up the Cloudera Manager environment.

```
# View the first 5 files
ls -U | head -5
```

By default HDFS has no folders created. Lets create a folder:

```
hdfs dfs -mkdir -p /user/projects/flightDatasets
```

You’ll receive an error saying `‘Permission denied: user=root, access=WRITE, inode=”/user”:hdfs:supergroup’.` This is because unlike unix, the superuser in HDFS is <b>not</b> root.

```
# Lets create a home directory for root under HDFS
sudo -u hdfs hadoop fs -mkdir /user/root
# Lets assign the ownership to root
sudo -u hdfs hadoop fs -chown root /user/root
# Now lets create the folder again
hdfs dfs -mkdir -p /user/root/projects/flightDatasets
# Quick test
hdfs dfs -ls /user/root/projects
```

---
<br></br>
## <b>Getting the data into HDFS</b>

- copyFromLocal

```
# Copying one of the files from local to HDFS
hdfs dfs -copyFromLocal 2020–01–01–0000Z.json /user/root/projects/flightDatasets/
```

so the data is in now...

```
# File System Check for some interesting details
hdfs fsck /user/root/projects/flightDatasets/ -files -blocks -replicaDetails
```

- put

Another option is to use put, which pretty much does the same thing as copyFromLocal. Notice the change in file name

```
hdfs dfs -put 2020–01–01–0001Z.json /user/root/projects/flightDatasets/
```

- moveFromLocal

The above 2 options was to copy/paste the data, but what if we want to move the data into HDFS?

```
hdfs dfs -moveFromLocal 2020–01–01–0003Z.json /user/root/projects/flightDatasets/
```
---
<br></br>
## <b>In and around HDFS</b>

If you’re from a unix background, you probably already know some/all these

- du vs df

Disk Usage information for a folder vs entire system

```
hdfs dfs -du /user/root/projects/flightDatasets
hdfs dfs -df
```

- cp, ls, rm, mv

Copying vs Moving data between HDFS locations

```
# First lets create a new folder
hdfs dfs -mkdir -p /user/root/projects/test
# Copying all files from flightDatasets to test
hdfs dfs -cp /user/root/projects/flightDatasets/* /user/root/projects/test/
# Quick test
hdfs dfs -ls /user/root/projects/test/
# Removing files from flightDatasets
# The “*” in the end ensures all files are removed
# The “-R” is not required in this case as it is required only for adding the recursive option
hdfs dfs -rm -R /user/root/projects/flightDatasets/*
# Moving the deleted files back from test to flightDatasets folder
hdfs dfs -mv /user/root/projects/test/* /user/root/projects/flightDatasets/
# Quick test
hdfs dfs -ls /user/root/projects/flightDatasets
hdfs dfs -ls /user/root/projects/test
```
---
<br></br>
## <b>Back to Local</b>

- copyToLocal, get

```
# New folder on my local
mkdir /root/test
# copyToLocal
hdfs dfs -copyToLocal /user/root/projects/flightDatasets/* /root/test/
# remove files and recopy using get
rm *.json
hdfs dfs -get /user/root/projects/flightDatasets/* /root/test/
```

On a related note, moveToLocal has not been implemented yet for hdfs.

For more commands, perhaps you’d like to visit [this page](https://hadoop.apache.org/docs/r2.6.3/hadoop-project-dist/hadoop-common/FileSystemShell.html).

---
<br></br>
## <b>The GUI Way</b>

Visit this page [here](/Supporting_Docs/Commands_The-GUI-Way.md)

---
<br></br>
## <b>The Dataset</b>

The Homepage — https://www.adsbexchange.com/

The data — https://history.adsbexchange.com/downloads/samples/

While you can go ahead and download any zipped file, I have chosen to go with the last one, i.e. unzip 2020–01–01.zip. (Latest Data + Latest fields)

More details on the fields can be found [here](https://www.adsbexchange.com/legacy-datafields/).

Explanation with some data? [Here you go](/Supporting_Docs/ata_Fields_Description.md).

These datasets are free of cost, without any need for registration/password, but please be sure to give the appropriate credit.

---

If you are using free-credits provided by your choice of cloud-vendor, please do ensure you shut down your applications & VMs in order to save credits.

More details [here](/Supporting_Docs/Cloudera_Manager_Startup_and_Shutdown.md).

---

<br></br>
Lastly:

- Should you have any feedback/suggestions, I would love to hear them.
- If you'd like to contribute, feel free to submit a pull request.
- If you'd like for me to address a specific topic in further detail, do not hesitate to connect with me.
<br></br>

Originally published on my Medium account [here](https://medium.com/@prathamesh.nimkar/hdfs-commands-79dccfd721d7)