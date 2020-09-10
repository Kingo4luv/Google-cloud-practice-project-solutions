# Implement Private Google Access and Cloud NAT

## **Overview**

In this lab, you implement Private Google Access and Cloud NAT for a VM instance that doesn't have an external IP address.  
Then, you verify access to public IP addresses of Google APIs and services and other connections to the internet.

VM instances without external IP addresses are isolated from external networks. Using Cloud NAT, these instances can access the internet for updates and patches, and in some cases, for bootstrapping.

As a managed service, Cloud NAT provides high availability without user management and intervention.

## **Objectives**

In this lab, you learn how to perform the following tasks:

- Configure a VM instance that doesn't have an external IP address

- Connect to a VM instance using an Identity-Aware Proxy (IAP) tunnel

- Enable Private Google Access on a subnet

- Configure a Cloud NAT gateway

## **Task 1. Create the VM instance**

Create a VPC network with some firewall rules and a VM instance that has no external IP address, and connect to the instance using an IAP tunnel.

### **Create a VPC network and firewall rules**

First, create a VPC network for the VM instance and a firewall rule to allow SSH access.

1. In the Cloud Console, click Activate Cloud Shell (Cloud Shell). If prompted, click Continue.
2. Configure/set your project id with the command:
   > `gcloud config set project [Project-id]`
3. To create a new vpc network, type the following command and wait for it to finish running:

   > `gcloud compute networks create privatenet --subnet-mode=custom --bgp-routing-mode=regional`

4. Enter the following command, to create a new subnet in privatenet VPC:

   > `gcloud compute networks subnets create privatenet-us --range=10.130.0.0/20 --network=privatenet --region=us-central1`

5. Enter the following command to create a firewall rule for ssh access:

   > `gcloud compute firewall-rules create privatenet-allow-ssh --direction=INGRESS --priority=1000 --network=privatenet --action=ALLOW --rules=tcp:22 --source-ranges=35.235.240.0/20` >

   _In order to connect to your private instance using SSH, you need to open an appropriate port on the firewall. IAP connections come from a specific set of IP addresses (35.235.240.0/20). Therefore, you can limit the rule to this CIDR range._

### **Create the VM instance with no public IP address**

1. Enter the following command to create a VM instance with no public address:
   > `gcloud compute instances create vm-internal --zone=us-central1-c --machine-type=n1-standard-1 --subnet=privatenet-us --no-address --image=debian-10-buster-v20200902 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=vm-internal --no-shielded-secure-boot --no-shielded-vtpm`

### SSH to vm-internal to test the IAP tunnel

1. To connect to vm-internal, run the following command:
   > `gcloud compute ssh vm-internal --zone us-central1-c --tunnel-through-iap`
2. If prompted about continuing, type Y.
3. When prompted for a passphrase, press ENTER.
4. When prompted for the same passphrase, press ENTER.
5. To test the external connectivity of vm-internal, run the following command:

   > `ping -c 2 www.google.com`

   _This should not work because vm-internal has no external IP address!_

6. Wait for the ping command to complete.
7. To return to your Cloud Shell instance, run the following command:
   > `exit`

## Task 2. Enable Private Google Access

VM instances that have no external IP addresses can use Private Google Access to reach external IP addresses of Google APIs and services. By default, Private Google Access is disabled on a VPC network.

### Create a Cloud Storage bucket

Create a Cloud Storage bucket to test access to Google APIs and services.

1. Create a cloud storage bucket with the command below:

   > `gsutil mb gs://[project-id]`

   _Note the name of your storage bucket for the next subtask. It will be referred to as [my_bucket]_

### Copy an image from a public Cloud Storage bucket to your own bucket.

1. In Cloud Shell, run the following command, replacing [my_bucket] with your bucket's name:

   > `gsutil cp gs://cloud-training/gcpnet/private/access.svg gs://[my_bucket]`

2. To verify that the image has been copied enter, replacing [my_bucket] with your bucket's name:
   > `gsutil ls gs://[my_bucket]`

## Access the image from your VM instance

1. In Cloud Shell, to try to copy the image from your bucket, run the following command, replacing [my_bucket] with your bucket's name:

   > `gsutil cp gs://[my_bucket]/*.svg .`

   _This should work because Cloud Shell has an external IP address!_

2. To connect to vm-internal, run the following command:

   > `gcloud compute ssh vm-internal --zone us-central1-c --tunnel-through-iap`

   _If prompted, type Y to continue._

3. To try to copy the image to vm-internal, run the following command, replacing [my_bucket] with your bucket's name:

   > `gsutil cp gs://[my_bucket]/*.svg .`

   _Press Ctrl+C to stop the request._

