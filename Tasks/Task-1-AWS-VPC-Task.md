
## Project Overview: Real-Time Web Application in AWS VPC

* We will create a web application that can receive requests over the internet and respond in real-time. This will include:

   1. Creating a VPC
   2. Creating subnets
   3. Setting up an Internet Gateway
   4. Configuring route tables
   5. Setting up Network ACLs
   6. Launching an EC2 instance to serve the web application

### What is VPC ?

* A VPC is an isolated portion of the AWS Cloud populated by AWS objects, such as Amazon EC2 instances.
* Amazon Virtual Private Cloud (Amazon VPC) enables you to launch AWS resources into a virtual network that you've defined. 
* This virtual network closely resembles a traditional network that you'd operate in your own data center, with the benefits of using the scalable infrastructure of AWS.
* The default limit for the number of Amazon Virtual Private Clouds (VPCs) that can be created per region is five, but this limit is adjustable:
   * `VPCs per region`: The default limit is five, but this limit is adjustable.
   * `Subnets per VPC`: The default limit is 200, and this limit is adjustable.
   * `IPv4 CIDR blocks per VPC`: The default limit is five, but this limit is adjustable up to 50.
   * `IPv6 CIDR blocks per VPC`: The default limit is one, and this limit cannot be increased

#### Step-by-Step Guide

##### Step 1: Create a VPC

1. Log in to the AWS Management Console and navigate to the `VPC` dashboard.

2. **Create a New VPC**

   *  Click on `Create VPC`
   *  Enter a name for your VPC
   *  Use an appropriate CIDR block like(`10.0.0.0/16`)
   *  Click `Create` and After creation you will see this options
      * `VPC created`:

         ![preview](images/dev9.png)

###### VPC Overview:

   * In `Resource Map` : you can see here overview on VPC

      ![preview](images/dev1.png)

   * In `CIDR` : you can add and edits IPv4/IPv6 cidr's here

      ![preview](images/dev2.png)         ![preview](images/dev3.png)

   * In `Flowlogs`:  
      * A flow log enables you to capture information about the IP traffic going to and from network interfaces in your VPC. 
      * Flow log data can be published to Amazon CloudWatch Logs or Amazon S3. 
      * After you've created a flow log, you can retrieve and view its data in the chosen destination.

         ![preview](images/dev4.png)
         ![preview](images/dev5.png)
         ![preview](images/dev6.png)
         ![preview](images/dev7.png)

* In `Integrations`: Here you can see cloudwatch internet monitor of you vpc.
   * Add this resource to a new (or existing) monitor to help you quickly visualize internet performance and availability issues, and pinpoint affected locations and internet service providers.

      ![preview](images/dev8.png)

### What is Subnet ? 

* A subnet is a range of IP addresses in your VPC. After creating a VPC, you can add one or more subnets in each Availability Zone.
* To add a new subnet to your VPC, you must specify an IPv4 CIDR block for the subnet from the range of your VPC. You can specify the Availability Zone in which you want the subnet to reside. 
* You can have multiple subnets in the same Availability Zone.

#### Step 2: Create Subnets

1. **Create Public Subnet:**

   * Go to the **Subnets** section.
   * Click on `Create subnet`
   * Choose your VPC from the dropdown.
   * Name the subnet (eg:`PublicSubnet`).
   * Specify a CIDR block (eg: `10.0.1.0/24`).
   * Choose an Availability Zone.
   * Click `Create`  

      ![preview](images/dev10.png)  
      ![preview](images/dev11.png)  
      ![preview](images/dev12.png) 

2. **Create Private Subnet:**
   * Repeat the above steps to create a private subnet (ex: `PrivateSubnet` with CIDR `10.0.2.0/24`).

      ![preview](images/dev13.png)

##### Subnet overview: 

   ![preview](images/dev14.png)
   ![preview](images/dev15.png)
   ![preview](images/dev17.png)
   ![preview](images/dev16.png)
   ![preview](images/dev18.png)

### What is Internet Gateway ?

* An internet gateway is a virtual router that connects a VPC to the internet. To create a new internet gateway specify the name for the gateway below.
* An internet gateway is a horizontally scaled, redundant, and highly available VPC component that enables communication between your VPC and the internet.
* To use an internet gateway, attach it to your VPC and specify it as a target in your subnet route table for internet-routable IPv4 or IPv6 traffic. 
* An internet gateway performs network address translation (NAT) for instances that have been assigned public IPv4 addresses

#### Step 3: Set Up an Internet Gateway

1. **Create an Internet Gateway:**
   - Go to the `Internet Gateways` section.
   - Click `Create Internet Gateway`
   - Name it (ex:`MyIGW`).
   - Click `Create`.

      ![preview](images/dev19.png)
      ![preview](images/dev20.png)

2. **Attach the Internet Gateway to Your VPC:**
   - Select the Internet Gateway and click on `Actions` > `Attach to VPC.`
   - Choose your VPC and click `Attach`.
      
      ![preview](images/dev21.png)
      ![preview](images/dev22.png)

### What is Route Tables ?

