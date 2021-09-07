# AWS Virtual Private Cloud (VPC) CIDR Block for VPC
## Internet Gateway
## Subnets
### Route Tables
#### Network Access Control list (NACLs)
##### Security Groups 

![](AWS_deployment_networking_security.png)

- What is a VPC?
- AWS isolated virtual network - It allows us to control virtual network environment including selectionof your own IPs address range, we can create multiple subnets within one VPC with specific network configuration. We can use both IPv4 and IPv6 for most resources, it provides security for your services or instances

- What is an Internet Gateway?
-  An Internet gateway - can transfer communications between an enterprise network and the internet - it allows internet access into the VPC
  
- What is Subnet?
- A Subnet is a segmented piece of a larget network - the goal of a subnet to split a large network intor a group of smaller, interconnected networks to help minimise traffice - or navigate traffice securly  

- Route Table RT
- RT contains set of rules, called routes, that are used to determine where the network traffic form your subnet or gateway is directed

- Network Access Control List (NACL)
- NACLs are stateless - we have to explicitly allow inbound and outbound rules - they are an added layer of security at subnet leve

- 4.3 Billion IP address in the world aproximatley 
- 
- Step 1: create a VPC with IPV valid CIDR block
- `10.0.0.0/16` - next tea member to use `10..10.0.0/16`

- Step 2: Create internet gateway
- 2.1: Attach the IG to your VPC

- Step 3: Create route table
- 3.1 Edit route and insert your IG in `target`

- Step 4: Create public subnet
- -`10.0.1.0/24`
- 4.1 associate public subnet with our RT

- Step 5: Create public NACLs
- set inbound and outbound rules for this 

- Step 6: Create a Security group for our app



Create network acl
Name -> yourname-app
VPC -> choose yours

Select your NACL
Inbound rules
Edit

This is what we need.

| Rule | Source IP | Protocol | Port       | Allow/Deny | Comments                                                                                                            |
|------|-----------|----------|------------|------------|---------------------------------------------------------------------------------------------------------------------|
|  100 | 0.0.0.0/0 | TCP      | 80         | ALLOW      | Allows inbound HTTP traffic from any IPv4 address.                                                                  |
|  110 | My IP     | TCP      | 22         | ALLOW      | Allows inbound SSH traffic from your work network (over the Internet gateway).                                      |
|  120 | 0.0.0.0/0 | TCP      | 1024-65535 | ALLOW      | Allows inbound return traffic from hosts on the Internet that are responding to requests originating in the subnet. |
|   *  | 0.0.0.0/0 | all      | all        | DENY       | Denies all inbound IPv4 traffic not already handled by a preceding rule (not modifiable).                           |

Let's discuss. Rules 100 and 110 are the same as we saw in the security groups. We are allowing all http traffic and ssh traffic from our own source IP. ( Make sure you swap MY IP for your actual IP ).

So what is rule 120 about? These are known as ephemeral ports and it's Network Address Translation that forces us to open them. We'll discuss that in a bit. But let's do the outbound ports first.

Ok let's set up the outbound ports now too.

| Rule | Source IP   | Protocol | Port       | Allow/Deny | Comments                                                                                                                                |
|------|-------------|----------|------------|------------|-----------------------------------------------------------------------------------------------------------------------------------------|
|  100 | 0.0.0.0/0   | TCP      | 80         | ALLOW      | Allows inbound HTTP traffic from any IPv4 address.                                                                                      |
|  110 | 10.0.1.0/24 | TCP      | 27017      | ALLOW      | Allows outbound Mongo access to database servers in the private subnet.                                                                 |
|  120 | 0.0.0.0/0   | TCP      | 1024-65535 | ALLOW      | Allows outbound responses to clients on the Internet (for example, serving web pages to people visiting the web servers in the subnet). |
|   *  | 0.0.0.0/0   | all      | all        | DENY       | Denies all outbound IPv4 traffic not already handled by a preceding rule (not modifiable).                                              |


## PORTS, SERVERS AND STANDARDS

So why are we opening up these huge ranges of ports?

On a particular machine, a port number coupled with the IP address of the machine is known as a socket. A combination of IP and port on both client and server is known as four tuple. This four tuple uniquely identifies a connection. In this section we will discuss how port numbers are chosen.

You already know that some of the very common services like FTP, telnet etc run on well known port numbers. While FTP server runs on port 21, Telent server runs on port 23. So, we see that some standard services that are provided by any implementation of TCP/IP have some standard ports on which they run. These standard port numbers are generally chosen from 1 to 1023. The well known ports are managed by Internet Assigned Numbers Authority(IANA).

While most standard servers (that are provided by the implementation of TCP/IP suite) run on standard port numbers, clients do not require any standard port to run on.

Client port numbers are known as ephemeral ports. By ephemeral we mean short lived. This is because a client may connect to server, do its work and then disconnect. So we used the term ‘short lived’ and hence no standard ports are required for them.

Also, since clients need to know the port numbers of the servers to connect to them, so most standard servers run on standard port numbers.

The ports reserved for clients generally range from 1024 to 5000. Port number higher than 5000 are reserved for those servers which are not standard or well known.

