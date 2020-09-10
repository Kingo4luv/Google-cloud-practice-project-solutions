# Virtual Private Networks (VPN)

## Overview

In this lab, you establish VPN tunnels between two networks in separate regions such that a VM in one network can ping a VM in the other network over its internal IP address.

## Objectives

In this lab, you learn how to perform the following tasks:

- Create VPN gateways in each network

- Create VPN tunnels between the gateways

- Verify VPN connectivity

## Task 1: Explore the networks and instances

Two custom networks with VM instances have been configured for you. For the purposes of the lab, both networks are VPC networks within a Google Cloud project. However, in a real-world application, one of these networks might be in a different Google Cloud project, on-premises, or in a different cloud.

### Explore the networks

Verify that vpn-network-1 and vpn-network-2 have been created with subnets in separate regions.

1. In GCP console, on the top right toolbar, click the Open Cloud Shell button.
2. Click Continue to proceed to the cloud shell.
3. Enter the command below:

   > `gcloud compute networks subnets list`

   - Note the vpn-network-1 network and its subnet-a in us-central1.

   - Note the vpn-network-2 network and its subnet-b in europe-west1.

### Explore the firewall rules

1. Enter the command below:

   > `gcloud compute firewall-rules list`

   - Note the network-1-allow-ssh and network-1-allow-icmp rules for vpn-network-1.
   - Note the network-2-allow-ssh and network-2-allow-icmp rules for vpn-network-2.

   _These firewall rules allow SSH and ICMP traffic from anywhere._

## Explore the instances and their connectivity

Currently, the VPN connection between the two networks is not established. Explore the connectivity options between the instances in the networks.

1. Enter the command below:
   > `gcloud compute instances list`
2. Note the external and internal IP addresses for server-2.
3. SSH into server-1 with the command below:

   > `gcloud compute ssh server-1`

   If you are asked about zone make sure the zone is the same with the listed server-1 zone

4. To test connectivity to server-2's external IP address, run the following command, replacing server-2's external IP address with the value noted earlier:

   > `ping -c 3 <Enter server-2's external IP address here>`

   _This works because the VM instances can communicate over the internet._

5. To test connectivity to server-2's internal IP address, run the following command, replacing server-2's internal IP address with the value noted earlier:

   > `ping -c 3 <Enter server-2's internal IP address here>`

   _You should see 100% packet loss when pinging the internal IP address because you don't have VPN connectivity yet._

6. Exit the Server-1 SSH terminal with:

   > `exit`

   _You will be returned to the cloud shell terminal_

   **Let's try the same from server-2.**

7. Note the external and internal IP addresses for server-1 by running the command:

   > `gcloud compute instances list`

8. SSH into server-2 with the command below:

   > `gcloud compute ssh server-2`

   If you are asked about zone make sure the zone is the same with the listed server-1 zone

9. To test connectivity to server-1's external IP address, run the following command, replacing server-1's external IP address with the value noted earlier:

   > `ping -c 3 <Enter server-1's external IP address here>`

10. To test connectivity to server-1's internal IP address, run the following command, replacing server-1's internal IP address with the value noted earlier:

    > `ping -c 3 <Enter server-1's internal IP address here>`

    _You should see similar results._

11. Exit the Server-1 SSH terminal with:

    > `exit`

    _You will be returned to the cloud shell terminal_

## Task 2: Create the VPN gateways and tunnels

Establish private communication between the two VM instances by creating VPN gateways and tunnels between the two networks.

### **Reserve two static IP addresses**

Reserve one static IP address for each VPN gateway.

1. Enter the command below to reserve a static ip named vpn-1-static-ip:

   > `gcloud compute addresses create vpn-1-static-ip --region=us-central1`

2. Enter the command below to reserve a static ip named vpn-2-static-ip:

   > `gcloud compute addresses create vpn-2-static-ip --region=europe-west1`

   _Note both IP addresses for the next step. They will be referred to us [VPN-1-STATIC-IP] and [VPN-2-STATIC-IP]_

## Create the vpn-1 gateway and tunnel1to2

