# AWS-Migrating-with-DMS
Migrating a simulated on-premises database server using AWS Database Migration Service

## Scenario
There is a web and database server in an on premises location and you'd like to migrate to AWS. Migrating to AWS is made easy with AWS Database Migration Service (DMS). The web server in this case will be running a wordpress instance and the wordpress database is initially directed to the on-premises database server. SSH secure copy (SCP) will be used to transfer the wordpress web files to and AWS instance and DMS will be used to migrate the on-premises database to an RDS instance. 

![migrationcut](https://user-images.githubusercontent.com/62077185/126720100-359b9789-5010-4f52-ab4e-657ca6e92317.png)

## Building initial environment
For the sake of time a Cloudformation template will be provided to speed things along. In this example a simulated on-premises environment will be created (different vpc). A vpc peering connection will be created to simulate a VPN connection. If you'd like to go through the steps of creating a VPN check out my other repo [AWS-Site-to-Site-VPN](https://github.com/SConnolly1886/AWS-Site-to-Site-VPN). There are 2 pre-baked amis that will represent the simulated on-premises servers. On the AWS side a wordpress instance and rds instance are created. Be patient as this will take a while to provision. More subnets than you require have been created if you'd like to go the extra mile and make it highly available. Check out my repo [AWS-Monolith-2-Fault-Tolerant](https://github.com/SConnolly1886/AWS-Monolith-2-Fault-Tolerant) for more info.



The template will create:
### Simulated On-Premises Environment
- vpc
- public subnet
- wordpress instance for the web server
- database server (mariadb)
- security groups for web server, db servers 
- route table for public subnet
- IAM role need for SSM

### AWS Environment
- vpc
- 3 public, 6 private subnets
- 3 availability zones (us-east-1)
- routes tables/assocations 
- vpc peering connection
- vpc peering routes
- security groups needed for [AWS-Monolith-2-Fault-Tolerant](https://github.com/SConnolly1886/AWS-Monolith-2-Fault-Tolerant)
- wordpress instance
- rds mariadb instance

### Outputs
The CloudFormation template will create the following outputs that you will need along the way:
- OnPremDBPrivateIP: Private IP of OnPrem DB
- DBName: WordPress database name
- DBPassword: WordPress database admin account password
- DBRootPassword: MySQL root password
- DBUser: WordPress database admin account username
- AWSWebPrivateIP: Private IP of AWS Web Server

Again, be patient. The RDS instance will take up some time. Once everything is created your environment should look like this

![migrationinitial](https://user-images.githubusercontent.com/62077185/126720104-b0ac8bb4-f4c1-46ca-b8c5-ca04b2e9fd81.png)

Great! Now on to the next step. Migrating the on-premises web server info to the AWS instance using SCP. <br/>
[Migrating-Wordpress-Content](https://github.com/SConnolly1886/AWS-Migrating-with-DMS/blob/main/AWS-Migrating-with-DMS-2.md)
