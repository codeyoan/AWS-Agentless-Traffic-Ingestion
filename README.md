# AWS Agentless Traffic Ingestion

# Overview
An infrastructure first approach to network security, this project demonstrates the deployment of a secure, agentless pakcet capture pipeline within AWS. By leveraging the Nitro Hypervisor, the architecture achieves 100% network visibility without the performance overhead or security risks associated with traditional guest-level agents.

# Lab Architecture
| System | Role | Subnet | Image | Instance |
| **Workload EC2** | Production Data Source | 10.0.1.0/24 (Workload) | Amazon Linux 2023 (Nitro) | t3.micro
| **Security EC2** | Dedicated Security Sensor | 10.0.2.0/24 (Security) | Amazon Linux 2023 (Nitro) | t3.micro
