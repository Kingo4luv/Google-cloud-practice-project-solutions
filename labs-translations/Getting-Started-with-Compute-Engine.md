# **Google Cloud Fundamentals: Getting Started with Compute Engine**

## Overview

In this lab, you will create virtual machines (VMs) and connect to them. You will also create connections between the instances.

## Objectives

In this lab, you will learn how to perform the following tasks:

- Create the first Compute Engine virtual machine using the gcloud command-line interface.

- Create the second compute Engine virtual machine using the gcloud command-line interface.

- Connect between the two instances.

## Task 1: Create a virtual machine using the GCP Console

1. In GCP console, on the top right toolbar, click the Open Cloud Shell button.
2. Click Continue to proceed to the cloud shell.
3. Configure/set your project id with the command. Replace [Project-id] with your provided project id:

   > `gcloud config set project [Project-id]`

4. Enter the command below to set your default zone:

   > `gcloud config set compute/zone <zone-name>`

   replace `<zone-name>` with the zone name zone assigned by Qwiklabs

5. To create a VM instance called my-vm-1 in that zone, execute this command:

   > `gcloud compute instances create "my-vm-1" --machine-type "n1-standard-1" --image-project "debian-cloud" --image "debian-9-stretch-v20190213" --subnet "default"`

   > `gcloud compute firewall-rules create default-allow-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server`

6. Verify that Compute Engine created the VM:
   > `gcloud compute instances describe my-vm-1`

**Note:** _The VM can take about two minutes to launch and be fully available for use._

## Task 2: Create the second virtual machine using the gcloud command line

1. To display a list of all the zones in the region to which Qwiklabs assigned you, enter this partial command gcloud compute zones list | grep followed by the region that Qwiklabs or your instructor assigned you to.

   Your completed command will look like this:

   > `gcloud compute zones list | grep us-central1`

2. Choose a zone from that list other than the zone to which Qwiklabs assigned you. For example, if Qwiklabs assigned you to region us-central1 and zone us-central1-a you might choose zone us-central1-b

3. To set your default zone to the one you just chose, enter this partial command gcloud config set compute/zone followed by the zone you chose.

   Your completed command will look like this:

   > `gcloud config set compute/zone us-central1-b`

4. To create a VM instance called my-vm-2 in that zone, execute this command:

   > `gcloud compute instances create "my-vm-2" --machine-type "n1-standard-1" --image-project "debian-cloud" --image "debian-9-stretch-v20190213" --subnet "default"`

   **Note:** _The VM can take about two minutes to launch and be fully available for use._

## Task 4: Connect between VM instances

1. From cloud shell ssh into my-vm-2 using the command below:

   > `gcloud compute ssh my-vm-2`

   _If prompted, type Y to continue._

2. If you are prompted to enter passphrase press enter twice on your keyboard to proceed
3. If you are asked about your zone, make sure you choose where the vm is located

4. Use the ping command to confirm that my-vm-2 can reach my-vm-1 over the network:
   > `ping -c 4 my-vm-1`
5. Press Ctrl+C to abort the ping command
6. Use the ssh command to open a command prompt on my-vm-1:

   > `ssh my-vm-1`

   If you are prompted about whether you want to continue connecting to a host with unknown authenticity, enter yes to confirm that you do.

7. At the command prompt on my-vm-1, install the Nginx web server:

   > `sudo apt-get install nginx-light -y`

8. Use the nano text editor to add a custom message to the home page of the web server:

   > `sudo nano /var/www/html/index.nginx-debian.html`

9. Use the arrow keys to move the cursor to the line just below the h1 header. Add text like this, and replace YOUR_NAME with your name:

   > `Hi from YOUR_NAME`

10. Press Ctrl+O and then press Enter to save your edited file, and then press Ctrl+X to exit the nano text editor.

    > `curl http://localhost/`

    The response will be the HTML source of the web server's home page, including your line of custom text.

11. To exit the command prompt on my-vm-1, execute this command:

    > `exit`

    You will return to the command prompt on my-vm-2

12. To confirm that my-vm-2 can reach the web server on my-vm-1, at the command prompt on my-vm-2, execute this command:

    > `curl http://my-vm-1/`

    The response will again be the HTML source of the web server's home page, including your line of custom text.

13. To get the external IP of the my-vm-1 instance, enter the command below:

    > `gcloud compute instances list`

14. Copy the external IP of my-vm-1 instance and paste it on the broweser new tab address bar. You will see your web server's home page, including your custom text.

## Congratulations!

In this lab, you created virtual machine (VM) instances in two different zones and connected to them using ping, ssh, and HTTP.
