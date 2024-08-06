# Lab 3: Networking and Security Groups

## Overview of Networking and Security

In this lab, you will learn about the essential components of AWS networking and security, including setting up a Virtual Private Cloud (VPC) and configuring security groups.

## Setting up a VPC for the Project

**What is a VPC?**

Amazon Virtual Private Cloud (Amazon VPC) lets you provision a logically isolated section of the AWS Cloud where you can launch AWS resources in a virtual network that you define. With VPC, you have complete control over your virtual networking environment, including the selection of your own IP address range, creation of subnets, and configuration of route tables and network gateways.

**Components of a VPC**

1. **Subnets**: A subnet is a range of IP addresses in your VPC. You can launch AWS resources into a specified subnet. There are two types of subnets:

   - **Public Subnets**: Subnets that are associated with a route table that has a route to an Internet Gateway.
   - **Private Subnets**: Subnets that are associated with a route table that does not have a route to an Internet Gateway.

2. **Route Tables**: A route table contains a set of rules, called routes, that are used to determine where network traffic is directed.

3. **Internet Gateway**: An Internet Gateway is a horizontally scaled, redundant, and highly available VPC component that allows communication between instances in your VPC and the internet.

4. **NAT Gateway**: A Network Address Translation (NAT) gateway enables instances in a private subnet to connect to the internet or other AWS services, but prevents the internet from initiating connections with those instances.

## Configuring Security Groups

**What are Security Groups?**

- **Security Groups**: Security groups act as virtual firewalls for your instances to control inbound and outbound traffic. You can specify allowed inbound and outbound traffic at the instance level.
- **Stateful Nature**: Security groups are stateful. This means that if you allow an incoming request from a particular IP address, the response to that request is allowed regardless of outbound rules.

**Best Practices for Configuring Security Groups**

1. **Least Privilege Principle**:

   - Only open necessary ports and IP ranges to minimize the attack surface. For example, limit SSH access (port 22) to a specific IP address range rather than allowing wide-open access.

2. **Use Separate Security Groups for Different Layers**:

   - Assign separate security groups for different application tiers (e.g., web servers, application servers, and database servers). This approach helps to enforce the principle of least privilege and simplifies rule management.

3. **Regularly Review and Audit Security Groups**:

   - Periodically review your security group configurations to ensure they follow your security policies and remove any unnecessary rules.

4. **Implement Logging and Monitoring**:

   - Enable VPC Flow Logs to capture information about the IP traffic going to and from network interfaces in your VPC. Use AWS CloudWatch and AWS CloudTrail to monitor and log changes to your security configurations.

5. **Test Configurations**:
   - Test your security group configurations to ensure they are correctly restricting or allowing traffic as intended. Use tools like AWS Trusted Advisor and third-party security assessment tools.

## Architecture Diagram

**Description:**

The architecture diagram for this lab illustrates the key components and their interactions within a Virtual Private Cloud (VPC).

![Placeholder for Architecture Diagram](placeholder-for-architecture-diagram.png)

**Explanation of the Diagram:**

- **VPC**: The outermost box represents the VPC, encompassing all network components.
- **Public Subnet**: Located within the VPC, connected to the Internet Gateway, and hosting resources like web servers.
- **Private Subnet**: Also within the VPC, connected to the NAT Gateway, and hosting resources like application servers and databases.
- **Route Tables**: Indicating how traffic is routed within the VPC. The public subnet’s route table includes a route to the Internet Gateway, while the private subnet’s route table includes a route to the NAT Gateway.
- **Security Groups**: Represented as boundaries around individual resources, showing the control of traffic at the instance level. These will include rules to allow or deny specific types of traffic to and from the resources within the VPC.

This diagram will help visualize how different components in the VPC interact with each other and how security is enforced at the instance level using security groups.
