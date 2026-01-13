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
### Phase 1: Environment Foundation
* **VPC Segmentation:** Deployed a custom VPC with isolated subnets to decouple monitoring from production.
* **Identity & Access:** Provisioned IAM roles for AWS Systems Manager (SSM), enabling keyless, identity based management.
* **Nitro-Based Provisioning:** Launched t3.micro instances to utilize modern hypervisor level traffic mirroring.

### Phase 2: The Network Tap (Ingestion)
* **Mirror Targets:** Configured the Security Sensor's ENI as the destination for all intercepted traffic.
* **Traffic Mirroring Session:** Established a secure VXLAN (UDP 4789) tunnel to stream production data out-of-band.
* **Hardened Security Groups:** Workload instance maintains zero inbound rules; Sensor instance restricted exclusively to VXLAN traffic.

### Phase 3: Validation & Analysis
* **Traffic Generation:** Executed standard network requests to simulate production activity.
* **Live Capture:** Utilized ```tcpdump``` on the sensor to intercept and inspect the raw data stream.
* **Decapsulation Verification:** Successfully verified application-layer details (HTTP/NTP) delivered via the mirror session.
