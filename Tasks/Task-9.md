Procedure:
----------
Create 4 different subnets, 1 public 3 private, 1 pub rt, 1 pvt rt, internet gateway, connect igw to vpc and pub subnet,
create nat in public subnet, and attach to private rt.
 
create subnet groups for application(22,80,443,5000,3306,*), loadbalancer(*), databases(3306,5000,80,*) formerly.
 
Then create 4 ec2 instances- 1 public, 3 private respectively.
 
copy pem file to public ec2:-  scp -i <pemfile> <pemfile> ec2-user@<public-ip>:~
 
Login into public instance first, and then login private ec2.
follow three times for 3 private ec2 instances.
And deploy application mentioned above.
 
 
Create database, db subnet groups(select 3 private ec2 subnets)
db free tier, select private subnet for db, general purpose, storage(20gb), username, password(self-managed), create
 
Create Target group - select vpc, then select private subnets, register those subnets with protocal(http), port(5000), create
create load balancer, internet-facing, select 4 subnets(include public also), listner(select target group), create.
http:<dns>/app1