* A route table contains a set of rules, called routes, that are used to determine where network traffic from your subnet or gateway is directed.
* A route table specifies how packets are forwarded between the subnets within your VPC, the internet, and your VPN connection.

#### Step 4: Configure Route Tables

1. **Create a Route Table for the Public Subnet:**
   - Go to the `Route Tables` section.
   - Click on `Create route table`
   - Name it (e.g., `PublicRT`).
   - Choose your VPC and click `Create`.
      
      ![preview](images/dev23.png)

2. **Add Route to the Route Table:**
   - Select your public route table.
   - Click on the `Routes` tab and then `Edit routes`.
   - Click `Add route`, set the destination as `0.0.0.0/0`, and target as your `Internet Gateway` and select `igw-id`
   - Click `Save changes`.
      
      ![preview](images/dev25.png)
      ![preview](images/dev24.png)

3. **Associate the Route Table with the Public Subnet:**

   - Go to the `Subnet Associations` tab.
   - Click `Edit subnet associations`.
   - Select your `public subnet` and click `Save associations`.

      ![preview](images/dev26.png)
      ![preview](images/dev27.png)

4. **Private Route Table (optional):**

   - The private subnet will not have a direct route to the internet. Ensure it has the default route that points to the local VPC.
   - For Private route table follow above process

##### Route table overview:

![preview](images/dev27.png)
![preview](images/dev28.png)
![preview](images/dev29.png)

### What is NACL ?

* A network access control list (ACL) is an optional layer of security for your VPC that acts as a firewall for controlling traffic in and out of one or more subnets.
* A network ACL is an optional layer of security that acts as a firewall for controlling traffic in and out of a subnet.

#### Step 5: Set Up Network ACLs

1. **Create a Network ACL:**
   - Go to the **Network ACLs** section.
   - Click "Create network ACL."
   - Name it (e.g., `MyNACL`) and choose your VPC.
   - Click "Create."

      ![preview](images/dev30.png)


2. **Configure Inbound Rules:**
   - Select the NACL and go to the `Inbound rules tab`
   - Click `Edit inbound rules`
   - Add rules to allow `HTTP (port 80)` and `HTTPS (port 443)`, and `SSH (port 22)` traffic.
     - Rule #100: Type: `HTTP`, Protocol: `TCP`, Port range: `80`, Source: `0.0.0.0/0` (Allow all)
     - Rule #110: Type: `HTTPS`, Protocol: `TCP`, Port range: `443`, Source: `0.0.0.0/0` (Allow all)
     - Rule #111: Type: `SSH`, Protocol: `TCP`, Port range: `22`, Source: `0.0.0.0/0` (Allow all)
   - click `save changes`

      ![preview](images/dev31.png)
      ![preview](images/dev32.png)
      ![preview](images/dev33.png)

3. **Configure Outbound Rules:**
   - Go to the "Outbound rules" tab and click "Edit outbound rules."
   - Add rules to allow all outbound traffic 
      - Rule `#100`: Type: `All Traffic`, Protocol: `All`, Port range: `All`, Destination: `0.0.0.0/0`
   - click `save changes`

     ![preview](images/dev34.png)
     ![preview](images/dev35.png)

4. **Associate the NACL with the Public Subnet:**
   - Go to the `Subnet associations` tab.
   - Click `Edit subnet associations.`
   - Select your `public subnet` and click `Save`

      ![preview](images/dev36.png)
      ![preview](images/dev37.png)

#### Step 6: Launch an EC2 Instance

1. **Navigate to the EC2 Dashboard:**
   - Click on `Launch Instance`
   - Choose an Amazon Machine Image (AMI), such as `Amazon Linux 2`
   - Select an instance type (ex: `t2.micro`)
   - In the `Configure Instance` step, ensure you select your VPC and the public subnet.
   - Configure storage and security group (allow inbound traffic on HTTP and SSH).
   - Review and launch the instance.

      ![preview](images/dev38.png)

2. **Get the Public IP Address:**
   - After launching, note the public IP address of the instance.

#### Step 7: Set Up a Simple Web Application

1. **SSH into Your EC2 Instance:**

   ```bash
   ssh -i <your-key.pem> ec2-user@<your-instance-public-ip>   
   ```

2. **Install a Simple Web Server:**

   ```bash
   sudo yum update -y
   sudo yum install -y httpd
   ```
   ![preview](images/dev40.png)

3. **Start the Web Server:**

   ```bash
   sudo systemctl start httpd
   sudo systemctl enable httpd
   ```
   ![preview](images/dev39.png)

4. **Create a Simple HTML Page:**

   ```bash
   echo "<html><h1>Welcome to My VPC Web App\!</h1></html>" | sudo tee /var/www/html/index.html
   ```
   ![preview](images/dev41.png)

5. **Access Your Application:**
   - Open a web browser and navigate to `http://<your-instance-public-ip>`. You should see the welcome page.

      ![preview](images/dev42.png)

#### Conclusion

You've successfully created a real-time web application within a VPC, utilizing subnets, route tables, an Internet Gateway, and Network ACLs. This setup provides a solid understanding of AWS networking components and how they work together.
