# awswebserver
AWS. basic site infrastructure (Wordpress): use VPC, EC2, RDS services


- AWS - use free tier for your account.
- Please create basic site infrastructure (Wordpress): use VPC, EC2, RDS services.

1) Creating Your Key Pair Using Amazon EC2
test@vvv:~/aws/WP$ aws ec2 create-key-pair --key-name MyKeyPair
mv MyKeyPair MyKeyPair.pem
chmod 400 MyKeyPair.pem

2) Amazon EC2 and Amazon Virtual Private Cloud
http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-vpc.html
Amazon Virtual Private Cloud (Amazon VPC) enables you to define a virtual network in your own logically isolated area within the AWS cloud, known as a virtual private cloud (VPC).

create a VPC     http://docs.aws.amazon.com/cli/latest/reference/ec2/create-vpc.html

aws ec2 create-vpc --cidr-block 10.0.0.0/16

VPC 10.0.0.0/16 dopt-e774968f default False pending vpc-d39841bb

create-subnet http://docs.aws.amazon.com/cli/latest/reference/ec2/create-subnet.html

aws ec2 create-subnet --vpc-id vpc-d39841bb --availability-zone eu-central-1b --cidr-block 10.0.1.0/24  web
aws ec2 create-subnet --vpc-id vpc-d39841bb --availability-zone eu-central-1a --cidr-block 10.0.2.0/24  web
aws ec2 create-subnet --vpc-id vpc-d39841bb --availability-zone eu-central-1a --cidr-block 10.0.3.0/24  db

look at result
aws ec2 describe-subnets --subnet-ids subnet-e9e44793
SUBNET False eu-central-1b 251 10.0.1.0/24 False False  available subnet-eef25194 vpc-d39841bb                  web
SUBNETS False eu-central-1a 251 10.0.2.0/24 False False available subnet-e1ba2489 vpc-d39841bb               web
SUBNETS False eu-central-1a 251 10.0.3.0/24 False False available subnet-e548d68d vpc-d39841bb               db

delete a subnet http://docs.aws.amazon.com/cli/latest/reference/ec2/delete-subnet.html
aws ec2 delete-subnet --subnet-id subnet-9d4a7b6c
aws ec2 delete-subnet --subnet-id subnet-e9e44793

3) create a security group

create a security group for EC2-Classic
aws ec2 create-security-group --group-name MySecurityGroup --description "My security group"
aws ec2 create-security-group --group-name WEBSG --description "WEB security group" --vpc-id vpc-d39841bb
sg-91796af9
aws ec2 create-security-group --group-name DBSG --description "DB security group" --vpc-id vpc-d39841bb
sg-bf796ad7

create a security group for EC2-VPC
aws ec2 create-security-group --group-name MySecurityGroup --description "My security group" --vpc-id vpc-d39841bb

add a rule
EC2-VPC] To add a rule for WEBSG that allows inbound SSH, http, https traffic
inbound
aws ec2 authorize-security-group-ingress --group-id sg-91796af9 --protocol tcp --port 22 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-id sg-91796af9 --protocol tcp --port 80 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-id sg-91796af9 --protocol tcp --port 443 --cidr 0.0.0.0/0
outbound
add a rule for WEBSG outbond 3306 from DBFG, 80, 443
aws ec2 authorize-security-group-egress --group-id sg-91796af9 --protocol tcp --port 3306 --source-group sg-bf796ad7
aws ec2 authorize-security-group-egress --group-id sg-91796af9 --protocol tcp --port 80 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-egress --group-id sg-91796af9 --protocol tcp --port 443 --cidr 0.0.0.0/0

add a rule for DBSG
aws ec2 authorize-security-group-ingress --group-id sg-bf796ad7 --protocol tcp --port 3306 --source-group sg-91796af9
aws ec2 authorize-security-group-egress --group-id sg-bf796ad7 --protocol tcp --port 80 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-egress --group-id sg-bf796ad7 --protocol tcp --port 443 --cidr 0.0.0.0/0

