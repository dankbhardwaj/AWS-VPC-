Amazon VPC (Virtual Private Cloud) - Comprehensive Guide
Table of Contents
VPC Fundamentals

VPC Components & Architecture

Network Connectivity Patterns

Security & Compliance

Advanced VPC Configurations

Troubleshooting & Best Practices

Real-World Scenarios

VPC Fundamentals
1. What is Amazon VPC?
Amazon Virtual Private Cloud (VPC) is a logically isolated section of the AWS cloud where you can launch AWS resources in a virtual network that you define. You have complete control over your virtual networking environment.

Key Characteristics:

Region-specific service (VPCs exist within a single AWS region)

Logical isolation from other virtual networks in AWS

Complete control over IP address ranges, subnets, route tables, and gateways

Ability to configure network gateways and security settings

2. VPC Terminology Deep Dive
CIDR (Classless Inter-Domain Routing):

Method for allocating IP addresses and routing IP packets

Format: x.x.x.x/y where y is the prefix length (0-32)

Example: 10.0.0.0/16 provides 65,536 IP addresses (10.0.0.0 - 10.0.255.255)

Private IP Ranges (RFC 1918):

10.0.0.0/8 (10.0.0.0 - 10.255.255.255)

172.16.0.0/12 (172.16.0.0 - 172.31.255.255)

192.168.0.0/16 (192.168.0.0 - 192.168.255.255)

VPC Components & Architecture
1. VPC Core Components
Subnets
yaml
Definition: A range of IP addresses in your VPC
Types:
  - Public Subnet: Has a route to an Internet Gateway
  - Private Subnet: No direct route to the internet
  - VPN-only Subnet: Route to Virtual Private Gateway

Key Points:
  - Subnets are Availability Zone (AZ) specific
  - Each subnet must reside entirely within one AZ
  - AWS reserves 5 IP addresses in each subnet
Route Tables
yaml
Purpose: Controls traffic routing between subnets and to external networks
Components:
  - Local Route: Automatic route for VPC internal communication
  - Custom Routes: User-defined routes for specific traffic patterns
  - Main Route Table: Default route table for the VPC
  - Custom Route Tables: Associated with specific subnets
Internet Gateway (IGW)
yaml
Function: Horizontally scaled, redundant gateway for internet communication
Characteristics:
  - One IGW can be attached to one VPC
  - Provides NAT for instances with public IPs
  - No availability risks or bandwidth constraints
NAT Gateway
yaml
Purpose: Allows private subnets to connect to the internet while remaining private
Types:
  - NAT Gateway: Managed AWS service (recommended)
  - NAT Instance: Self-managed EC2 instance

NAT Gateway Details:
  - Deployed in a specific AZ
  - Requires EIP (Elastic IP)
  - Supports up to 10 Gbps bandwidth
  - No security groups required
2. Advanced VPC Components
Security Groups vs Network ACLs
sql
Security Groups (Stateful):
  - Operate at instance level
  - Evaluate all rules before deciding
  - Stateful: Return traffic is automatically allowed
  - Can reference other security groups

Network ACLs (Stateless):
  - Operate at subnet level
  - Process rules in order (lowest number first)
  - Stateless: Return traffic must be explicitly allowed
  - Default NACL allows all traffic
VPC Endpoints
yaml
Purpose: Private connection to AWS services without using IGW/NAT
Types:
  - Interface Endpoints (AWS PrivateLink):
    * ENI with private IP in your subnet
    * $ per hour + $ per GB
    * Supports most AWS services
  
  - Gateway Endpoints:
    * Route table target for S3 and DynamoDB
    * Free of charge
    * Only for S3 and DynamoDB
