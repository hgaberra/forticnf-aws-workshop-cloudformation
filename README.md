
# Secure AWS workloads with FortiGate CNF SaaS

## Welcome!

AWS Software-Defined Networking (SDN) is elastic, complex, and quite different than traditional on-premise networking. In this workshop you will learn how to use FortiGate Cloud Native Firewall as a Service (FortiGate CNF) to protect your AWS workloads deployed in common architecture patterns.

### Learning Objectives

  * Learn common AWS networking concepts such as routing traffic in and out of VPCs for various traffic flows
  * Interact with FortiGate CNF Portal to deploy CNF instances, build security policy sets, and deploy them
  * Test traffic flows in an example environment and use FortiGate CNF to control traffic flows

### Workshop Components

  * AWS Marketplace
  * FortiCloud 
  * FortiGate CNF Portal
  * AWS CloudFormation Templates (Infrastructure as Code, IaC)
  * AWS SDN (AWS intrinsic router and route tables in a VPC)
  * AWS Gateway Load Balancer (GWLB)
  * AWS Transit Gateway (TGW)
  * AWS EC2 Instances (Amazon Linux OS)


# Workshop Logistics

## Accessing the AWS environment

--TBD--

## AWS Reference Architecture Diagram
![](./content/image-ref-diag1.png)

## Notes:

  * With AWS networking there are several different ways to organize your AWS architecture to take advantage of FortiGate CNF traffic inspection. The important point to know is that as long as the traffic flow has a symmetrical routing path (for forward and reverse flows), the architecture will work.
  * This diagram will highlight two main designs that are common architecture patterns for securing traffic flows.
    * Distributed Ingress + Egress
	* Centralized Egress + East-West

### Learning Objectives

  * Understand AWS Networking Concepts
  * Understand AWS Common Architecture Patterns
  * Understand FortiGate CNF terminology
  * Subscribe to FortiGate CNF
  * Onboard an AWS account
  * Deploy a FortiGate CNF Instance & GWLBe endpoints
  * Create a policy set and apply it to a FortiGate CNF Instance
  * Test traffic flows (distributed in + egress and centralized e-w + egress)


# AWS Networking Concepts

Before diving into the reference architecture for this workshop, let's review core AWS networking concepts.

**AWS Virtual Private Cloud (VPC)** is a logically isolated section of the AWS Cloud where you can launch AWS resources in a virtual network that you define. You have complete control over your virtual networking environment, including selection of your own IP address range, creation of subnets, and configuration of route tables and network gateways.

**Availability zones (AZ)** are multiple, isolated locations within each Region that have independent power, cooling, physical security, etc. A VPC spans all of the AZs in the Region.

**Region**, is a collection of multiple regional AZs in a geographic location. The collection of AZs in the same region are all interconnected via redundant, ultra-low-latency networks.

All **subnets** within a VPC are able to reach each other with the default or intrinsic router within the VPC. All resources in a subnet use the intrinsic router (1st host IP in each subnet) as the default gateway. Each subnet must be associated with a VPC route table, which specifies the allowed routes for outbound traffic leaving the subnet.

An **Internet Gateway (IGW)** is a horizontally scaled, redundant, and highly available VPC component that allows communication between instances in your VPC and the internet. It therefore imposes no availability risks or bandwidth constraints on your network traffic.

**AWS NAT Gateway (NAT GW)** is a Network Address Translation (NAT) service. You can use a NAT gateway so that instances in a private subnet can connect to services outside your VPC but external services cannot initiate a connection with those instances.

![](./content/image-vpc.png)

**AWS Transit Gateway (TGW)** is a highly scalable cloud router that connects your VPCs in the same region to each other, to on-premise networks, and even to the internet through one hub. With the use of multiple route tables for a single TGW, you can design hub and spoke routing for traffic inspection and enforcement of security policy across multiple VPCs.

![](./content/image-tgw.png)