EC2-VPC] To add a rule that allows inbound HTTP traffic from another security group
aws ec2 authorize-security-group-ingress --group-id sg-3eebfb56 --protocol tcp --port 80 --source-group sg-1a2b3c4d
EC2-VPC] To add a custom ICMP rule
aws ec2 authorize-security-group-ingress --group-id sg-3eebfb56 --ip-permissions '[{"IpProtocol": "icmp", "FromPort": 3, "ToPort": 4, "IpRanges": [{"CidrIp": "0.0.0.0/0"}]}]'

EC2-VPC] To delete a security group
aws ec2 delete-security-group --group-id sg-903004f8

Security Group Rules
web server

Protocol type

Protocol number

Port

Source IP

Notes

TCP

6

80 (HTTP)

0.0.0.0/0

Allows inbound HTTP access from any IPv4 address

TCP

6

443 (HTTPS)

0.0.0.0/0

Allows inbound HTTPS access from any IPv4 address

Database server

Protocol type

Protocol number

Port

Notes

TCP

6

1433 (MS SQL)

The default port to access a Microsoft SQL Server database, for example, on an Amazon RDS instance

TCP

6

3306 (MYSQL/Aurora)

The default port to access a MySQL or Aurora database, for example, on an Amazon RDS instance

TCP

6

5439 (Redshift)

The default port to access an Amazon Redshift cluster database.

TCP

6

5432 (PostgreSQL)

The default port to access a PostgreSQL database, for example, on an Amazon RDS instance

TCP

6

1521 (Oracle)

The default port to access an Oracle database, for example, on an Amazon RDS instance

Access from another instance in the same group

Protocol type

Protocol number

Ports

Source IP

-1 (All)

-1 (All)

-1 (All)

The ID of the security group

create-route-table

aws ec2 create-route-table --vpc-id vpc-d39841bb

ROUTETABLE rtb-dece4eb6 vpc-d39841bb
ROUTES 10.0.0.0/16 local CreateRouteTable active

create-internet-gateway

aws ec2 create-internet-gateway

INTERNETGATEWAY igw-29efc940
attach internet gateway
aws ec2 attach-internet-gateway --vpc-id vpc-d39841bb --internet-gateway-id igw-29efc940

create a route-table
aws ec2 create-route-table --vpc-id vpc-d39841bb

create a route

aws ec2 create-route --route-table-id rtb-f43dbc9c --destination-cidr-block 0.0.0.0/0 --gateway-id igw-29efc940

see  the result
aws ec2 describe-route-tables --route-table-id rtb-f43dbc9c

choose which subnet to associate with the custom route table
aws ec2 associate-route-table  --subnet-id subnet-e1ba2489 --route-table-id rtb-f43dbc9c
aws ec2 associate-route-table  --subnet-id subnet-eef25194 --route-table-id rtb-f43dbc9c

modify the public IP addressing behavior of your subnet so that an instance launched into the subnet automatically receives a public IP address
aws ec2 modify-subnet-attribute --subnet-id subnet-e1ba2489 --map-public-ip-on-launch
aws ec2 modify-subnet-attribute --subnet-id subnet-eef25194 --map-public-ip-on-launch

4)RDS
create db subnet

aws rds create-db-subnet-group --db-subnet-group-name dbsubnet --db-subnet-group-description dbsubnetdescription --subnet-ids subnet-e548d68d subnet-e1ba2489

creates a MySQL db instance named mydbinstance.
For Linux
aws rds create-db-instance --db-name wp --db-instance-identifier mydbinstance --db-instance-class db.t2.micro --db-security-groups sg-bf796ad7 --vpc-security-group-ids sg-bf796ad7  --engine MySQL --allocated-storage 5 --master-username masterawsuser  --master-user-password masteruserpassword --backup-retention-period 3

"MasterUsername": "masterawsuser",
"VpcSecurityGroupId": "sg-19726171"
PreferredBackupWindow": "00:31-01:01"
SubnetIdentifier": "subnet-a5ea75cd
Name": "eu-central-1a
SubnetIdentifier": "subnet-0b1cbd71
Name": "eu-central-1b
VpcId": "vpc-4d78a725
DbiResourceId": "db-HPNT5U2JDECVZ4JWV73N3O6WFM
CACertificateIdentifier": "rds-ca-2015
DBInstanceIdentifier": "mydbinstance

