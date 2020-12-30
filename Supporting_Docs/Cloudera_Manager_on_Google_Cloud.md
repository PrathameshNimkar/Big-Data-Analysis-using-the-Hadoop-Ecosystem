![Alt Text](/Images/Cloudera_Manager_on_Google_Cloud_Platform.png?raw=true "Cloudera Manager on GCP")
<br></br>

# <b>Cloudera Manager on Google Cloud Platform</b>

<b>Part 1</b>:
- Creating a cluster with 4 nodes on GCP
- [Installing JAVA](/Supporting_Docs/Installing_JAVA.md)
- [Firewall Configurations](/Supporting_Docs/Firewall_Configurations.md)
- Installing Cloudera Manager

<b>[Part 2](/Supporting_Docs/Cloudera_Manager_on_Google_Cloud-2.md)</b>:
- Cloudera Manager — Cluster Installation
- Cloudera Manager — Cluster Configuration

---
<br></br>

<b>Assumptions</b>:
- You have a Google Cloud account. If not, [click here to create a free-tier Google Cloud account](https://cloud.google.com/free). This will give you USD 300 of free credit
- This post is on a manual installation of Cloudera Manager without Google’s Dataproc functionality

---
<br></br>
## <b>Creating a cluster with 4 nodes on GCP</b>

![Alt Text](/Images/GCP_Console.png?raw=true "GCP Console")

Once you create a Google Cloud account. Navigate to the [console](https://console.cloud.google.com/) and hit the drop-down for “Select a project”.

Now on the top-right, hit “NEW PROJECT”. Add a “project name” and click save. Leave “organization” as-is.

![Alt Text](/Images/Compute_Engine.png?raw=true "Compute Engine")

From the navigation menu on the left, select Compute Engine -> VM Instances as shown. Create a new VM Instance.

![Alt Text](/Images/VM_Instance_Config.png?raw=true "VM Instance Config")

Add a generic name for the instance. I generally do instance-1 or instance-001 and continue the numbers consecutively

Select “us-central1 (Iowa)” region with the “us-central1-a” zone. This seems to be the cheapest option available

The “n1” series of general-purpose machine type is the cheapest option

Under machine type, select “Custom” with 2 cores of vCPU and 12 GB of RAM. Please note there is a limit to the number of cores and total RAMs provided under the free-tier usage policy

<br></br>

![Alt Text](/Images/Boot_Disk.png?raw=true "Boot Disk")

Under “Boot disk”, select Centos OS 7 as the OS and 100 GB as storage

![Alt Text](/Images/Security_and_Networking.png?raw=true "Security and Networking")

Under Identity and API access, leave the access scopes as-is.

Under Firewall, select both boxes to enable HTTP and HTTPS traffic.

Repeat the steps above to create 4 nodes each with the same configuration.

![Alt Text](/Images/ssh_access.png?raw=true "ssh Access")

In SSH drop-down, select “open in browser window”. Repeat for all nodes. Enter the commands:

```
sudo su -
vi /etc/selinux/config
```

![Alt Text](/Images/selinux_disabled.png?raw=true "selinux disabled")

Inside the config file, change SELINUX=disabled

```
vi /etc/ssh/sshd_config
```

![Alt Text](/Images/permit_root_login.png?raw=true "permit root login")

Under Authentication, change

```
PermitRootLogin yes
```

Now we can login into instance-2/3/4 from instance-1 without password

![Alt Text](/Images/reboot.png?raw=true "reboot")

Ensure that you’ve done the above steps on all nodes. Following which you should reboot all the 4 nodes

---
<br></br>

![Alt Text](/Images/Generating_Key_Pairs.png?raw=true "Generating Key Pairs")

Re-login into instance-1 as root user and enter:

```
ssh-keygen
hit enter three times
```

and your keys will be generated under /root/.ssh/

![Alt Text](/Images/sshkeys.png?raw=true "sshkeys")

```
cd /root/.ssh
cat id_rsa.pub
```

In instance-1, as root user, copy the public key

![Alt Text](/Images/paste_sshkeys.png?raw=true "paste sshkeys")

In cloud console menu, metadata -> sshkeys -> edit -> add item -> enter key and save

```
# Now, in the terminal, on all nodes:
service sshd restart
```

![Alt Text](/Images/ssh1-2.png?raw=true "ssh1-2")

```
ssh instance-2
# “yes” to establish connection
```

From instance-1, check connectivity to instance 2. Then, repeat for other instances.

Cluster setup is completed for 4 nodes on Google Cloud Platform

---
<br></br>

## <b>Installing JAVA</b>

In order to install Java, please visit this [link](/Supporting_Docs/Installing_JAVA.md).

Java is installed on all 4 nodes on Google Cloud Platform.

---
<br></br>

## <b>Installing Cloudera Manager</b>

![Alt Text](/Images/CM_Downloads.png?raw=true "CM Downloads")

Head over to [Cloudera Manager Downloads page](https://www.cloudera.com/downloads/manager/6-3-1.html), enter your details and hit download.

![Alt Text](/Images/CM_Installation-1.png?raw=true "CM_Installation-1")

```
# To install CM, change permissions and run the installer.bin

wget <and paste it here>
chmod u+x cloudera-manager-installer.bin
sudo ./cloudera-manager-installer.bin
```

![Alt Text](/Images/CM_Installation-2.png?raw=true "CM_Installation-2")

This window will now open, hit Next and accept all licenses

![Alt Text](/Images/CM_Installation-3.png?raw=true "CM_Installation-3")

This launches the Cloudera Manager Login Page. Use admin/admin as credentials

![Alt Text](/Images/CM_Installation-4.png?raw=true "CM_Installation-4")

Here’s the Cloudera Manager Homepage

![Alt Text](/Images/CM_Installation-5.png?raw=true "CM_Installation-5")

Accept the licenses

![Alt Text](/Images/CM_Installation-6.png?raw=true "CM_Installation-6")

There are other options, but this 60 day Enterprise-trial period seems to be the best option

---
<br></br>

## <b>Firewall Configurations</b>

Please review the [link](/Supporting_Docs/Firewall_Configurations.md) to configure the required firewall configurations

---
<br></br>

This completes part 1 of the Cloudera Manager Installation on Google Cloud Platform. For part 2, please visit this [link](/Supporting_Docs/Cloudera_Manager_on_Google_Cloud-2.md)

---

<br></br>
Lastly:

- Should you have any feedback/suggestions, I would love to hear them.
- If you'd like to contribute, feel free to submit a pull request.
- If you'd like for me to address a specific topic in further detail, do not hesitate to connect with me.
<br></br>

Originally published on my Medium account [here](https://medium.com/@prathamesh.nimkar/cloudera-manager-on-google-cloud-3da9b4d64d74)