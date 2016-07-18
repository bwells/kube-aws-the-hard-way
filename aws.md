## Network

1. elastic ip per machine
2. Multi AZ

VPC - 192.168.0.0/16
Two subnetes -
us-east-1b - 192.168.10.0/24
us-east-1c - 192.168.11.0/24
- enable auto assign public IP for now

create internet gateway
attach to k8s vpc
add 0.0.0.0/0 route to internet gateway to vpc route table
associate k8s subnets, whatever that means

create security group that allows
all traffic from 192.168.0.0/16
ssh traffic form 0.0.0.0

# create a vpc
aws ec2 create-vpc --cidr-block 192.168.0.0/16

# create two subnets
#    one in us-east-1b
#    one in us-east-1c
aws ec2 create-subnet \
	--vpc-id $VPC_ID \
	--cidr-block 192.168.10.0/24 \
	--availability-zone us-east-1b
aws ec2 create-subnet \
	--vpc-id $VPC_ID \
	--cidr-block 192.168.11.0/24 \
	--availability-zone us-east-1b

# create an internet gateway.
aws ec2 create-internet-gateway

# associate internet gateway to our vpc
aws ec2 attach-internet-gateway \
	--internet-gateway-id $INTERNET_GATEWAY_ID \
	--vpc-id $VPC_ID

# associate each vpc with the vpc's route table
aws ec2 associate-route-table \
	--subnet-id $SUBNET1_ID \
	--route-table-id $ROUTE_TABLE_ID
aws ec2 associate-route-table \
	--subnet-id $SUBNET2_ID \
	--route-table-id $ROUTE_TABLE_ID

# create a route for all non vpc traffic through the internet gateway
aws ec2 create-route \
	--route-table-id $ROUTE_TABLE_ID \
	--destination-cidr-block 0.0.0.0/0 \
	--gateway-id $INTERNET_GATEWAY_ID

# create a security group for all k8s traffic
aws ec2 create-security-group \
	--group-name k8s \
	--description "SSH and all internal traffic" \
	--vpc-id $VPC_ID

# add a security group rule allowing SSH traffic in from anywhere
aws ec2 authorize-security-group-ingress \
	--group-id $SECURITY_GROUP_ID \
	--protocol tcp \
	--port 22 \
	--cidr 0.0.0.0/0

# add a security group rule allowing all traffic within the VPC
aws ec2 authorize-security-group-ingress \
	--group-id $SECURITY_GROUP_ID \
	--protocol -1 \
	--cidr 192.168.0.0/16

# TODO: add 8080 all access
aws ec2 authorize-security-group-ingress \
	--group-id $SECURITY_GROUP_ID \
	--protocol tcp \
	--port 8080 \
	--cidr 0.0.0.0/0

Give public IP to each worker node? helps with services, How do service endpoints work?

export AMI_ID=ami-0c2aa81b

# spawn a master
aws ec2 run-instances \
        --image-id $AMI_ID \
	--count 1 \
	--instance-type t2.medium \
	--key-name devops.local \
	--security-group-ids $SECURITY_GROUP_ID \
	--subnet-id $SUBNET1_ID \
	--associate-public-ip-address

# spawn two worker node instances
#    image-id is ubuntu 14:04
aws ec2 run-instances \
        --image-id $AMI_ID \
	--count 1 \
	--instance-type t2.medium \
	--key-name devops.local \
	--security-group-ids $SECURITY_GROUP_ID \
	--subnet-id $SUBNET1_ID \
	--associate-public-ip-address

aws ec2 run-instances \
	--image-id $AMI_ID \
	--count 1 \
	--instance-type t2.medium \
	--key-name devops.local \
	--security-group-ids $SECURITY_GROUP_ID \
	--subnet-id $SUBNET2_ID \
	--associate-public-ip-address
