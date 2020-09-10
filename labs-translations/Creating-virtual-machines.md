# Creating Virtual Machines

## Overview

In this lab, you will explore the Virtual Machine instance options and create several VMs with different characteristics.

## Objectives

In this lab, you explore the available options for VMs and see the differences between locations.

In this lab, you learn how to perform the following tasks:

- Create several standard VMs

- Create advanced VMs

## Task 1: Create a utility virtual machine

1. In the Cloud Console, click Activate Cloud Shell (Cloud Shell). If prompted, click Continue.
2. Configure/set your project id with the command. Replace [Project-id] with your provided project id:

   > `gcloud config set project [Project-id]`

3. Enter the following command to create a VM instance called utility-vm:
   > `gcloud compute instances create utility-vm --zone=us-central1-c --machine-type=n1-standard-1 --subnet=default --no-address --image=debian-10-buster-v20200902 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=instance-1 --no-shielded-secure-boot`

## Task 2: Create a Windows virtual machine

1. To create a new windows virtual machine, run the following command:

   > `gcloud compute instances create windows-vm --zone=europe-west2-a --machine-type=n1-standard-2 --subnet=default --tags=http-server,https-server --image=windows-server-2016-dc-v20200813 --image-project=windows-cloud --boot-disk-size=100GB --boot-disk-type=pd-ssd --boot-disk-device-name=windows-vm --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any`

2. Run the following command to allow http ingress traffic to the created windows-vm:

   > `gcloud compute firewall-rules create default-allow-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server`

3. Run the following command to allow https ingress traffic to the created windows-vm:
   > `gcloud compute firewall-rules create default-allow-https --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:443 --source-ranges=0.0.0.0/0 --target-tags=https-server`

## Task 3: Create a custom virtual machine

1. To create a new custom virtual machine, run the following command:
   > `gcloud compute instances create custom-vm --zone=us-west1-b --machine-type=custom-6-32768 --subnet=default --network-tier=PREMIUM --image=debian-10-buster-v20200902 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=custom-vm --no-shielded-secure-boot --no-shielded-vtpm --no-shielded-integrity-monitoring --reservation-affinity=any`

## Connect via SSH to your custom VM

1. Run the following command to ssh into custom-vm:

   > `gcloud compute ssh custom-vm`

   _If prompted, type Y to continue._

2. If you are prompted to enter passphrase press enter twice on your keyboard to proceed
3. If you are asked about your zone, make sure you choose where the vm is located
4. To see information about unused and used memory and swap space on your custom VM, run the following command:

   > `free`

5. To see details about the RAM installed on your VM, run the following command:
   > `sudo dmidecode -t 17`
6. To verify the number of processors, run the following command:

   > `nproc`

7. To see details about the CPUs installed on your VM, run the following command:
   > `lscpu`
8. To exit the SSH terminal, run the following command:
   > `exit`

## Task 4: Review

In this lab, you created several virtual machine instances of different types with different characteristics. One was a small utility VM for administration purposes. You also created a standard VM and a custom VM. You launched both Windows and Linux VMs and deleted VMs.
