
### 1. Identity & Access Management (IAM)
Role created within IAM with the correct policy to allow use of AWS Systems Manager on EC2 Instances.
<img width="374" height="89" alt="image" src="https://github.com/user-attachments/assets/bc96e97b-c72a-4911-925d-ea77f43e9859" />


### 2. Virtual Private Cloud (VPC) 
VPC will house two subnets, two EC2 instances, and the traffic mirror session for passive 
<img width="1010" height="598" alt="image" src="https://github.com/user-attachments/assets/8e7dd64a-4ebf-4d5a-b8b4-847a3d599042" />


### 3. Subnets
Segmented the VPC into two public subnets. They are pubic for ease of testing and configuration without need of VPC endpoints for SSM.
<img width="949" height="172" alt="image" src="https://github.com/user-attachments/assets/0a9133bc-ed29-474b-9366-05c00302d185" />


### 4. Route Table 
Uses ```0.0.0.0/0``` to accept all outbound traffic with a destination to our Internet Gateway (IGW).
<img width="714" height="432" alt="image" src="https://github.com/user-attachments/assets/6e0ee554-d5a3-45f8-936e-1e406e1aaab7" />


### 5. Workload EC2 
Instance launched utilizing Amazon Linux 2023 image on a t3.micro instance. Configured network to utilize our created VPC, the workload subnet, assign a public IP for testing, and a Security Group with no inbound rules.
<img width="1053" height="694" alt="image" src="https://github.com/user-attachments/assets/528fbe14-11a0-49ba-8431-6e3c2bb903e6" />


### 6. Security EC2 
Instance launched using the same configuration as the Workload EC2 instance with adjusted Security Group. Inbound only accepts UDP Port 4789 from the Workload Subnet (10.0.1.0/24).
<img width="1061" height="870" alt="image" src="https://github.com/user-attachments/assets/5171c1ad-2908-4c8d-b397-2d2e45b22597" />


### 7. IAM Role for EC2 Instances
Applied previously created IAM role to both EC2 instances so SSM access is enabled. Left key pairs disabled.
<img width="694" height="202" alt="image" src="https://github.com/user-attachments/assets/0b219a84-0445-405f-a7f7-a4af4c456d35" />


### 8. Traffic Mirror Target
Utilize the Security EC2 network interface ID as the Traffic Mirror Target. This defines where the mirrored packets should be sent.
<img width="416" height="460" alt="image" src="https://github.com/user-attachments/assets/f4177927-5729-4753-ae34-7aeca53fe769" />


### 9. Traffic Mirror Filter 
Determines what traffic is worth capturing. We set all protocols and ```0.0.0.0/0``` to capture everything.
<img width="1418" height="743" alt="image" src="https://github.com/user-attachments/assets/1cd5d289-47c8-4674-91e4-c99469ba0491" />


### 10. Traffic Mirror Session 
Connects the source (Workload EC2) to the target (Security EC2) utilizing the created Traffic Mirror Filter.
<img width="942" height="876" alt="image" src="https://github.com/user-attachments/assets/a92cd4ef-78f9-48ec-8992-d51066a204f8" />


## 11. Architecture layout
<img width="1680" height="1160" alt="Architecture" src="https://github.com/user-attachments/assets/d5cd2e92-06d5-4f5e-a822-52db092927a9" />


### 11. Launch EC2 terminals for Workload and Security
Connect to the Workload and Security EC2 instances using SSM. In the Security EC2 terminal, run ```sudo tcpdump -i ens5 udp port 4789 -nn -vv``` to capture all traffic coming from the Workload EC2 instance.
<img width="776" height="64" alt="image" src="https://github.com/user-attachments/assets/9015ce19-24ca-4ac3-8433-3c42fa794374" />


### 12. Validate Packet Capture
Test on the Workload EC2 terminal regular network activity by running ```curl http://www.google.com```. Confirm on the Security EC2 terminal Workload EC2 network activity is captured.
<img width="858" height="144" alt="image" src="https://github.com/user-attachments/assets/1aa19990-60a8-4e74-b1bd-4f76a53eff7f" />
