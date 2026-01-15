# AWS Agentless Traffic Ingestion
![My Skills](https://skillicons.dev/icons?i=aws,linux)

## Overview
An infrastructure first approach to network security, this project demonstrates the deployment of a secure, agentless packet capture pipeline within AWS. By leveraging the Nitro Hypervisor, the architecture achieves 100% network visibility without the performance overhead or security risks associated with traditional guest-level agents.

## 🏛️Architecture
| System | Role | Subnet | Image | Instance |
|---|---|---|---|---|
| **Workload EC2** | Production Data Source | 10.0.1.0/24 (Workload) | Amazon Linux 2023 (Nitro) | t3.micro
| **Security EC2** | Dedicated Security Sensor | 10.0.2.0/24 (Security) | Amazon Linux 2023 (Nitro) | t3.micro

### Project Design Pillars
* **Operational Integrity:** Out-of-band design ensures zero performance impact on production workloads.
* **Security by Design:** Segmented VPC architecture with zero inbound management ports (No SSH/22).
* **Packet Level Verification:** Validation of HTTP traffic decapsulation via VXLAN.

## 🛠️What I Built
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

## 📖Documentation
For a full breakdown of the setup process  and results with screenshots: [documentation](documentation.md).
<img width="1680" height="1160" alt="Architecture" src="https://github.com/user-attachments/assets/d5cd2e92-06d5-4f5e-a822-52db092927a9" />

## 🛡️Technical Impact & Wins
* **Non-Intrusive Visibility:** 100% packet ingestion with zero impact on production resources.
* **Operational Decoupling:** Out-of-band design ensures that security sensor failures or spikes do not affect production uptime.
* **Zero-Trust Management:** Eliminated the attack surface of traditional management (SSH/22) via SSM integration.
* **Infrastructure-Level Fidelity:** Leveraged Nitro Hypervisor for raw, immutable data that is independent of guest OS limitations.

## 💡Key Learnings & Troubleshooting
* **Modern Interface Naming:** Resolved "No such device" errors by identifying the transition from ```eth0``` to naming ```ens5``` in AL2023.
* **Control Plane Routing:** Debugged SSM connectivity issues by ensuring correct Route Table entries to the Internet Gateway.
* **Encryption Awareness:** Analyzed the difference between plaintext HTTP captures and encrypted HTTPS payloads in a security context.

## 🚀Enhancements
1. **Network Hardening (Private Subnets)**
   * **VPC Endpoints (AWS PrivateLink):** Transition both instances to fully private subnets by deploying interface endpoints for SSM, EC2 Messages, and SSM Messages.
   * **Eliminate Internet Gateways:** Remove the requirement for a public IP and IGW, ensuring that all management traffic stays within the AWS backbone.

2. **Automated Threat Detection**
   * **Lambda Triggered Analysis:** Implement an AWS Lambda function to automatically trigger a ```tcpdump``` capture on the Security Sensor when specific CloudWatch alarms are met.
   * **Centralized Logging:** Intregate Amazon Kinesis to stream capatured packet metadata to an S3 data lake for long-term forensic analysis.

3. **Scalability and Orchestration**
   * **Gateway Load Balancer:** Replace the single security EC2 with a fleet of sensors behind a GWLB to allow for horizontal scaling and high availiabity of ingestion pipeline.
   * **Infrastructure as Code:** Codify the entire mirroring stack using Terraform to allow for repeatable, one click deployments across multiple regions.

---

**Author:** Yoan Oviedo

**Date Completed:** January 12th, 2026

**Focus:** Cloud Infrastructure & Network Security