delete db instance
aws rds delete-db-instance --db-instance-identifier mydbinstance --skip-final-snapshot


  create-db-instance
  [--db-name <value>]
  --db-instance-identifier <value>
  [--allocated-storage <value>]
  --db-instance-class <value>
  --engine <value>
  [--master-username <value>]
  [--master-user-password <value>]
  [--db-security-groups <value>]
  [--vpc-security-group-ids <value>]
  [--availability-zone <value>]
  [--db-subnet-group-name <value>]

  aws rds create-db-subnet-group --db-subnet-group-name dbsubnetgroup --db-subnet-group-description dbsubnetgroupdescription  --subnet-ids subnet-eef25194 subnet-e1ba2489

test@vvv:~/aws/WP$ aws rds describe-db-subnet-groups
DBSUBNETGROUPS arn:aws:rds:eu-central-1:198573302643:subgrp:dbsubnetgroup dbsubnetgroupdescription dbsubnetgroup Complete vpc-d39841bb
SUBNETS subnet-e1ba2489 Active
SUBNETAVAILABILITYZONE eu-central-1a
SUBNETS subnet-eef25194 Active
SUBNETAVAILABILITYZONE eu-central-1b
DBSUBNETGROUPS arn:aws:rds:eu-central-1:198573302643:subgrp:default defaultdefault Complete vpc-4d78a725
SUBNETS subnet-a5ea75cd Active
SUBNETAVAILABILITYZONE eu-central-1a
SUBNETS subnet-0b1cbd71 Active
SUBNETAVAILABILITYZONE eu-central-1b

5) RUN INSTANCE
aws ec2 run-instances --image-id ami-fe408091 --instance-type t2.micro --count 1 --key-name MyKeyPair  --security-group-ids sg-3eebfb56 --subnet-id  subnet-eef25194 --private-ip-address 10.0.1.10
--security-groups  sg-3eebfb56

aws ec2 run-instances --image-id ami-fe408091 --count 1 --instance-type t2.micro --key-name MyKeyPair --user-data file://~/aws/userdata/puppet --security-group-ids sg-8fdecee7--subnet-id subnet-8f9907e7

userdata
apt install apache2 apache2-utils
systemctl enable apache2
systemctl start apache2
apt install php7.0 php7.0-mysql libapache2-mod-php7.0 php7.0-cli php7.0-cgi php7.0-gd
echo " <?php phpinfo(); ?> " > /var/www/html/info.php
wget -c http://wordpress.org/latest.tar.gz
tar -xzvf latest.tar.gz
rsync -av wordpress/* /var/www/html/
chown -R www-data:www-data /var/www/html/
chmod -R 755 /var/www/html/
mysql -u root -p
mysql> CREATE DATABASE wp_database;
mysql> GRANT ALL PRIVILEGES ON wp_database.* TO 'wp_user'@'localhost' IDENTIFIED BY 'password';
mysql> FLUSH PRIVILEGES;
mysql> EXIT;
mv wp-config-sample.php wp-config.php

vi wp-config.php
/** Имя базы данных для WordPress */
define('DB_NAME', 'database_name_here');
/** Имя пользователя MySQL */
define('DB_USER', 'username_here');
/** Пароль пользователя MySQL */
define('DB_PASSWORD', 'password_here');
/** MySQL хост */
define('DB_HOST', 'localhost');
/** Кодировка по умолчанию для базы данных */
define('DB_CHARSET', 'utf8');

systemctl restart apache2.service
systemctl restart mysql.service

- создал ключи
- запустил инстанс убунту
- подключиться по ssh

установить софт при старте инстанса
прокинуть файлы с локального каталога в каталог инстанса
синхронизация файлов с гита по расписанию или событию

adminiam     accesskey.csv

VPC     vpc-ff6eb097 10.17.0.0/16
testsubnet1     10.17.0.0/16     subnet-8f9907e7
network acl     acl-6008b008
security group      sg-8fdecee7
