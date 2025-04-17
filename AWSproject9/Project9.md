# **The Challenge**

## Objective: To configure our environment to meet Vendor A's security requirements and enable our Kubernetes services to access their resources through the VPN connection while adhering to their private IP whitelisting constraint.

## Step-by-step Implementation

- Assuming Vendor A provides a private CIDR block of 192.0.2.0/24 and requires our traffic to originate from the private IP 10.0.1.10"
- How traffic flows from K8s pods → NAT instance → VPN Tunnel → Vendor A

## _This architecture solves the challenge by forcing all traffic from Kubernetes services (which reside in a private subnet) to route through a specific NAT instance that has a private IP whitelisted by Vendor A. That NAT instance then forwards traffic over a site-to-site VPN tunnel to Vendor A._

## Traffic Flow (Step-by-Step)

1. K8s Pods (Private Subnet)

- Your workloads (applications/services) are running inside a Kubernetes cluster, and the nodes are in a private subnet—they can’t access the internet or other networks directly.

2. Route Table

- You update the private subnet’s route table to send all egress traffic destined for Vendor A’s CIDR block to a NAT instance instead of the usual NAT Gateway.

3. NAT Instance (Custom EC2)

- This EC2 instance acts like a traditional NAT device.

- It's configured to forward traffic, and its source IP is the one Vendor A has whitelisted.

- You enable IP forwarding on it and configure iptables/NAT rules to masquerade outbound traffic.

4. VPN Tunnel

- The NAT instance is connected to a site-to-site VPN, which is already established between your AWS environment and Vendor A’s on-prem/VPC setup.
- This tunnel ensures encrypted traffic, meeting Vendor A's secure communication requirement.

5. Vendor A

- From Vendor A’s point of view, all traffic appears to come from one known private IP—the NAT instance.
- They grant access only to that IP, completing the security model.

The architecture ensures compliance with Vendor A's strict access controls by introducing a NAT instance with a known IP address, tunneled through a secure VPN.

1. All Kubernetes workloads, deployed in private subnets, route traffic destined for Vendor A through a specially configured NAT instance.

2. The NAT instance uses IP masquerading (SNAT) to replace the pod’s source IP with its own.

3. The NAT instance is connected to an AWS Site-to-Site VPN, ensuring encrypted communication.

4. Vendor A only receives traffic from the whitelisted IP, and is unaware of the originating pod or service behind the NAT.

### This design satisfies:

- Security: Traffic is encrypted (VPN).

- IP Restriction: A single IP is exposed to Vendor A.

- Isolation: Kubernetes workloads remain in a private subnet.