**AWS (Gateway Load Balancer (GWLB)** is a transparent network gateway that distributes traffic (in a 3/5 tuple flow aware manner) to a fleet of virtual appliances for inspection. This is a regional load balancer that uses GWLB endpoints (GWLBe) to securely intercept data plane traffic within consumer VPCs in the same region.  

![](./content/image-gwlb.png)

In this workshop we will use all these components to test FortiGate CNF in an enterprise design. 


# AWS Common Architecture Patterns

While there are many ways to organize your infrastructure there are two main ways to design your networking when using GWLB, centralized and distributed. From the perspective of networking, routing, and GWLBe placement.  We will discuss this further below.

FortiGate CNF is a SaaS offering that on the backend uses FortiGates, AWS GWLB, and GWLB endpoints to intercept customer traffic and inspect this transparently. As part of the deployment process for FortiGate CNF instances, the customer environment will need to implement VPC and ingress routing at the IGW to intercept the traffic to be inspected.

The FortiGate CNF security stack which includes the AWS GWLB and other components will deployed in Fortinet managed AWS accounts. The details of the diagram are simply an example of the main components used in FortiGate CNF. This is more to understand what happens when customer traffic is received at our GWLB.

![](./content/image-cap-1.png)

Decentralized designs do not require any routing between the protected VPC and another VPC through TGW. These designs allow simple service insertion with minimal routing changes to the VPC route table. The **yellow numbers** show the initial packet flow for a session and how it is routed (using ingress and VPC routes) to the GWLBe endpoint which then sends traffic to the FortiGate CNF stack. The **blue numbers** show the returned traffic after inspection by the FortiGate CNF stack.

**Note:** Any subnet where the GWLBe for the FortiGate CNF instance is to be deployed will need to have a specific tag name and value to be seen in the FortiGate CNF portal.  Currently this is the tag name '**fortigatecnf_subnet_type**' and tag value '**endpoint**'.

![](./content/image-cap-2.png)

![](./content/image-cap-3.png)

![](./content/image-cap-4.png)

![](./content/image-cap-5.png)

Centralized designs require the use of TGW to provide a simple hub and spoke architecture to inspect traffic. These can simplify east-west and egress traffic inspection needs while removing the need for IGWs and NAT Gateways to be deployed in each protected VPC for egress inspection. You can still mix a decentralized architecture to inspect ingress and even egress traffic while leveraging the centralized design for all east-west inspection.

The **yellow numbers** show the initial packet flow for a session and how it is routed (using ingress, VPC routes, and TGW routes) to the GWLBe which then sends traffic to the FortiGate CNF stack. The **blue numbers (east-west)** and **purple numbers (egress)** show the returned traffic after inspection by the FortiGate CNF stack.

![](./content/image-cap-6.png)

![](./content/image-cap-7.png)

![](./content/image-cap-8.png)

For more examples of distributed and centralized models, please reference the examples on the [FortiGate CNF Admin Guide](https://docs.fortinet.com/document/fortigate-cnf/latest/administration-guide/325439/deployment-scenarios).


# FortiGate CNF Terminology

| Term | Description |
| - | - |
| CNF Portal | The Portal in which you deploy CNF instances and manage policy sets |
| CNF Instance | A deployment of CNF resources in an auto scale group in the region of your choice |
| Policy Set | The group of FW rules, objects, and security profile groups that are assigned to one or many CNF Instance(s) |
| Security Profile Group | A group of Layer 7 inspection profiles such as DNS filter, and known bad IP blocking |


# Task 1: Subscribe to FortiGate CNF in AWS Marketplace

1.  Log into your AWS account and navigate to the AWS Marketplace listing for [FortiGate CNF](https://aws.amazon.com/marketplace/pp/prodview-vtjjha5neo52i).

![](./content/image-t1-1.png)

2.  In the upper right corner, click **View purchase options**. On the next page, click **Subscribe**.


~~2.  In the upper right corner, click **Continue to Subscribe**.  On the next page, click **Subscribe**. in the offers section, select the ***Public*** Offer and then click **Subscribe**.  Then in the green banner at the top of your screen, click **Set up your account**.~~

~~2.  In the upper right corner, click **View purchase options**.  Then in the offers section, select the ***Public*** Offer and then click **Subscribe**.  Then in the green banner at the top of your screen, click **Set up your account**.~~

![](./content/image-t1-2.png)

3. A green banner will be at the top of the screen. Click **Set up your account** and this will redirect you to associating this subscription to your FortiCloud account.

![](./content/image-t1-3.png)


4.  If you do not already have a FortiCloud account, the [Register](https://support.fortinet.com/cred/#/sign-up) button will navigate you to where you can create your own account quickly. Otherwise move on the step 2 and login to your existing FortiCloud Account.

![](./content/image-t1-4.png)

5.  You should see this page showing your information and that the subscription has been successfully applied to your FortiCloud account.

![](./content/image-t1-5.png)

6.  Click on your account and you will now be on the dashboard of FortiGate CNF.

![](./content/image-t1-6.png)


7. This concludes this section.


# Task 2: Onboard an AWS account to FortiGate CNF

1.  In the FortiGate CNF Portal, navigate to AWS Accounts, then click **New** to start the add account wizard.

![](./content/image-t2-1.png)

2.  In a new browser tab, log into your AWS account and click on your **IAM user name in the upper right corner**. This will allow you to see and copy your AWS account ID.

![](./content/image-t2-2.png)

3.  In the FortiGate CNF Portal, provide your AWS account ID and select Launch CloudFormation Template. This will redirect you to the CloudFormation Console in your AWS account in the us-west-2 region.

**Note:** Your browser may block the popup window to launch the CloudFormation console.  Please check your browser for blocked popup notifications.

![](./content/image-t-3.png)

4.  This CloudFormation Template creates the following items:

    a.  S3 bucket for sending logs

    b.  IAM Cross Account Role which allows us to manage GWLBe endpoints, describe VPCs, push logs to S3, and describe instances and EKS clusters for the SDN connector feature (dynamic address objects based on metadata).

    c.  Custom resources which kicks off automation on our managed accounts to complete backend tasks for onboarding the AWS account.

5.  Please follow through the create stack wizard **without changing the region or any of the parameter values**. Simply follow the steps outlined in the FortiGate CNF Portal and click through the CloudFormation wizard.

**Note:** This CloudFormation template must be ran in the us-west-2 (Oregon) region for successful onboarding and ongoing operations of this AWS account with FortiGate CNF.

![](./content/image-t2-4.png)

6.  Once the CloudFormation template has been created successfully, you should see your account showing **Success** in the AWS account page of FortiGate CNF.

![](./content/image-t2-5.png)

7.  This concludes this section.


# Task 3: Deploying CNF Instances and GWLBe endpoints

1.  In the FortiGate CNF portal, navigate to CNF instances and click **New**.

![](./content/image-t3-1.png)

2.  Provide a name for the CNF instance, select **us-east-2** for the region for deployment, select **Internal S3** for the log type, and select the **S3 bucket** created by CloudFormation for the logging destination. Then click **Save**. This will drop you back to the list of CNF instances while this is deployed in the background.

**Note:** FortiGate CNF is available in the following regions today.  Based on customer demand, more regions will be supported in the future.

  * us-east-1 (N. Virginia)
  * us-east-2 (Ohio)
  * us-west-1 (N. California)
  * us-west-1 (Oregon)
  * eu-central-1 (Frankfurt)
  * eu-west-1 (Ireland)
  * ap-northeast-1 (Tokyo)

![](./content/image-t3-2.png)

![](./content/image-t3-3.png)

3.  The CNF Instance should show up as **active after roughly 10 minutes** (Now is a great time for a break :) ). Then you can **select and edit** it to deploy endpoints and assign a policy set.

![](./content/image-t3-4.png)

![](./content/image-t3-5.png)

4.  On the Configure Endpoints section of the wizard, click the **New** button. Then you can select the account, VPC, then toggle the **Select from all subnets** to off (this filters the subnets to only show ones that are properly tagged), and the subnet to deploy the VPC endpoint to. Repeat this step for all subnets in the table below, then click the **Next** button.  Once all have been created, click **Next*.

**Note:** In order for FortiGate CNF to successfully create a GWLBe in a subnet, **the subnet must be properly tagged**.  The subnet needs a Tag ***Name = fortigatecnf_subnet_type*** and Tag ***Value = endpoint***. Otherwise you will see an error that the subnet ID is invalid.

| VPC | Subnet |
| - | - |
| Application-VPC | Application-GwlbeSubnet1 |
| Application-VPC | Application-GwlbeSubnet2 |
| Inspection-VPC | Inspection-GwlbeSubnet1 |
| Inspection-VPC | Inspection-GwlbeSubnet2 |

![](./content/image-t3-6.png)

![](./content/image-t3-7.png)

![](./content/image-t3-8.png)

5.  On the Configure Policy Set section of the wizard, use the default 'allow_all' policy to allow all traffic from a Layer 4 perspective and click **Finalize** to push that default policy. You should then see the list of CNF instances again.

![](./content/image-t3-9.png)

![](./content/image-t3-10.png)

6.  To validate all GWLBe endpoints have been deployed and are active (takes ~5 mins), **select and edit** the CNF instance and click **Next** to view the GWLBe endpoints on the Configure Endpoints section of the wizard. Then click **Exit** to leave the CNF configuration wizard.

![](./content/image-t3-11.png)

7.  Log into your AWS Account and navigate to the **VPC Console > Endpoints**.  Each of the GWLBe endpoints you deployed in the FortiGate CNF Portal should be visible in your account.  Notice the tag name and value pairs assigned to the endpoints.

![](./content/image-t3-12.png)

9.  At this point in a normal environment, you would need to create ingress and VPC routes to direct traffic to the GWLBe endpoints that were created by FortiGate CNF for inspection.  However, for this workshop there is a Lambda function that is creating these routes for you to match the AWS Reference Architecture Diagram.

10.  To validate that the relevant VPC routes have been automatically created to route traffic to the GWLBe endpoints, in the AWS VPC console, **navigate to Virtual Private Cloud > Route Tables**, select each of the route tables listed below and confirm these routes exist in the route tab of the route table details pane.


| VPC Route Table | # Routes to GLWBe |
| - | - |
| Application-IgwRouteTable | 2x (one per public subnet), one per GWLBe (each AZ) |
| Application-Public1RouteTable | 1x (default route), GWLBe AZ1 |
| Application-Public2RouteTable | 1x (default route), GWLBe AZ2 |
| Inspection-Public1RouteTable | 1x (10.0.0.0/8 route), GWLBe AZ1 |
| Inspection-Public2RouteTable | 1x (10.0.0.0/8 route), GWLBe AZ2 |
| Inspection-TgwAttach1RouteTable | 1x (default route), GWLBe AZ1 |
| Inspection-TgwAttach2RouteTable | 1x (default route), GWLBe AZ2 |

![](./content/image-t3-13.png)


11.  To confirm that app1 in the Application VPC is reachable, in the AWS CloudFormation console, **toggle the view nested button to off** > then select the stack name > and on the details pane select the outputs tab.  You should see the output for **URLforApp1**.  Click on the value for that output to check that App1 is reachable now.  You should see a simple webpage with some metadata about the backend web server instance that is reachable via the public Network Load Balancer (NLB).

![](./content/image-t3-14.png)

![](./content/image-t3-15.png)

8.  This concludes this section.


# Task 4: Create a policy set and apply it to a FortiGate CNF Instance

1.  At this point, we are using the default **allow_all** policy set which allows all communication to be allowed without any restriction from a Layer 4 and Layer 7 perspective.

![](./content/image-t4-1.png)

2.  To customize the actual L4 rules and L7 security profile groups applied, in the FortiGate CNF Portal **navigate to Policy & Objects > Policy Sets** to create your own policy set.  Simply **click Create New**, select **Policy Set**, and give your policy set a name.

![](./content/image-t4-2.png)

3.  Before adding in L4 rules within the policy set, create a few simple address objects.  **Navigate to Policy & Objects > Addresses*, click New, and Address. Then create each of the address objects below.

| Name | Type | IP/Netmask Value |
| - | - | - |
| ClassA | Subnet | 10.0.0.0/8 |
| ClassB | Subnet | 172.16.0.0/12 |
| ClassC | Subnet | 192.168.0.0/16 |
| GooglePublicDNS1 | Subnet | 8.8.8.8/32 |
| GooglePublicDNS2 | Subnet | 8.8.4.4/32 |
| AppPublicSubnet1 | Subnet | 10.1.1.0/24 |
| AppPublicSubnet2 | Subnet | 10.1.2.0/24 |

![](./content/image-t4-3.png)

![](./content/image-t4-4.png)

4.  Next, create an Address Group to include all the RFC 1918 class objects. On the same page, **click New, and Address Group**. Then create each the address object below.

| Name | Members Value |
| - |  - |
| RFC-1918 | ClassA, ClassB, ClassC |

![](./content/image-t4-5.png)

![](./content/image-t4-6.png)

5.  In FortiGate CNF you can create different types of address objects to be more flexible and granular in your rules within your policy set. Create an FQDN based address object by **clicking New, and Address**. Select FQDN for Type, then create the address object below.

**Note:** This can be used for internal Application, Network, and even legacy Elastic Load Balancers (ie ALB, NLB, ELB) to dynamically resolve their private IPs.

| Name | Type | FQDN Value |
| - | - | - |
| ipinfo.io | FQDN | ipinfo.io |

![](./content/image-t4-7.png)

6. Geography based address objects are available in FortiGate CNF. This allows controlling traffic based on public IPs assigned to countries around the globe. These objects can be used as a source or destination object within policies used in a policy set. Create a geo based address object by **clicking New, and Address**. Select Geography for Type, then create the address objects below.

**Note:** The IP for the country or region is automatically determined from the Geography IP database which is provided by FortiGuard Servers on a recurring basis.  For more granular control to applications (especially external), it is recommended to use URL or DNS filtering and even Application Control for L7 inspection.

| Name | Type | Country/Region Value |
| - | - | - |
| UnitedStates | Geography | United States |
| Russia | Geography | Russian Federation |

![](./content/image-t4-8.png)

7.  Dynamic metadata based address objects are available in FortiGate CNF. This allows controlling of traffic based on things such as VPC ID, Auto Scale Group, EKS Cluster or Pod, and even Tag Name + Value pairs for a given AWS account and region. Create a dynamic based address object by **clicking New, and Address**. Select Dynamic for Type, then create the address objects below.

**Note:** This is using AWS API calls behind the scenes such as ec2:DescribeInstances, eks:ListClusters, eks:DescribeCluster, etc. For instances, these must be running to have their IP address(es) (public and or private IPs) returned.

**Note:** For each object, you will use the same values for these settings:

| Type | AWS Account ID | AWS Region |
| - | - | - |
| Dynamic | Workshop-AWS-Account-ID | us-east-2 |

Here is the list of dynamic objects to create:

| Name | SDN Address Type | Filter Value |
| - | - | - |
| ProdAPIBackend | Private | Tag.env=prod AND Tag.app-role=api AND Tag.app-tier=backend |
| ProdAuthBackend | Private | Tag.env=prod AND Tag.app-role=auth AND Tag.app-tier=backend |
| SDNGroup1 | Private | Tag.sdn-group=group1 |
| SDNGroup2 | Private | Tag.sdn-group=group2 |
| SDNGroup3 | Private | Tag.sdn-group=group3 |

![](./content/image-t4-9.png)

![](./content/image-t4-10.png)

8.  Now you will create a policy set to enforce L4 rules using the address objects you just created in the previous steps.  **Navigate to Policy & Objects > Policy Sets** and click New, Policy Set. Give it a name and click Ok. You will be returned to the list of policy sets. **Select your policy set and click Edit**. 

![](./content/image-t4-11.png)

![](./content/image-t4-12.png)

![](./content/image-t4-13.png)

9. Now you can create the policies listed below to control all directions of traffic within the example environment. **Click New** and create the policies listed below:

| Name | Source | Destination | Service | Action | Log Allowed Traffic |
| - | - | - | - | - | - |
| BlockList-Inbound | Russia | all | ALL | DENY | All Sessions |
| BlockList-Outbound | all | Russia | ALL | DENY | All Sessions |
| HTTPS-Inbound | UnitedStates | RFC-1918 | HTTPS | ACCEPT | All Sessions |
| ICMP-EastWest | RFC-1918 | RFC-1918 | ALL_ICMP | ACCEPT | All Sessions |
| AuthSharedServices-EastWest | ProdAPIBackend | ProdAuthBackend | HTTPS + RADIUS | ACCEPT | All Sessions |
| ICMP-Egress | RFC-1918 | UnitedStates | ALL_ICMP | ACCEPT | All Sessions |
| IPinfo-Egress | SDNGroup1 + SDNGroup2 + AppPublicSubnet1 + AppPublicSubnet2 | ipinfo.io | HTTPS | ACCEPT | All Sessions |

![](./content/image-t4-14.png)

![](./content/image-t4-15.png)

10. In order to use this policy set, it must be applied to the deployed FortiGate CNF Instance. Navigate to CNF instances and **select and edit** the CNF Instance then **click the Configure Policy Set** bread crumb. In the Apply Policy Set, select your policy set then **click Save then Finalize**.

![](./content/image-t4-16.png)


11. This concludes this section.


# Task 5: Test traffic flows (distributed in + egress and centralized e-w + egress)

In this section you will validate that the policy set is controlling traffic as intended. Below is the overall architecture of the environment.

We will look into these specific traffic flows:
  * Distributed Ingress
  * Distributed Egress
  * Centralized Egress
  * Centralized East West

![](./content/image-ref-diag1.png)


# Distributed Ingress

For this traffic flow we will focus on the Application VPC. Distributed ingress is commonly used when there is a need to inspect traffic for a VPC that is directly accessible with an attached IGW and resources with a public Elastic IP (EIP) or behind a public load balancer (ie ALB, NLB, etc). The benefit of this design is that traffic does not need to traverse additional AWS networking components for inspection so each VPC is isolated from others. The caveat to consider is that each VPC would need a directly attached IGW and resources such as load balancers, NAT GWs, etc that have additional cost.

![](./content/image-dist-ingress-diag1.png)

**Step 1:** An inbound connection starts with an external user initiating a connection to a public resource such as a public NLB. The public NLB has a DNS record that resolves to a public IP for one of the NLB's Elastic Network Interface (ENI) in either public subnet. The first packet (ie TCP SYN) will then be seen at the IGW attached to the VPC where the public NLB is deployed. Since there is a Ingress route table assigned to the IGW, traffic destined to either public subnet will be sent to the GWLBe endpoint in the same AZ.

**Note:** The IGW will perform destination NAT to change the public IP of the NLB to the private IP of the NLB ENI.

**Step 2:** The traffic is received at the GWLBe endpoint which then routes the traffic to the associated GWLB ENI in the same AZ in the managed Fortinet AWS account/VPC. This is done behind the scene using AWS Private Link.

**Step 3:** The traffic is received at the GWLB ENI and is then encapsulated in a GENEVE tunnel and routed to one of the instances in the FortiGate CNF auto scale group for traffic inspection. Post inspection, if the traffic is allowed, the instance will hairpin the traffic back to the same GWLB ENI over GENEVE. Then the GWLB ENI will hairpin the traffic back to the same GWLBe endpoint.

**Step 4:** The GWLBe endpoint will then route the inspected traffic to the intrinsic router. The intrinsic router will route traffic directly to the NLB's ENI as specified in the VPC route table assigned to the subnet.

**Step 5:** The NLB will send traffic to a healthy target, in either AZ since cross zone load balancing is enabled.

**Note:** The NLB will perform destination NAT to change the private IP to that of the healthy target.

**Step 6:** The web server will receive the traffic and respond. The return traffic will flow these steps in reverse.

1.  To test out this flow navigate to the AWS CloudFormation console and **toggle the view nested button to off** > then select the stack name > and on the details pane select the outputs tab. You should see the output for **URLforApp1**. Click on the value for that output to check that App1 is no longer reachable now. Click on the value for the output **EncryptedURLforApp1** and you will see the self-signed certificate warning. Once you accept the warning, you will see the test web page.

**Note:** You are now only allowing HTTPS inbound to your environment that is sourced from a public IP within the United States!

![](./content/image-t5-1.png)

![](./content/image-t5-2.png)


# Distributed Egress

For this traffic flow we will focus on the Application VPC. Distributed egress is commonly used when there is a need to inspect traffic for a VPC that has an attached IGW and resources with a public Elastic IP (EIP) or that are behind a NAT GW. The benefit of this design is that traffic does not need to traverse additional AWS networking components for inspection so each VPC is isolated from others. The caveat to consider is that each VPC would need a directly attached IGW and resources such as NAT GWs that have additional cost.

![](./content/image-dist-egress-diag1.png)

**Step 1:** An outbound connection starts with a private EC2 instance initiating a connection to a public resource. The first packet (ie TCP SYN) will be routed to the intrinsic router which will route traffic to the NAT GW in the same AZ, as configured in the assigned VPC route table. The EC2 instance has a default route, received via DHCP, that points to the first host IP in the subnet which is the intrinsic router.

**Step 2:** The traffic is received at the NAT GW ENI which then routes the traffic to the associated GWLBe endpoint in the same AZ, as configured in the assigned VPC route table.

**Note:** The NAT GW will source NAT the traffic to the private IP assigned to its ENI.

**Step 3:**  The traffic is received at the GWLBe endpoint which then routes the traffic to the associated GWLB ENI in the same AZ in the managed Fortinet AWS account/VPC. This is done behind the scene using AWS Private Link.

**Step 4:** The traffic is received at the GWLB ENI and is then encapsulated in a GENEVE tunnel and routed to one of the instances in the FortiGate CNF auto scale group for traffic inspection. Post inspection, if the traffic is allowed, the instance will hairpin the traffic back to the same GWLB ENI over GENEVE. Then the GWLB ENI will hairpin the traffic back to the same GWLBe endpoint.

**Step 5:** The GWLBe endpoint will then route the inspected traffic to the intrinsic router. The intrinsic router will route traffic directly to the IGW as specified in the VPC route table assigned to the subnet.

**NOTE:** The IGW will source NAT the traffic to the public EIP assigned to the NAT GW ENI.

**Step 6:** The destination will receive the traffic and respond. The return traffic will be intercepted at the IGW and routed to the GWLBe endpoint. Then the return traffic follow these steps in reverse.

1.  To test out this flow navigate to the **AWS EC2 console and go to Instances > Instances**. Then select either **AppInstance** and click Connect > EC2 serial console. Copy the instance ID as this will be the username and click connect.

![](./content/image-t5-3.png)

![](./content/image-t5-4.png)

2.  Login to the instance with the instance ID as the username and **Fortinet123!** as the password. Then run the commands below to test traffic:

```
ping 8.8.8.8
curl http://ipinfo.io
curl https://ipinfo.io
```
**Note:** You are now only allowing HTTPS outbound to one FQDN and ICMP to any public IP within the United States!

![](./content/image-t5-5.png)


# Centralized Egress

For this traffic flow we will focus on the Shared Services, Workload, and Inspection VPCs. Centralized egress is commonly used when there is a strong desire to control egress traffic through a common set of NAT GWs in an egress or what we call an Inspection VPC. The benefit of this design is that you only need NAT GWs in the Inspection VPC (one per AZ) vs every VPC (one per AZ), which have an hourly cost. The caveat of this design is traffic will traverse additional AWS networking components for inspection (ie TGW, etc) that will have additional cost.

![](./content/image-cent-egress-diag1.png)

**Step 1:** An outbound connection starts with a private EC2 instance initiating a connection to a public resource. The first packet (ie TCP SYN) will be routed to the intrinsic router which will route traffic to the TGW attachment in the same AZ, as configured in the assigned VPC route table. The EC2 instance has a default route, received via DHCP, that points to the first host IP in the subnet which is the intrinsic router.

**Step 2:** The traffic is received at the TGW ENI which then routes the traffic to the Inspection VPC TGW attachment, as configured in the associated Spoke VPC TGW route table.

**Step 3:**  The traffic is received at the TGW ENI in the Inspection VPC which then routes the traffic to the GWLBe endpoint in the same AZ, as configured in the associated VPC route table.

**Step 4:** The traffic is received at the GWLBe endpoint which then routes the traffic to the associated GWLB ENI in the same AZ in the managed Fortinet AWS account/VPC. This is done behind the scene using AWS Private Link.

**Step 5:** The traffic is received at the GWLB ENI and is then encapsulated in a GENEVE tunnel and routed to one of the instances in the FortiGate CNF auto scale group for traffic inspection. Post inspection, if the traffic is allowed, the instance will hairpin the traffic back to the same GWLB ENI over GENEVE. Then the GWLB ENI will hairpin the traffic back to the same GWLBe endpoint.

**Step 6:** The GWLBe endpoint will then route the inspected traffic to the intrinsic router. The intrinsic router will route traffic directly to the NAT GW in the same AZ as specified in the VPC route table assigned to the subnet.

**Step 7:** The traffic is received at the NAT GW ENI which then routes the traffic to the intrinsic router. The intrinsic router will route traffic directly to the IGW as specified in the VPC route table assigned to the subnet.

**Note:** The NAT GW will source NAT the traffic to the private IP assigned to its ENI.

**Step 8:** The destination will receive the traffic and respond. The return traffic will follow these steps in reverse.

**NOTE:** The IGW will source NAT the traffic to the public EIP assigned to the NAT GW ENI.

1.  To test out this flow navigate to the **AWS EC2 console and go to Instances > Instances**. Then select either **WrkInstance** and click Connect > EC2 serial console. Copy the instance ID as this will be the username and click connect.

![](./content/image-t5-6.png)

![](./content/image-t5-7.png)

2.  Login to the instance with the instance ID as the username and **Fortinet123!** as the password. Then run the commands below to test traffic:

```
ping 8.8.8.8
curl http://ipinfo.io
curl https://ipinfo.io
```
**Note:** You are now only allowing HTTPS outbound to one FQDN and ICMP to any public IP within the United States!

**Question 1:** What happens if you try the same test from **SSInstance1**?

**Question 2:** What address objects are allowing this communication to work even though the sdn-group = group3 for this instance?

![](./content/image-t5-8.png)


# Centralized East West

For this traffic flow we will focus on the Shared Services, Workload, and Inspection VPCs. Centralized East West is commonly used when there is need for multiple VPCs in the same region to access common private resources such as a shared services VPC, premise items, or workloads/services in other VPCs. The benefit of this design is that this a flexible but simple way to interconnect many resources in the same region. The caveat of this design is traffic will traverse additional AWS networking components for inspection (ie TGW, etc) that will have additional cost.

![](./content/image-cent-eastwest-diag1.png)

**Step 1:** An outbound connection starts with a private EC2 instance initiating a connection to a public resource. The first packet (ie TCP SYN) will be routed to the intrinsic router which will route traffic to the TGW attachment in the same AZ, as configured in the assigned VPC route table. The EC2 instance has a default route, received via DHCP, that points to the first host IP in the subnet which is the intrinsic router.

**Step 2:** The traffic is received at the TGW ENI which then routes the traffic to the Inspection VPC TGW attachment, as configured in the associated Spoke VPC TGW route table.

**Step 3:**  The traffic is received at the TGW ENI in the Inspection VPC which then routes the traffic to the GWLBe endpoint in the same AZ, as configured in the associated VPC route table.

**Step 4:** The traffic is received at the GWLBe endpoint which then routes the traffic to the associated GWLB ENI in the same AZ in the managed Fortinet AWS account/VPC. This is done behind the scene using AWS Private Link.

**Step 5:** The traffic is received at the GWLB ENI and is then encapsulated in a GENEVE tunnel and routed to one of the instances in the FortiGate CNF auto scale group for traffic inspection. Post inspection, if the traffic is allowed, the instance will hairpin the traffic back to the same GWLB ENI over GENEVE. Then the GWLB ENI will hairpin the traffic back to the same GWLBe endpoint.

**Step 6:** The GWLBe endpoint will then route the inspected traffic to the intrinsic router. The intrinsic router will route traffic to the TGW attachment in the same AZ as specified in the VPC route table assigned to the subnet.

**Step 7:** The traffic is received at the TGW ENI which then routes the traffic to the Shared Services VPC TGW attachment, as configured in the associated Inspection VPC TGW route table.

**Step 8:** The traffic is received at the TGW ENI in the Shared Services VPC which then routes the traffic to the destination, as configured in the associated VPC route table.

1.  To test out this flow navigate to the **AWS EC2 console and go to Instances > Instances**. Then select **WrkInstance2** and click Connect > EC2 serial console. Copy the instance ID as this will be the username and click connect.

![](./content/image-t5-9.png)

![](./content/image-t5-10.png)

2.  Login to the instance with the instance ID as the username and **Fortinet123!** as the password. Then run the commands below to test traffic:

```
ping 10.1.2.10
ssh ec2-user@10.1.2.10
curl -k https://10.1.2.10
```
**Note:** You are now only allowing HTTPS and RADIUS access between two resourced based on metadata (ie Tags)!

**Question:** What happens if you try the same test from **WrkInstance1**?

**Question 2:** What tags are allowing this communication to match the dynamic address objects?

![](./content/image-t5-11.png)


# Task 6: Providing Feedback

We truly appreciate your time and efforts to review FortiGate CNF and value your feedback greatly. Please take the time to capture your thoughts on:

1.  What do you like about the FortiGate CNF solution. Does anything beneficial stand out in comparison to the competition?

2.  What room for improvement do you see in the solution. Is there anything we are missing that the competition has or companies are asking for but no solution exists on the market?

3.  For any issues or enhancements, please capture screenshots and provide as much detail as possible. What specific problem or use case does the enhancement solve?

4.  What are your thoughts on using FortiManager to manage policy and objects vs having all items in the FortiGate CNF portal? Any pros/cons for leaning heavy towards one side or is parity expected by customers?

5.  This concludes this section.  **Thank you!!!**