1. Enter the commands below to create a VPN-1 gateway and tunnel:

   > `gcloud compute --project <gcp-project-id> target-vpn-gateways create "vpn-1" --region "us-central1" --network "vpn-network-1"`

   > `gcloud compute --project <gcp-project-id> forwarding-rules create "vpn-1-rule-esp" --region "us-central1" --address "[vpn-1-static-ip]" --ip-protocol "ESP" --target-vpn-gateway "vpn-1"`

   > `gcloud compute --project <gcp-project-id> forwarding-rules create "vpn-1-rule-udp500" --region "us-central1" --address "[vpn-1-static-ip]" --ip-protocol "UDP" --ports "500" --target-vpn-gateway "vpn-1"`

   > `gcloud compute --project <gcp-project-id> forwarding-rules create "vpn-1-rule-udp4500" --region "us-central1" --address "[vpn-1-static-ip]" --ip-protocol "UDP" --ports "4500" --target-vpn-gateway "vpn-1"`

   > `gcloud compute --project <gcp-project-id> vpn-tunnels create "tunnel1to2" --region "us-central1" --peer-address "[vpn-2-static-ip]" --shared-secret "gcprocks" --ike-version "2" --local-traffic-selector "0.0.0.0/0" --target-vpn-gateway "vpn-1"`

   > `gcloud compute --project <gcp-project-id> routes create "tunnel1to2-route-1" --network "vpn-network-1" --next-hop-vpn-tunnel "tunnel1to2" --next-hop-vpn-tunnel-region "us-central1" --destination-range "10.1.3.0/24"`

   _Make sure to replace [VPN-2-STATIC-IP] with your reserved IP address for europe-west1 and [VPN-1-STATIC-IP] for us-central1 and <gcp-project-id> with gcp id provided by Qwiklabs_

## Create the vpn-2 gateway and tunnel2to1

1. Enter the commands below to create a VPN-2 gateway and tunnel:

   > `gcloud compute --project "<gcp-project-id>" target-vpn-gateways create "vpn-2" --region "europe-west1" --network "vpn-network-2"`

   > `gcloud compute --project "<gcp-project-id>" forwarding-rules create "vpn-2-rule-esp" --region "europe-west1" --address "[vpn-2-static-ip]" --ip-protocol "ESP" --target-vpn-gateway "vpn-2"`

   > `gcloud compute --project "<gcp-project-id>" forwarding-rules create "vpn-2-rule-udp500" --region "europe-west1" --address "[vpn-2-static-ip]" --ip-protocol "UDP" --ports "500" --target-vpn-gateway "vpn-2"`

   > `gcloud compute --project "<gcp-project-id>" forwarding-rules create "vpn-2-rule-udp4500" --region "europe-west1" --address "[vpn-2-static-ip]" --ip-protocol "UDP" --ports "4500" --target-vpn-gateway "vpn-2"`

   > `gcloud compute --project "<gcp-project-id>" vpn-tunnels create "tunnel2to1" --region "europe-west1" --peer-address "[vpn-1-static-ip]" --shared-secret "gcprocks" --ike-version "2" --local-traffic-selector "0.0.0.0/0" --target-vpn-gateway "vpn-2"`

   > `gcloud compute --project "<gcp-project-id>" routes create "tunnel2to1-route-1" --network "vpn-network-2" --next-hop-vpn-tunnel "tunnel2to1" --next-hop-vpn-tunnel-region "europe-west1" --destination-range "10.5.4.0/24"`

   _Make sure to replace [VPN-1-STATIC-IP] with your reserved IP address for us-central1 and [VPN-2-STATIC-IP] for europe-west1 and <gcp-project-id> with gcp id provided by Qwiklabs_

   _Wait for the VPN tunnels status to change to Established for both tunnels before continuing_

## Task 3: Verify VPN connectivity

### Verify server-1 to server-2 connectivity

1. Enter the command below:
   > `gcloud compute instances list`
2. Note the external and internal IP addresses for server-2.
3. SSH into server-1 with the command below:

   > `gcloud compute ssh server-1`

   If you are asked about zone make sure the zone is the same with the listed server-1 zone

4. To test connectivity to server-2's internal IP address, run the following command, replacing server-2's external IP address with the value noted earlier:

   > `ping -c 3 <Enter server-2's internal IP address here>`

5. Exit the Server-1 SSH terminal with:

   > `exit`

   _You will be returned to the cloud shell terminal_

   **Let's try the same from server-2.**

6. Note the external and internal IP addresses for server-1 by running the command:

   > `gcloud compute instances list`

7. SSH into server-2 with the command below:

   > `gcloud compute ssh server-2`

   If you are asked about zone make sure the zone is the same with the listed server-1 zone

8. To test connectivity to server-1's external IP address, run the following command, replacing server-1's internal IP address with the value noted earlier:

   > `ping -c 3 <Enter server-1's internal IP address here>`

9. To test connectivity to server-1's internal IP address, run the following command, replacing server-1's internal IP address with the value noted earlier:

   > `ping -c 3 <Enter server-1's internal IP address here>`

10. Exit the Server-1 SSH terminal with:

    > `exit`

    _You will be returned to the cloud shell terminal_

## Task 4: Review

In this lab, you configured a VPN connection between two networks with subnets in different regions. Then you verified the VPN connection by pinging VMs in different networks using their internal IP addresses.

You configured the VPN gateways and tunnels using the Cloud Console. However, this approach obfuscated the creation of forwarding rules, which you explored with the command line button in the Console. This can help in troubleshooting a configuration.
