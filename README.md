# AWS Agentless Traffic Ingestion

## Overview
An infrastructure first approach to network security, this project demonstrates the deployment of a secure, agentless pakcet capture pipeline within AWS. By leveraging the Nitro Hypervisor, the architecture achieves 100% network visibility without the performance overhead or security risks associated with traditional guest-level agents.

## Lab Architecture
| System | Role | Subnet | Image | Instance |
|---|---|---|---|---|
| **Workload EC2** | Production Data Source | 10.0.1.0/24 (Workload) | Amazon Linux 2023 (Nitro) | t3.micro
| **Security EC2** | Dedicated Security Sensor | 10.0.2.0/24 (Security) | Amazon Linux 2023 (Nitro) | t3.micro

### Project Design Pillars
* **Operational Integrity:** Out-of-band design ensures zero performance impact on production workloads.
* **Security by Design:** Segmented VPC architecture with zero inbound management ports (No SSH/22).
* **Packet Level Verification:** Technical validation of HTTP/NTP traffic decapsulation via VXLAN.

## What I Built
### Phase 1: Establishing the Foundation
* **VPC Segmentation:** Deployed a custom VPC with isolated subnets to decouple monitoring from production.
* **Identity & Access:** Provisioned IAM roles for AWS Systems Manager (SSM), enabling keyless, identity based management.
* **Nitro-Based Provisioning:** Launched t3.micro instances to utilize modern hypervisor level traffic mirroring.

### Phase 2: Ingestion Logic
* **Mirror Targets:** Configured the Security Sensor's ENI as the destination for all intercepted traffic.
* **Traffic Mirroring Session:** Established a secure VXLAN (UDP 4789) tunnel to stream production data out-of-band.
* **Hardened Security Groups:** Workload instance maintains zero inbound rules and sensor instance restricted exclusively to VXLAN traffic.

### Phase 3: Validation & Analysis
* **Traffic Generation:** Executed standard network requests to simulate production activity.
* **Live Capture:** Utilized ```tcpdump``` on the sensor to intercept and inspect the raw data stream.
* **Decapsulation Verification:** Successfully verified application layer details (HTTP/NTP) delivered via the mirror session.

## Documentation
### 1. Identity & Access Management (IAM) Role with correct policy to allow use of AWS Systems Manager
<img width="374" height="89" alt="image" src="https://github.com/user-attachments/assets/bc96e97b-c72a-4911-925d-ea77f43e9859" />

### 2. Create Virtual Private Cloud (VPC) which will house two subnets, two EC2 instances, and the traffic mirror session.
<img width="1010" height="598" alt="image" src="https://github.com/user-attachments/assets/8e7dd64a-4ebf-4d5a-b8b4-847a3d599042" />

### 3. Segmented the VPC into two subnets for Workload and Security monitoring.
<img width="949" height="172" alt="image" src="https://github.com/user-attachments/assets/0a9133bc-ed29-474b-9366-05c00302d185" />

### 4. Route Table uses ```0.0.0.0/0``` to accept all outbound traffic with a destination to our Internet Gateway (IGW).
<img width="714" height="432" alt="image" src="https://github.com/user-attachments/assets/6e0ee554-d5a3-45f8-936e-1e406e1aaab7" />

### 5. Workload EC2 instance launched utilizing Amazon Linux 2023 image on a t3.micro instance. Configured network to utilize our created VPC, the workload subnet, assign a public IP for testing, and a Security Group with no inbound rules.
<img width="1053" height="694" alt="image" src="https://github.com/user-attachments/assets/528fbe14-11a0-49ba-8431-6e3c2bb903e6" />

### 6. Security EC2 instance launched using the same configuration as the Workload EC2 instance with adjusted Security Group. Inbound only accepts UDP Port 4789 from the Workload Subnet (10.0.1.0/24).
<img width="1061" height="870" alt="image" src="https://github.com/user-attachments/assets/5171c1ad-2908-4c8d-b397-2d2e45b22597" />