VPC Peering
yaml
Definition: Direct network route between two VPCs using private IP addresses
Characteristics:
  - No transitive peering (A<>B and B<>C doesn't mean A<>C)
  - No overlapping CIDR blocks
  - Cross-region and cross-account peering supported
  - DNS resolution can be enabled between peered VPCs
Network Connectivity Patterns
1. Internet Connectivity Patterns
Public Subnet Architecture
bash
# Route Table for Public Subnet
Destination      Target
10.0.0.0/16      local
0.0.0.0/0        igw-id

# Instance Requirements:
- Must have public IP or EIP
- Security groups must allow required traffic
Private Subnet with NAT Gateway
bash
# Route Table for Private Subnet
Destination      Target
10.0.0.0/16      local
0.0.0.0/0        nat-gateway-id

# NAT Gateway Placement:
- Must be in public subnet
- Requires EIP allocation
2. Hybrid Connectivity Patterns
AWS Site-to-Site VPN
yaml
Components:
  - Virtual Private Gateway (VGW): VPN endpoint on AWS side
  - Customer Gateway (CGW): Public IP of customer router
  - VPN Connection: Encrypted tunnels between VGW and CGW

Characteristics:
  - IPSec VPN tunnels
  - Supports static and dynamic (BGP) routing
  - Typically provides ~1.25 Gbps throughput
AWS Direct Connect
yaml
Purpose: Dedicated network connection from on-premises to AWS
Types:
  - Dedicated Connection: 1/10/100 Gbps physical port
  - Hosted Connection: 50 Mbps - 10 Gbps via AWS partners

Benefits:
  - Consistent network performance
  - Lower latency than internet-based connections
  - Bypasses internet for data transfer
  - Private connectivity to AWS services
3. Multi-VPC Architectures
Transit Gateway
yaml
Purpose: Hub-and-spoke model for connecting multiple VPCs and on-premises networks
Benefits:
  - Simplifies network architecture
  - Supports transitive routing
  - Cross-region peering capability
  - Route tables per attachment for segmentation

Use Cases:
  - Large enterprises with multiple VPCs
  - Shared services architecture
  - Network segmentation and isolation
Security & Compliance
1. VPC Security Best Practices
Network Segmentation
bash
# Recommended CIDR Allocation:
- Management VPC: 10.0.0.0/16
- Production VPC: 10.1.0.0/16
- Development VPC: 10.2.0.0/16
- DMZ VPC: 10.3.0.0/16

# Subnet Design:
- Public subnets: 10.x.1.0/24, 10.x.2.0/24 (per AZ)
- Private subnets: 10.x.10.0/24, 10.x.20.0/24 (per AZ)
- Data subnets: 10.x.100.0/24, 10.x.200.0/24 (per AZ)
Security Group Design
yaml
Web Server SG:
  - Inbound: 80, 443 from 0.0.0.0/0
  - Inbound: 22 from Management CIDR
  - Outbound: All traffic

Application Server SG:
  - Inbound: 8080 from Web Server SG
  - Inbound: 22 from Management CIDR
  - Outbound: 443 to internet (via NAT)

Database SG:
  - Inbound: 3306 from Application Server SG
  - Inbound: 22 from Management CIDR
  - Outbound: None
2. Network Access Control Lists
NACL Best Practices
bash
# Example NACL Rules (Stateless)
Rule# Type    Protocol Port Range Source     Allow/Deny
100   SSH     TCP      22         Mgmt-CIDR  ALLOW
110   HTTP    TCP      80         0.0.0.0/0  ALLOW
120   HTTPS   TCP      443        0.0.0.0/0  ALLOW
*     ALL     ALL      ALL        0.0.0.0/0  DENY

# Ephemeral Ports (Return Traffic)
Rule# Type    Protocol Port Range Source     Allow/Deny
*     Custom  TCP      1024-65535 0.0.0.0/0  ALLOW
Advanced VPC Configurations
1. VPC Flow Logs
Configuration and Analysis
bash
# Enable VPC Flow Logs
aws ec2 create-flow-logs \
    --resource-type VPC \
    --resource-ids vpc-12345678 \
    --traffic-type ALL \
    --log-destination-type cloud-watch-logs \
    --log-group-name VPCFlowLogs

# Flow Log Record Format
version account-id interface-id srcaddr dstaddr srcport dstport protocol packets bytes start end action log-status
Common Flow Log Queries
sql
-- Find rejected traffic
SELECT srcaddr, dstaddr, srcport, dstport, protocol
FROM VPCFlowLogs
WHERE action = 'REJECT'

-- Top talkers by bytes
SELECT srcaddr, SUM(bytes) as total_bytes
FROM VPCFlowLogs
GROUP BY srcaddr
ORDER BY total_bytes DESC
2. DNS Configuration
Route 53 Resolver
yaml
Route 53 Resolver Features:
  - Conditional Forwarding: Forward specific domains to on-premises
  - DNS Firewall: Filter DNS queries based on rules
  - Query Logging: Log all DNS queries for audit

Configuration:
  - Inbound Endpoint: DNS resolution from on-premises to Route 53
  - Outbound Endpoint: DNS resolution from VPC to on-premises
Troubleshooting & Best Practices
1. Common VPC Issues and Solutions
Connectivity Problems
markdown
1. Cannot reach internet from EC2 instance:
   - Check if instance has public IP/EIP
   - Verify route table has 0.0.0.0/0 -> igw-id
   - Check NACL and Security Group rules
   - Verify IGW is attached to VPC

2. Cannot connect between instances in different subnets:
   - Verify they're in same VPC
   - Check route tables for both subnets
   - Verify Security Groups allow traffic
   - Check NACL rules for both subnets

3. Cannot connect to on-premises:
   - Verify VPN/Direct Connect status
   - Check BGP routes (if using dynamic routing)
   - Verify route propagation in route tables
   - Check security group/NACL rules
Performance Issues
markdown
1. Network throughput lower than expected:
   - Check instance type network performance
   - Verify no network congestion in AZ
   - Check for packet loss in VPC Flow Logs
   - Verify NAT Gateway is not bottleneck

2. High latency to AWS services:
   - Use VPC Endpoints for AWS services
   - Consider Direct Connect for consistent performance
   - Check for cross-region traffic
2. VPC Design Best Practices
CIDR Planning
bash
# Large Enterprise CIDR Plan
Region: us-east-1 (10.0.0.0/8)
  - VPC 1 (Production): 10.1.0.0/16
  - VPC 2 (Staging): 10.2.0.0/16
  - VPC 3 (Development): 10.3.0.0/16
  - VPC 4 (Shared Services): 10.4.0.0/16

# Subnet Allocation per VPC (/16 VPC -> /20 subnets)
- Public Subnets: 10.x.0.0/20, 10.x.16.0/20 (per AZ)
- Private Subnets: 10.x.32.0/20, 10.x.48.0/20 (per AZ)
- Data Subnets: 10.x.64.0/20, 10.x.80.0/20 (per AZ)
- Reserved: 10.x.240.0/20 (for future expansion)
High Availability Design
yaml
Multi-AZ Architecture:
  - NAT Gateway in each AZ
  - Application Load Balancer across multiple AZs
  - Database with multi-AZ deployment
  - Route 53 with health checks for DNS failover

Disaster Recovery:
  - VPC in secondary region with similar design
  - RTO/RPO defined for failover scenarios
  - Regular DR drills and testing
Real-World Scenarios
1. Three-Tier Web Application Architecture
Complete VPC Setup
yaml
VPC: 10.0.0.0/16
Availability Zones: us-east-1a, us-east-1b

Subnets:
  Public Subnets:
    - us-east-1a: 10.0.1.0/24
    - us-east-1b: 10.0.2.0/24
  
  Private App Subnets:
    - us-east-1a: 10.0.10.0/24
    - us-east-1b: 10.0.20.0/24
  
  Private Data Subnets:
    - us-east-1a: 10.0.100.0/24
    - us-east-1b: 10.0.200.0/24

NAT Gateways:
  - nat-us-east-1a: in 10.0.1.0/24
  - nat-us-east-1b: in 10.0.2.0/24

Route Tables:
  Public RT (10.0.1.0/24, 10.0.2.0/24):
    - 10.0.0.0/16 -> local
    - 0.0.0.0/0 -> igw-id
  
  Private App RT (10.0.10.0/24, 10.0.20.0/24):
    - 10.0.0.0/16 -> local
    - 0.0.0.0/0 -> nat-gw-1a (for 10.0.10.0/24)
    - 0.0.0.0/0 -> nat-gw-1b (for 10.0.20.0/24)
  
  Private Data RT (10.0.100.0/24, 10.0.200.0/24):
    - 10.0.0.0/16 -> local
    - No internet access
2. Hub-and-Spoke Multi-Account Architecture
Transit Gateway Design
yaml
Transit Gateway: tgw-123456 (10.100.0.0/16)

Attachments:
  - Shared Services VPC (10.1.0.0/16)
  - Production VPC (10.2.0.0/16)
  - Development VPC (10.3.0.0/16)
  - On-premises (via DX/VPN)

Route Tables:
  - Shared-RT: For shared services access
  - Prod-RT: Production isolation
  - Dev-RT: Development isolation
  - Inspection-RT: For traffic inspection

Route Propagation:
  - Each VPC route table has routes to other VPCs via TGW
  - Specific route tables control traffic flow between segments
3. VPC Endpoints for Private Service Access
Service Endpoint Configuration
bash
# S3 Gateway Endpoint
aws ec2 create-vpc-endpoint \
    --vpc-id vpc-123456 \
    --service-name com.amazonaws.us-east-1.s3 \
    --route-table-ids rtb-123456

# SSM Interface Endpoint
aws ec2 create-vpc-endpoint \
    --vpc-id vpc-123456 \
    --service-name com.amazonaws.us-east-1.ssm \
    --vpc-endpoint-type Interface \
    --subnet-ids subnet-123456 subnet-789012 \
    --security-group-ids sg-123456
4. Monitoring and Maintenance
CloudWatch Metrics and Alarms
yaml
Key VPC Metrics to Monitor:
  - NATGatewayBytesIn/Out: Traffic through NAT Gateway
  - VPNTunnelState: VPN connection status (1=up, 0=down)
  - VPCPeeringConnectionRequests: Peering traffic
  - ActiveConnectionCount: For Network Load Balancers

Recommended Alarms:
  - NATGatewayErrorPortAllocation: > 0 for 5 minutes
  - VPNTunnelState: = 0 for 1 minute
  - UnHealthyHostCount: > 0 for ELB/ALB
This comprehensive VPC guide covers everything from basic concepts to advanced enterprise architectures, providing a solid foundation for designing, implementing, and troubleshooting AWS networking solutions.
