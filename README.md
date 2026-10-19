# AWS VPC Peering: Connecting Two Isolated Networks

Two VPCs, each with its own public subnet, EC2 instance, security group, network ACL, route table, and internet gateway, connected privately through a **VPC peering connection** and monitored with **VPC Flow Logs** sent to **CloudWatch**.

## Architecture

![Architecture diagram](./architecture.png)

> Analogy: each VPC is a separate office building with its own front door (internet gateway). VPC peering is a private hallway between the two buildings, so staff can walk across without going outside.

## What I Built

| Component | VPC 1 | VPC 2 |
|---|---|---|
| Public subnet | Public Subnet 1 | Public Subnet 2 |
| EC2 instance | Instance in VPC 1 | Instance in VPC 2 |
| Security group | Security Group 1 | Security Group 2 |
| Network ACL | Network ACL 1 | Network ACL 2 |
| Route table | Route table 1 | Route table 2 |
| Internet gateway | Internet Gateway 1 | Internet Gateway 2 |

Shared pieces:
- **VPC Peering Connection** linking VPC 1 and VPC 2
- **VPC Flow Logs** on VPC 1, delivered to **CloudWatch Logs**

## How Traffic Flows

1. A client reaches each instance through its own internet gateway.
2. Each VPC's route table sends traffic for the *other* VPC's CIDR to the peering connection.
3. The network ACL (subnet level) and security group (instance level) must both allow the traffic.
4. Flow Logs record accepted and rejected traffic to CloudWatch for review.

## Step-by-Step Setup

1. **Create VPC 1 and VPC 2** with non-overlapping CIDR blocks (peering fails if they overlap).
2. **Create a public subnet** in each VPC.
3. **Attach an internet gateway** to each VPC and add a `0.0.0.0/0` route to it in each route table.
4. **Launch an EC2 instance** in each public subnet with its own security group.
5. **Create the peering connection** (requester: VPC 1, accepter: VPC 2) and **accept** the request.
6. **Update both route tables** so each points to the other VPC's CIDR via the peering connection.
7. **Update security groups and network ACLs** to allow traffic (for example ICMP or SSH) from the peer VPC's CIDR.
8. **Enable VPC Flow Logs** on VPC 1 with a CloudWatch log group as the destination.
9. **Test:** SSH into the instance in VPC 1, then ping or SSH to the **private IP** of the instance in VPC 2.

## Verification

```bash
# From the instance in VPC 1
ping <private-ip-of-instance-in-vpc-2>
```

Successful replies over the private IP confirm the peering and routing work. Check the CloudWatch log group to see the matching `ACCEPT` entries in the flow logs.

## Key Concepts Learned

- **VPC peering is not transitive** and needs non-overlapping CIDRs.
- **Routes on both sides are required.** The peering connection alone does nothing without them.
- **Security groups are stateful; network ACLs are stateless** (allow rules are needed in both directions).
- **Flow Logs** give visibility into allowed and denied traffic for troubleshooting and security.

## Troubleshooting Notes

| Symptom | Likely cause |
|---|---|
| Ping times out | Missing route to the peer CIDR, or security group/NACL blocks ICMP |
| Peering stuck in `pending-acceptance` | Request not accepted by the accepter VPC |
| Can't create peering | Overlapping CIDR blocks |
| No logs in CloudWatch | Flow Log IAM role or log group misconfigured |

## Clean Up

To avoid charges, delete in this order: EC2 instances, Flow Logs and log group, peering connection, internet gateways, subnets, and finally the VPCs.

## Tech Stack

AWS VPC, EC2, Internet Gateway, Route Tables, Network ACLs, Security Groups, VPC Peering, VPC Flow Logs, CloudWatch

## Author

**Mostafa Tarek**, IT Specialist | Networking, Virtualization, Cloud
Riyadh, Saudi Arabia
[LinkedIn](https://www.linkedin.com/in/your-profile) · [GitHub](https://github.com/your-username)
