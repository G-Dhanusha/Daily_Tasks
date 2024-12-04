### CLI - Task

#### Create Access Key and Secrect Key for accessing AWS through CLI

* Collect Access Key and Secret Key

    ![preview](images/dev76.png)

##### Install and Setup AWS cli on Local machine
##### Config PA credentials 

- Create VPC using 
- create Pub and Pvt subnets
- create IGW
- Attach IGW to VPCq
- Create Pub and PVT RT
- Attach Pub sub to Pub rt
- Attach Pvt Sub to Pvt rt
- Attach IGW to Pub RT

- Create Sg for ssh // http
- Create a Ec2 in Pub Sub
- Create a Ec2 in Pvt Sub 

## Task: 

### Install AWS CLI and we need to configure our aws credentials using AWS Access Key ID and AWS Secret Access Key before doing these below commands:

#### **Create VPC**

1) `VPC Creation`

    ```bash
    aws ec2 create-vpc --cidr-block 10.0.0.0/16

    # for tags purpose use below commands

    $VPC_ID = (aws ec2 create-vpc --cidr-block 10.0.0.0/16 --query 'Vpc.VpcId' --output text)

    aws ec2 create-tags --resources $VPC_ID --tags Key=Name,Value=MyVPC Key=Environment,Value=Production
    ```
    ![preview](images/dev77.png)

    ![preview](images/dev79.png)

    ![preview](images/dev78.png)

#### **Create Subnets**

2) `Creating subnets`

    ```bash

    aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block 10.0.1.0/24 --availability-zone us-east-1a

    # for tags purpose use below commands

    $SUBNET_ID1 = (aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block 10.0.1.0/24 --query 'Subnet.SubnetId' --output text)

    aws ec2 create-tags --resources $SUBNET_ID1 --tags Key=Name,Value=MySubnet1 Key=Environment,Value=Production
    ```
    ![preview](images/dev80.png)

3) `subnet2 creation`

    ```bash
    aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block 10.0.2.0/24 --availability-zone us-east-1b

    # For tag purpose use below commands

    $SUBNET_ID2 = (aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block 10.0.2.0/24 --query 'Subnet.SubnetId' --output text)

    aws ec2 create-tags --resources $SUBNET_ID2 --tags Key=Name,Value=MySubnet2 Key=Environment,Value=Production
    ```

    ![preview](images/dev81.png)

    ![preview](images/dev82.png)

    ![preview](images/dev83.png)

#### **Create and Attach IGW(Internet Gateway)**

* `Create IGW`

    ```bash
    aws ec2 create-internet-gateway

    aws ec2 attach-internet-gateway --internet-gateway-id <your-internet-gateway-id> --vpc-id <your-vpc-id>

    # For tag purpose use below commands

    $IGW_ID=$(aws ec2 create-internet-gateway --query 'InternetGateway.InternetGatewayId' --output text)

    aws ec2 attach-internet-gateway --internet-gateway-id $IGW_ID --vpc-id $VPC_ID

    aws ec2 create-tags --resources $IGW_ID --tags Key=Name,Value=MyInternetGateway Key=Environment,Value=Production
    ```

    ![preview](images/dev84.png)

#### **Create and Attach RT(Route Tables)**

* `Create RT`

    ```bash
    aws ec2 create-route-table --vpc-id <your-vpc-id>

    aws ec2 create-route --route-table-id <your-route-table-id> --destination-cidr-block 0.0.0.0/0 --gateway-id <your-internet-gateway-id>

    aws ec2 associate-route-table --route-table-id <your-route-table-id> --subnet-id <your-subnet-id-1>
    
    aws ec2 associate-route-table --route-table-id <your-route-table-id> --subnet-id <your-subnet-id-2>

    # For tag purpose use below commands

    $RT_ID=$(aws ec2 create-route-table --vpc-id $VPC_ID --query 'RouteTable.RouteTableId' --output text)

    aws ec2 create-route --route-table-id $RT_ID --destination-cidr-block 0.0.0.0/0 --gateway-id $IGW_ID

    aws ec2 associate-route-table --route-table-id $RT_ID --subnet-id $SUBNET_ID1

    aws ec2 associate-route-table --route-table-id $RT_ID --subnet-id $SUBNET_ID2

    aws ec2 create-tags --resources $RT_ID --tags Key=Name,Value=MyRouteTable Key=Environment,Value=Production
    ```

    ![preview](images/dev85.png)
    ![preview](images/dev88.png)

#### **Create SG(Security Group)**

* Security Group

    ```bash
    aws ec2 create-security-group --group-name MySecurityGroup --description "My security group" --vpc-id <your-vpc-id>

    aws ec2 authorize-security-group-ingress --group-id <your-security-group-id> --protocol tcp --port 22 --cidr 0.0.0.0/0
    
    aws ec2 authorize-security-group-ingress --group-id <your-security-group-id> --protocol tcp --port 80 --cidr 0.0.0.0/0

    # for tags purpose use below commands

    $SG_ID=$(aws ec2 create-security-group --group-name MySecurityGroup --description "My security group" --vpc-id $VPC_ID --query 'GroupId' --output text)

    aws ec2 authorize-security-group-ingress --group-id $SG_ID --protocol tcp --port 22 --cidr 0.0.0.0/0

    aws ec2 authorize-security-group-ingress --group-id $SG_ID --protocol tcp --port 80 --cidr 0.0.0.0/0

    aws ec2 create-tags --resources $SG_ID --tags Key=Name,Value=MySecurityGroup Key=Environment,Value=Production    
    ```
    ![preview](images/dev86.png)
    ![preview](images/dev89.png)

#### **Create EC2 Instance**

* `EC2 Instance`

    ```bash
    aws ec2 run-instances --image-id ami-xxxxxxxxxxxxxxxxx --instance-type t2.micro --key-name MyKeyPair --security-group-ids <your-security-group-id> --subnet-id <your-subnet-id-1>

    # for tags purpose use below commands, before that collect AMI ID and Create Keypair

    $INSTANCE_ID=$(aws ec2 run-instances --image-id ami-012967cc5a8c9f891 --instance-type t2.micro --key-name awscli --security-group-ids $SG_ID --subnet-id $SUBNET_ID1 --query 'Instances[0].InstanceId' --output text)

    aws ec2 create-tags --resources $INSTANCE_ID --tags Key=Name,Value=MyEC2Instance Key=Environment,Value=Production
    ```
    ![preview](images/dev87.png)
    
    ![preview](images/dev90.png)