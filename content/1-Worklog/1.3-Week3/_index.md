---
title: "Week 3 Worklog"
date: 2026-06-15
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

{{% notice tip %}}
The third week focused on learning AWS networking infrastructure, including Amazon VPC, Subnets, Internet Gateway, and Security Groups. Hands-on practice was also conducted to build a basic network environment and understand how AWS resources can communicate with each other.
{{% /notice %}}

## Week 3 Objectives

- Understand the basic networking architecture on AWS.
- Learn about the components of Amazon VPC.
- Differentiate between Public Subnet and Private Subnet.
- Practice creating and configuring a simple VPC.
- Understand the role of Security Groups in access control.

## Weekly Tasks

| Day | Tasks | Start Date | Completion Date | Reference |
| --- | --- | --- | --- | --- |
| Mon | - Learn about Amazon VPC.<br>- Understand CIDR Blocks and IP Addresses.<br>- Study the basic AWS networking architecture. | 04/08/2026 | 04/08/2026 | https://docs.aws.amazon.com/vpc/ |
| Tue | - Learn about Subnets, Route Tables, and Internet Gateways.<br>- Differentiate between Public Subnets and Private Subnets. | 05/08/2026 | 05/08/2026 | https://docs.aws.amazon.com/vpc/latest/userguide/ |
| Wed | - Practice creating a VPC.<br>- Create a Public Subnet.<br>- Attach an Internet Gateway and configure Route Tables. | 06/08/2026 | 06/08/2026 | AWS Management Console |
| Thu | - Learn about Security Groups and Network ACLs.<br>- Configure Security Groups for EC2 Instances.<br>- Test SSH connectivity. | 07/08/2026 | 07/08/2026 | https://docs.aws.amazon.com/ec2/ |
| Fri | - Review all networking concepts.<br>- Launch an EC2 Instance inside the configured VPC and verify accessibility from the Internet. | 08/08/2026 | 08/08/2026 | AWS Documentation |

## Week 3 Outcomes

By the end of the third week, a fundamental understanding of AWS networking infrastructure was gained, along with how AWS resources communicate with each other.

The key achievements include:

- Understood the role of Amazon VPC in building private networks on AWS.
- Understood the concept of CIDR Blocks and how IP addresses are allocated.
- Differentiated between:
  - Public Subnet.
  - Private Subnet.
  - Internet Gateway.
  - Route Table.

- Successfully created:
  - An Amazon VPC.
  - Public Subnet.
  - Internet Gateway.
  - Route Table.

- Understood how Security Groups control inbound and outbound traffic for EC2 Instances.

- Configured Security Groups to:
  - Allow SSH connections.
  - Allow HTTP access when required.
  - Verify connectivity from a local machine to an EC2 Instance.

- Understood the relationship between VPC, Subnet, Route Table, and Internet Gateway when building a basic networking environment on AWS.

- Prepared networking knowledge required for deploying AWS services and applications in the following weeks.