## Enable Private Google Access

Private Google Access is enabled at the subnet level. When it is enabled, instances in the subnet that only have private IP addresses can send traffic to Google APIs and services through the default route (0.0.0.0/0) with a next hop to the default internet gateway.

1. To enable private Google access in privatenet-us subnet network enter:

   > `gcloud compute networks subnets update privatenet-us --region=us-central1 --enable-private-ip-google-access`

2. In Cloud Shell for vm-internal, to try to copy the image to vm-internal, run the following command, replacing [my_bucket] with your bucket's name:

   > `gsutil cp gs://[my_bucket]/*.svg .`

   _This should work because vm-internal's subnet has Private Google Access enabled!_

3. To return to your Cloud Shell instance, run the following command:
   > `exit`

## Task 3. Configure a Cloud NAT gateway

Although vm-internal can now access certain Google APIs and services without an external IP address, the instance cannot access the internet for updates and patches. Configure a Cloud NAT gateway, which allows vm-internal to reach the internet.

### Try to update the VM instances

1. In Cloud Shell, to try to re-synchronize the package index, run the following:

   > `sudo apt-get update`

2. To connect to vm-internal, run the following command:

   > `gcloud compute ssh vm-internal --zone us-central1-c --tunnel-through-iap`

   _If prompted, type Y to continue._

3. To try to re-synchronize the package index of vm-internal, run the following command:

   > ` sudo apt-get update`

   _This should only work for Google Cloud packages because vm-internal only has access to Google APIs and services!_

4. Press Ctrl+C to stop the request and enter.
   > `exit`

## Configure a Cloud NAT gateway

Cloud NAT is a regional resource. You can configure it to allow traffic from all ranges of all subnets in a region, from specific subnets in the region only, or from specific primary and secondary CIDR ranges only.

1. Enter this command to create a nat gateway
   > `gcloud compute routers nats create nat-config --router=nat-router --auto-allocate-nat-external-ips --region=us-central1 --nat-custom-subnet-ip-ranges=privatenet-us`

## Verify the Cloud NAT gateway

It may take up to 3 minutes for the NAT configuration to propagate to the VM, so wait at least a minute before trying to access the internet again.

1. In Cloud Shell for vm-internal, to try to re-synchronize the package index of vm-internal, run the following command:

   > `sudo apt-get update`

2. To return to your Cloud Shell instance, run the following command:

   > `exit`

## Task 4. Configure and view logs with Cloud NAT Logging

Cloud NAT logging allows you to log NAT connections and errors. When Cloud NAT logging is enabled, one log entry can be generated for each of the following scenarios:

- When a network connection using NAT is created.
- When a packet is dropped because no port was available for NAT.
- You can opt to log both kinds of events, or just one or the other. Created logs are sent to Cloud Logging.

### Enabling logging

If logging is enabled, all collected logs are sent to Cloud Logging by default. You can filter these so that only certain logs are sent.

You can also specify these values when you create a NAT gateway or by editing one after it has been created. The following directions show how to enable logging for an existing NAT gateway.

1. Enable logging in the Nat Gateway - To log network address translation events and errors:
   > `gcloud compute routers nats update NAT_GATEWAY --router=ROUTER_NAME --region=REGION --enable-logging`

### Generating logs

As a reminder, Cloud NAT logs are generated for the following sequences:

- When a network connection using NAT is created.
- When a packet is dropped because no port was available for NAT.
- Let's connect the host to the internal VM again to see if any logs are generated.

1. In Cloud Shell for vm-internal, to try to re-synchronize the package index of vm-internal, run the following command:

   > `gcloud compute ssh vm-internal --zone us-central1-c --tunnel-through-iap`

2. If prompted, type Y to continue.

3. Try to re-synchronize the package index of vm-internal by running the following:
   > `sudo apt-get update`
4. To return to your Cloud Shell instance, run the following command:
   > `exit`

## Task 5. Review

You created vm-internal, an instance with no external IP address, and connected to it securely using an IAP tunnel. Then you enabled Private Google Access, configured a NAT gateway, and verified that vm-internal can access Google APIs and services and other public IP addresses.

VM instances without external IP addresses are isolated from external networks. Using Cloud NAT, these instances can access the internet for updates and patches, and in some cases, for bootstrapping. As a managed service, Cloud NAT provides high availability without user management and intervention.

IAP uses your existing project roles and permissions when you connect to VM instances. By default, instance owners are the only users that have the IAP Secured Tunnel User role. If you want to allow other users to access your VMs using IAP tunneling, you need to grant this role to those users.
