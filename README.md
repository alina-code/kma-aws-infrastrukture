# kma-aws-infrastrukture

# create vcp
aws ec2 create-vpc 
	--cidr-block 10.10.0.0/18  
	--no-amazon-provided-ipv6-cidr-block 
	--tag-specifications ResourceType=vpc,Tags=[{Key=Name,Value=kma-genesis},{Key=Lesson,Value=public-clouds}] 
	--query Vpc.VpcId --output text
	
#vpc-id should be pasted from the result above
 
# create 3 subnets for vcp
aws ec2 create-subnet --vpc-id=vpc-0ed35092cf8277691 --cidr-block 10.10.1.0/24
aws ec2 create-subnet --vpc-id=vpc-0ed35092cf8277691 --cidr-block 10.10.2.0/24
aws ec2 create-subnet --vpc-id=vpc-0ed35092cf8277691 --cidr-block 10.10.3.0/24

# create security groups, that allows connection to TCP ports 22 (SSH), 80 (HTTP), 443 (HTTPS)
aws ec2 create-security-group 
		--group-name kma-security 
		--description "security group for kma-genesis instance" 
		--vpc-id  vpc-0ed35092cf8277691
		
#security-group-id is pasted from results above
aws ec2 authorize-security-group-ingress --group-id sg-0403aca8dc2526a6d --protocol tcp  --port 22  --cidr 0.0.0.0/0        
aws ec2 authorize-security-group-ingress --group-id sg-0403aca8dc2526a6d --protocol tcp  --port 80  --cidr 0.0.0.0/0        
aws ec2 authorize-security-group-ingress --group-id sg-0403aca8dc2526a6d --protocol tcp  --port 443  --cidr 0.0.0.0/0        

aws ec2 create-key-pair --key-name test-key --query 'KeyMaterial' --output text > test-key.pem

# create EC2  instance within ASG based on latest AMI Amazon Linux 2 with 15GiB attached EBS (Elastic block storage) 
aws ec2 run-instances --image-id ami-0233ed7963bf7cea6  --security-group-ids sg-0403aca8dc2526a6d --instance-type t2.micro --key-name test-key --subnet-id subnet-0400e43ea71a10e53

aws autoscaling create-launch-configuration --launch-configuration-name kma-launch-config --instance-id i-0c46d335b5ba16daa

# create ASG
aws autoscaling create-auto-scaling-group --auto-scaling-group-name autoscaling-kma --min-size=1 --max-size=5 --launch-configuration-name kma-launch-config --vpc-zone-identifier "subnet-0400e43ea71a10e53"

aws ec2 create-internet-gateway --query InternetGateway.InternetGatewayId --output text
aws ec2 attach-internet-gateway --internet-gateway-id igw-0030b4929ff3bb531 --vpc-id vpc-0ed35092cf8277691
# add load-balancer
aws elb create-load-balancer --load-balancer-name kma-load-balancer --listeners "Protocol=HTTP,LoadBalancerPort=80,InstanceProtocol=HTTP,InstancePort=80" --subnets subnet-0400e43ea71a10e53 --security-groups sg-0403aca8dc2526a6d