### 7. Applied previously created IAM role to both EC2 instances so SSM access is enabled. Left key pairs disabled.
<img width="694" height="202" alt="image" src="https://github.com/user-attachments/assets/0b219a84-0445-405f-a7f7-a4af4c456d35" />

### 8. Utilize the Security EC2 network interface ID as the Traffic Mirror Target. This defines where the mirrored packets should be sent.
<img width="416" height="460" alt="image" src="https://github.com/user-attachments/assets/f4177927-5729-4753-ae34-7aeca53fe769" />

### 9. Traffic Mirror Filter that determines what traffic is worth capturing. We set all protocols and ```0.0.0.0/0``` to capture everything.
<img width="1418" height="743" alt="image" src="https://github.com/user-attachments/assets/1cd5d289-47c8-4674-91e4-c99469ba0491" />

### 10. Traffic Mirror Session connects the source (Workload EC2) to the target (Security EC2) utilizing the created Traffic Mirror Filter.
<img width="942" height="876" alt="image" src="https://github.com/user-attachments/assets/a92cd4ef-78f9-48ec-8992-d51066a204f8" />

## 11. Architecture layout.
<img width="1680" height="1160" alt="Architecture" src="https://github.com/user-attachments/assets/d5cd2e92-06d5-4f5e-a822-52db092927a9" />

### 11. Connect to the Workload and Security EC2 instances using SSM. In the Security EC2 terminal, run ```sudo tcpdump -i ens5 udp port 4789 -nn -vv``` to capture all traffic coming from the Workload EC2 instance.
<img width="776" height="64" alt="image" src="https://github.com/user-attachments/assets/9015ce19-24ca-4ac3-8433-3c42fa794374" />

### 12. Test on the Workload EC2 terminal regular network activity by running ```curl http://www.google.com```. Confirm on the Security EC2 terminal Workload EC2 network activity is captured.
<img width="858" height="144" alt="image" src="https://github.com/user-attachments/assets/1aa19990-60a8-4e74-b1bd-4f76a53eff7f" />

## Technical Impact & Wins
* **Non-Intrusive Visibility:** 100% packet ingestion with zero impact on production resources.
* **Operational Decoupling:** Out-of-band design ensures that security sensor failures or spikes do not affect production uptime.
* **Zero-Trust Management:** Eliminated the attack surface of traditional management (SSH/22) via SSM integration.
* **Infrastructure-Level Fidelity:** Leveraged Nitro Hypervisor for raw, immutable data that is independent of guest OS limitations.

## Key Learnings & Troubleshooting
* **Modern Interface Naming:** Resolved "No such device" errors by identifying the transition from ```eth0``` to naming ```ens5``` in AL2023.
* **Control Plane Routing:** Debugged SSM connectivity issues by ensuring correct Route Table entries to the Internet Gateway.
* **Encryption Awareness:** Analyzed the difference between plaintext HTTP captures and encrypted HTTPS payloads in a security context.

## Enhancements
1. **Network Hardening (Private Subnets)**
   * VPC Endpoints (AWS PrivateLink): Transition both instances to fully private subnets by deploying interface endpoints for SSM, EC2 Messages, and SSM Messages.
   * Eliminate Internet Gateways: Remove the requirement for a public IP and IGW, ensuring that all management traffic stays within the AWS backbone.

2. **Automated Threat Detection**
   * Lambda Triggered Analysis: Implement an AWS Lambda function to automatically trigger a ```tcpdump``` capture on the Security Sensor when specific CloudWatch alarms are met.
   * Centralized Logging: Intregate Amazon Kinesis to stream capatured packet metadata to an S3 data lake for long-term forensic analysis.

3. **Scalability and Orchestration**
   * Gateway Load Balancer: Replace the single security EC2 with a fleet of sensors behind a GWLB to allow for horizontal scaling and high availiabity of ingestion pipeline.
   * Infrastructure as Code: Codify the entire mirroring stack using Terraform to allow for repeatable, one click deployments across multiple regions.

---

**Author:** Yoan Oviedo

**Date Completed:** January 12th, 2026

**Focus:** Cloud Infrastructure & Network Security