The client that initiates the request chooses the ephemeral port range. The range varies depending on the client's operating system. Many Linux kernels (including the Amazon Linux kernel) use ports 32768-61000. Requests originating from Elastic Load Balancing use ports 1024-65535. Windows operating systems through Windows Server 2003 use ports 1025-5000. Windows Server 2008 and later versions use ports 49152-65535. A NAT gateway uses ports 1024-65535. For example, if a request comes into a web server in your VPC from a Windows XP client on the Internet, your network ACL must have an outbound rule to enable traffic destined for ports 1025-5000.

If an instance in your VPC is the client initiating a request, your network ACL must have an inbound rule to enable traffic destined for the ephemeral ports specific to the type of instance (Amazon Linux, Windows Server 2008, and so on).

In practice, to cover the different types of clients that might initiate traffic to public-facing instances in your VPC, you can open ephemeral ports 1024-65535. However, you can also add rules to the ACL to deny traffic on any malicious ports within that range. Ensure that you place the DENY rules earlier in the table than the ALLOW rules that open the wide range of ephemeral ports.

> EXERCISE ( 20 Minutes ): Create the NACL for the private subnet.

This one is a little simpler

#### Inbound

| Rule | Source IP   | Protocol | Port       | Allow/Deny | Comments                                                                                                               |
|------|-------------|----------|------------|------------|------------------------------------------------------------------------------------------------------------------------|
|  100 | 10.0.0.0/24 | TCP      | 27017      | ALLOW      | Allows inbound Mongo traffic from the public subnet                                                                    |
|  120 | 0.0.0.0/0   | TCP      | 1024-65535 | ALLOW      | Allows inbound return traffic from the NAT device in the public subnet for requests originating in the private subnet. |
|   *  | 0.0.0.0/0   | all      | all        | DENY       | Denies all IPv4 inbound traffic not already handled by a preceding rule (not modifiable).                              |

#### Outbound

| Rule | Source IP   | Protocol | Port       | Allow/Deny | Comments                                                                                                               |
|------|-------------|----------|------------|------------|------------------------------------------------------------------------------------------------------------------------|
|  100 | 10.0.0.0/0 | TCP      | 80      | ALLOW      | Allows outbound HTTP traffic from the subnet to the Internet.                                                                    |
|  120 | 10.0.0.0/24   | TCP      | 1024-65535 | ALLOW      | Allows outbound responses to the public subnet (for example, responses to web servers in the public subnet that are communicating with DB servers in the private subnet). |
|   *  | 0.0.0.0/0   | all      | all        | DENY       | Denies all IPv4 inbound traffic not already handled by a preceding rule (not modifiable).    

You'll notice that we've opened the ephemeral ports here too. That is because while mongo only makes request on 27017 it might be asked to respond on any of the ephemeral ports
db ip: 46.137.69.235

### Moving on to Simple Storage service on S3 

- Creating bucket on S3 
- Let's run the command 
  ```
  aws s3 mb s3://devops-content --region us-west-1
  ```
  - **mb stands for make bucket**
  
  - click on the link below to view our bucket
  
  https://s3.console.aws.amazon.com/s3/buckets/devops-content/?region=us-west-1&tab=overview 

![](https://github.com/spartaglobal/Ansible/blob/lesson1/images/S3-bucket.png)

- Our bucket is empty as expected so time to store our data inside it

**Uploading data to our S3 bucket**

- Transfer/copy files to S3 using awscli from EC2/Ubuntu
```
aws s3 cp filename.yml s3://devops-content/
```
> Note: Replace the filename and bucketname
```
aws s3 cp debug_ec2_launch.yml s3://devops-content/
```
**Output**
```
upload: ./debug_ec2_launch.yml to s3://devops-content/debug_ec2_launch.yml
```
- Let's head over to our S3 on AWS UI and check if our **debug_ec2_launch.yml** is available inside the bucket
  
![](../images/uploading-to-s3.png)
**WOW!!! We can see our file in the cloud storage!**

### Moving onto retrieving data from S3

- Download the file that we just uploaded to our S3 bucket devops-content
  - Command to download 
```
  aws s3 sync
 ```
- Retrieve our object from s3 bucket called devops-content and store it in downloads_s3 folder in our host machine
 ```
aws s3 sync s3://devops-content/ downloads_s3
```
Output:
```
download: s3://devops-content/debug_ec2_launch.yml to downloads_s3/debug_ec2_launch.yml
```
Verify:
 ```
  cd downloads_s3/ 
 ```

### How can we delete data from our bucket
- Let's delete single object from our bucket - in our case the file we uploaded earlier 
```
aws s3 rm s3://devops-content/debug_ec2_launch.yml
```
Output:
```
delete: s3://devops-content/debug_ec2_launch.yml
```
- Delete all content of S3
```
aws s3 rm s3://devops-content --recursive
```

**Time to clean up**

- Delete the S3 bucket we created
```
aws s3 rb s3://devops-content
```
- **```rb``` stands for remove bucket**
- Visit below links
- https://docs.aws.amazon.com/cli/latest/reference/s3/

- **High Level Commands for S3**
- https://docs.aws.amazon.com/cli/latest/userguide/cli-services-s3-commands.html#using-s3-commands-delete-buckets
  

  **Summary:**
- :white_check_mark: What is AWS S3 and use cases
- :white_check_mark:  Setting up awscli with required dependencies
- :white_check_mark:  S3 authentication setup - with aws configure on EC2
- :white_check_mark:  Create S3 bucket from localhost-cli using awscli
- :white_check_mark: Uploading content to S3 
- :white_check_mark: Downloading content from S3
- :white_check_mark: Deleting Content from S3 using awscli