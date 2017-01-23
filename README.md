- AWS - use free tier for your account.
- Please create basic site infrastructure (Wordpress): use VPC, EC2, RDS services.

http://35.157.92.252/wp-admin/install.php

aws ec2 create-key-pair --key-name mykey1 --query 'KeyMaterial' --output text > mykey1.pem
chmod 400 mykey1.pem
aws ec2 create-vpc --cidr-block 10.0.0.0/16
aws ec2 create-subnet --vpc-id vpc-d39841bb --availability-zone eu-central-1b --cidr-block 10.0.1.0/24
aws ec2 create-subnet --vpc-id vpc-d39841bb --availability-zone eu-central-1a --cidr-block 10.0.2.0/24
aws ec2 create-subnet --vpc-id vpc-d39841bb --availability-zone eu-central-1a --cidr-block 10.0.3.0/24

aws ec2 create-security-group --group-name WEBSG --description "WEB security group" --vpc-id vpc-d39841bb
aws ec2 create-security-group --group-name DBSG --description "DB security group" --vpc-id vpc-d39841bb

aws ec2 authorize-security-group-ingress --group-id sg-91796af9 --protocol tcp --port 22 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-id sg-91796af9 --protocol tcp --port 80 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-id sg-91796af9 --protocol tcp --port 443 --cidr 0.0.0.0/0

aws ec2 authorize-security-group-egress --group-id sg-91796af9 --protocol tcp --port 3306 --source-group sg-bf796ad7
aws ec2 authorize-security-group-egress --group-id sg-91796af9 --protocol tcp --port 80 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-egress --group-id sg-91796af9 --protocol tcp --port 443 --cidr 0.0.0.0/0

aws ec2 authorize-security-group-ingress --group-id sg-bf796ad7 --protocol tcp --port 3306 --source-group sg-91796af9
aws ec2 authorize-security-group-egress --group-id sg-bf796ad7 --protocol tcp --port 80 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-egress --group-id sg-bf796ad7 --protocol tcp --port 443 --cidr 0.0.0.0/0

aws ec2 create-route-table --vpc-id vpc-d39841bb

aws ec2 create-internet-gateway

aws ec2 attach-internet-gateway --vpc-id vpc-d39841bb --internet-gateway-id igw-29efc940

aws ec2 create-route-table --vpc-id vpc-d39841bb

aws ec2 create-route --route-table-id rtb-f43dbc9c --destination-cidr-block 0.0.0.0/0 --gateway-id igw-29efc940

aws ec2 associate-route-table  --subnet-id subnet-e1ba2489 --route-table-id rtb-f43dbc9c
aws ec2 associate-route-table  --subnet-id subnet-eef25194 --route-table-id rtb-f43dbc9c

aws ec2 modify-subnet-attribute --subnet-id subnet-e1ba2489 --map-public-ip-on-launch
aws ec2 modify-subnet-attribute --subnet-id subnet-eef25194 --map-public-ip-on-launch
aws rds create-db-subnet-group --db-subnet-group-name dbsubnetgroup --db-subnet-group-description dbsubnetgroupdescription  --subnet-ids subnet-eef25194 subnet-e1ba2489 subnet-e548d68d

aws rds create-db-instance --db-name wp1 --db-instance-identifier mydbinstance1 --db-instance-class db.t2.micro --vpc-security-group-ids sg-bf796ad7  --engine MySQL --allocated-storage 5 --master-username masterawsuser  --master-user-password masteruserpassword --backup-retention-period 3 --db-subnet-group-name dbsubnetgroup

aws ec2 run-instances --image-id ami-fe408091 --instance-type t2.micro --count 1 --key-name mykey1  --security-group-ids sg-91796af9 --subnet-id  subnet-e1ba2489 --private-ip-address 10.0.2.10 --user-data file://~/aws/userdata


to connect to a server via ssh
ssh -i "mykey1.pem" ubuntu@35.157.92.252

mv wp-config-sample.php wp-config.php

vi wp-config.php
define('DB_NAME', 'wp1');
define('DB_USER', 'masterawsuser');
define('DB_PASSWORD', 'masteruserpassword');
define('DB_HOST', 'mydbinstance1.c9ifoub4fht7.eu-central-1.rds.amazonaws.com');
define('DB_CHARSET', 'utf8');
