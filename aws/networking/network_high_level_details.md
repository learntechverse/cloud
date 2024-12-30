<style type="text/css">
    img {
    display: block;
    margin-left: auto;
    margin-right: auto;
    }
</style>

# AWS Infrastructure High Level Details

## Region
**Definition:** A geographical area in which AWS has data centers. Each region is completely independent and isolated from the others.

As of December 2023, AWS has `32 Launched Regions` each with multiple Availability Zones (AZs).

Please lookout and read about [AWS Global Infrastructure](https://aws.amazon.com/about-aws/global-infrastructure/).

**Key Points:**
  - Regions consist of multiple Availability Zones.
  - Users can select a specific region to host their AWS resources.
  - Services can also be region specific. (not all services available acrtoss all regions). You can read more about it [here](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/)


## Availibility Zone (AZ)
**Definition:** A data center or a group of data centers within a region. Each Availability Zone is isolated but connected to other zones within the same region.

As of December 2023, AWS has 102 Availability Zones.

**Key Points:**
  - Designed to be independent, with its own power, cooling, and networking.
  - Multiple AZs in a region provide high availability and fault tolerance.
  

## Route 53 (R53)
*Definition:** Amazon Route 53 is a scalable and highly available Domain Name System (DNS) web service provided by Amazon Web Services (AWS). It is designed to route end-user requests to globally distributed AWS resources.

**Key Features:**
  - **DNS Management:** Route 53 allows users to register and manage domain names and perform DNS routing.
  - **Scalability:** It scales automatically to handle a large volume of DNS queries.
  - **High Availability:** Route 53 is designed for high availability and fault tolerance.
  - **Integration:** Seamlessly integrates with other AWS services and resources.

**Use Cases:**
  - Domain registration and management.
  - DNS routing for web applications, APIs, and other AWS resources.
  - Load balancing traffic across multiple resources.

## Virtual Private Cloud (VPC)

**Definition:** A logically isolated section of the AWS Cloud where you can launch AWS resources.

A virtual private cloud (VPC) is a virtual network dedicated to your AWS account. It is logically isolated from other virtual networks in the AWS Cloud. You can launch your AWS resources, such as Amazon EC2 instances, into your VPC.

A VPC spans all the Availability Zones (AZ) in a region. It is always associated with a CIDR range (both IPv4 and IPv6) which defines the number of internal network addresses that may be used internally.

Amazon Web Services (AWS), a leading cloud service provider, offers several services to integrate local resources with the cloud seamlessly. One such service is the Amazon Virtual Private Cloud (VPC).

**Key Concepts:**
  - **CIDR Block:** A range of IP addresses for your VPC.
  - **Subnet:** A range of IP addresses in your VPC.
  - **Route Table:** A set of rules, called routes, that are used to determine where network traffic is directed.
  - **Internet Gateway (IGW):** Enables communication between instances in your VPC and the internet.

## Private, Public, and Elastic IP Addresses

### Private IP Address
IP addresses not reachable over the internet are defined as private. Private IPs enable communication between instances in the same network. When you launch a new instance, a private IP address is assigned, and an internal DNS hostname allocated to resolves to the private IP address of the instance. If you want to connect to this from the internet, it will not work. You would need a public IP address for that. NAT GW are used typically for this purpose.

### Public IP Address
Public IP addresses are used for communication between other instances on the internet and yours. Each instance with a public IP address is assigned an external DNS hostname too. Public IP addresses linked to your instances are from Amazon's list of public IPs. On stopping or terminating your instance, the public IP address gets released, and a new one is linked to the instance when it restarts. 

> For retention of this public IP address even after stoppage or termination, an elastic IP address needs to be used.

### Elastic IP Address (EIP)
**Definition:** A static IPv4 address designed for dynamic cloud computing.

Elastic IP addresses are static or persistent public IPs that come with your account. If any of your software or instances fail, they can be remapped to another instance quickly with the elastic IP address. An elastic IP address remains in your account until you choose to release it. A charge is associated with an Elastic IP address if it is in your account, but not allocated to an instance.

**Usage:** 
- Typically associated with EC2 instances for a persistent public IP.
- NAT GW also assigned to have an EIP for 3rd party services that need to block incoming traffic and only allows certain IP's to make sure only their customers incoming traffic is allowed.

## Internet Gateway (IGW)

*Definition:** A horizontally scaled, redundant, and highly available VPC component that allows communication between instances in your VPC and the internet.

**Key Points:**
  - Enables instances in a VPC to connect to the internet and vice versa.
  - Acts as a gateway for traffic entering or leaving the VPC.

**Usage:** Typically attached to a VPC and associated with a route in the main route table.

![](./images/aws-vpc-igw.png)

In this diagram, we've added the internet gateway that is providing the connection to the internet to your VPC. For an EC2 instance to be internet-connected, you have to adhere to the following rules:

1. Attach an Internet gateway to your VPC 
2. Ensure that your instances have either a public IP address or an elastic IP address 
3. Point your subnet’s route table to the internet gateway 
4. Make sure that your security group and network access control rules allow relevant traffic to flow in and out of your instance


## Subnets

**Definition:** A range of IP addresses in your VPC.

Within the VPC, we create subnets that are specific to AZs. 

A subnet is a way for you to divide your little cloud of IP addresses into smaller groups of IPs. This way, you can make one part of your virtual cloud public while keeping other parts of the cloud private.

It is possible to have multiple subnets in the same AZ. The purpose of subnets is to internally segregate resources contained in the VPC in every AZ. AWS Regions consist of multiple Availability Zones for DR purposes.

**Types:**
  - **Public Subnet:** Directly accessible from the internet.
  - **Private Subnet:** Not directly accessible from the internet.

### Public Subnet

A `public subnet` is a subnet that has traffic routed to an internet gateway. Its for resources that need to be accessible from the internet. 

Instances in a public subnet usually have a route to an Internet Gateway (IGW), allowing them to send and receive traffic directly from the internet.

Security:

    Public subnets may have a Network Access Control List (ACL) and Security Group rules allowing inbound and outbound traffic as needed.

### Private Subnet

Instances in a private subnet typically do not have direct access to the internet.

If instances in a private subnet need to access the internet, they do so via a `Network Address Translation (NAT) gateway` or `NAT instance` located in a public subnet.

Private subnets are used for resources that should not be directly accessible from the internet, such as databases, application servers, or internal services.

Security:

    Private subnets often have stricter security controls. They may have limited inbound traffic to enhance security.

Isolation:

    Private subnets provide an additional layer of isolation and security by limiting direct exposure to the internet.

## Route Tables
**Definition:**
- Amazon defines a route table as a set of rules, called routes, which are used to determine where network traffic is directed.
- A set of rules used to determine where network traffic is directed.

**Associations:**
Each subnet has to be linked to a route table, and a subnet can only be linked to one route table. On the other hand, one route table can have associations with multiple subnets. Every VPC has a default route table, and it is a good practice to leave it in its original state and create a new route table to customize the network traffic routes associated with your VPC. The route table diagram is as shown:

  - **Main Route Table:** Automatically associated with the main subnet.
  - **Custom Route Table:** Created and associated with custom subnets.

## NAT Gateway
**Definition:** Network Address Translation (NAT) gateway to enable instances in a private subnet to initiate outbound traffic to the internet.

**Usage:**
- Often used in private subnets for outbound internet access.
- 

![](./images/aws-vpc-routetables.png)

> All traffic inside the private subnet remains local.

## NAT Devices
A Network Address Translation (NAT) device can be used to enable instances in a private subnet to connect to the internet or the AWS services, but this prevents the internet from initiating connections with the instances in a private subnet.

Public and Private subnets protect your assets from being directly connected to the internet. For example, your web server would sit in the public subnet and database in the private subnet, which has no internet connectivity. However, your private subnet database instance might still need internet access or the ability to connect to other AWS resources. You can use a NAT device to do so. 

The NAT device directs traffic from your private subnet to either the internet or other AWS services. It then sends the response back to your instances. When traffic is directed to the internet, the source IP address of your instance is replaced with the NAT device address, and when the internet traffic returns, the NAT device translates the address to your instance’s private IP address.

![](./images/aws-vpc-NAT-device-diagram.JPG)

## NAT Gateway vs. NAT Device
AWS provides two kinds of NAT devices:

- NAT gateway 
- NAT instance 

AWS recommends the NAT gateway because it is a managed service that provides better bandwidth and availability compared to NAT instances. Every NAT gateway is created in a specific availability zone and with redundancy in that zone. A NAT Amazon Machine Image (AMI) is used to launch a NAT instance, and it subsequently runs as an instance in your VPC.

A NAT gateway must be launched in a public subnet because it needs internet connectivity. It also requires an elastic IP address, which you can select at the time of launch.

Once created, you need to update the route table associated with your private subnet to point internet-bound traffic to the NAT gateway. This way, the instances in your private subnet can communicate with the internet.

![](./images/aws-vpc-NAT-gateway.JPG)

## Security Group
**Definition:** Acts as a virtual firewall for your instance to control inbound and outbound traffic.

Rules are added to each security group, which allows traffic to or from its associated instances. Basically, a security group controls inbound and outbound traffic for one or more EC2 instances. It can be found on both the EC2 and VPC dashboards in the AWS web management console.

**Rules:**
- Specify allowed inbound and outbound traffic.
- Two-Way Traffic: Security Groups are stateful, meaning if you allow inbound traffic, the corresponding outbound traffic is automatically allowed.
- Simplified Rules: You define rules to allow traffic, and the Security Group automatically allows the response traffic. Its becaus eits statefull in nature.

![](./images/aws-vpc-sg-1.JPG)

### Example of Security Groups for Web Servers (pub;ic SUbnet)
![](./images/aws-vpc-sg-2.JPG)

### Example of Security Groups for DataStores (Private Subnet)
![](./images/aws-vpc-sg-3.JPG)
We've allowed the source to come from the internet. Because it's a Windows machine, you may need RDP access to log on and do some administration. We've also added RDP access to the security group. You could leave it open to the internet, but that would mean anyone could try and hack their way into your box. In this example, we've added a source IP address of 10.0.0.0, so the only IP ranges from that address can RDP to the instance.

## Network ACL (Access Control List)
**Definition:** An optional layer of security for your VPC that acts as a firewall for controlling traffic in and out of one or more subnets.
**Rules:**
- Define rules for allowing or denying traffic.
- Two-Way Traffic: NACLs are stateless, so you must define separate rules for inbound and outbound traffic.
- Explicit Rules: If you allow inbound traffic, you need to explicitly allow the corresponding outbound traffic, and vice versa. Its because its stateless in nature.

![](./images/aws-vpc-network-acl.JPG)

You can find the Network ACLs located somewhere between the root tables and the subnets. Here is a simplified diagram:

![](./images/aws-vpc-network-acl-2.JPG)

You can see an example of the default network ACL, which is configured to allow all traffic to flow in and out of the subnet to which it is associated. 

Each network ACL includes a rule an * (asterisk) as the rule number. The rule makes sure that if a packet is identified as not matching any of the other numbered rules, traffic is denied. You can't modify or remove this rule.

For traffic coming on the inbound:

- Rule 100 would allow traffic from all sources
- Rule * would deny traffic from all sources

![](./images/aws-vpc-network-acl-3.JPG)


## SG Vs NACL

| Feature                     | Security Groups                  | Network ACLs                       |
|-----------------------------|----------------------------------|-----------------------------------|
| **Level**                   | Instance (Layer 4)               | Subnet (Layer 3)                  |
| **Statefulness**            | Stateful                         | Stateless                         |
| **Rules**                   | Allow rules                      | Allow and deny rules              |
| **Default Action**          | Default deny; Explicit allow     | Default deny; Explicit allow      |
| **Scalability**             | Associated with multiple instances | Associated with a single subnet |
| **Changes**                 | Immediate effect                 | Changes may take some time       |
| **Logging**                 | Limited logging capabilities      | More extensive logging capabilities |

**Use Cases:**

- **Security Groups:** Controlling access to instances based on rules.
  
- **Network ACLs:** Setting rules for traffic entering or leaving a subnet.

**Summary:**

- Use **Security Groups** for instance-level control.
  
- Use **Network ACLs** for subnet-level control.

> In practice, a combination of both Security Groups and Network ACLs is often used for comprehensive network security in AWS.

## VPN (Virtual Private Network)
**Definition:** A secure connection between your on-premises equipment and your VPC.
**Usage:** Enables communication between your on-premises network and your AWS resources.

By default, instances that you launch into an Amazon VPC can't communicate with your network. You can connect your VPCs to your existing data center using hardware VPN access. By doing so, you can effectively extend your data center into the cloud and create a hybrid environment. To do this, you will need to set up a virtual private gateway. 

There is a VPN concentrator on the Amazon side of the VPN connection. For your data center, you need a customer gateway, which is either a physical device or a software application that sits on the customer’s side of the VPN connection. When you create a VPN connection, a VPN tunnel comes up when traffic is generated from the customer's side of the connection. 

![](./images/aws-custom-vpc-vpn-example-1.jpeg)

## Direct Connect
**Definition:** Establishes a dedicated network connection from your on-premises data center to AWS.
**Usage:** Provides a more reliable and consistent network experience compared to internet-based connections.

## Peering
**Definition:** Connection between two VPCs that enables instances in either VPC to communicate with each other.

A peering connection can be made between your own VPCs or with a VPC in another AWS account, as long as it is in the same region.

If you have instances in VPC A, they wouldn't be able to communicate with instances in VPC B or C unless you set up a peering connection. Peering is a one-to-one relationship; a VPC can have multiple peering connections to other VPCs, but transitive peering is not supported. In other words, VPC A can connect to B and C in the above diagram, but C cannot communicate with B unless directly paired.

> Additionally, VPCs with overlapping CIDRs cannot be paired. In the diagram, all VPCs have different IP ranges. If they have the same IP ranges, they wouldn't be able to pair.

![](./images/aws-vpc-peering-example-1.jpeg)


## Transit Gateway
**Definition:** A network transit hub that you can use to interconnect your VPCs and on-premises networks.

## CloudFront
**Definition:** A content delivery network (CDN) service that securely delivers data, videos, applications, and APIs to customers globally.

<br/>

## AWS Enterprise Architecture Cheat Diagram

![](./images/aws-enterprise-architecture-cheat-diagram.webp)

The way to define the blueprint is to segregate the services into the following tiers:

### Public Tier
The services accessible from outside AWS and act as frontline services to secure the underlying services. Even the developers have to first log in to the Jumohost VM via VPN and then are able to access the other services.

### Private Application Tier
This tier is our core where all the web and application servers reside. Mainly you deploy all your containers/VMs here. You can always decouple all you microservices here and use a message broker like SQS to decouple.

### Private Data Tier
Your database and storage services come here. Some companies still prefer to host their Database (PostgreSQL, MySQL, MongoDB), Elastic Search and Redis inside EC2 than using the managed RDS, Elasticache, CloudSearch service, which is not a problem at all, but an architecture decision.

### Gut Tier
All the communication going outside to any ERP, CRM, or other external services has to be centrally managed by a NAT gateway which is only public for the external services, and our application core services remain protected from being exposed to outside. 

`It also has static IP (Elastic IP/EIP)` so if any 3rd party service(s) requires to allow only certain IP's as per their firewall rules (Firewall as of tofay do not except DNS/FQDN) so these static IP's will be shared to those 3rd party services.

### Security Tier, DevOps Tier, AI Tier, BI & Data Processing Tier
We can either use open souce services like Terraform Vault, Metabase, Kafka, Spark and Jenkins or we can use AWS managed services to address our use case.

Keep a note of which services are Zonal, Regional and Global. For example:

1. `WAF, Route 53, CloudFront, and IAM` are global services.
2. `S3 Bucket` is Regional.
3. `EC2/ECS` is Zonal.

This architecture fits as a base for any domain be it eCommerce, Healthcare, Loyalty, Ed-tech or Fintech app.

> Above diagram and description credit goes to 
[Karan Sehgal](https://medium.com/@karansehgal_11686/aws-enterprise-architecture-cheat-diagram-c31fc4a00f48)


## Something Else
