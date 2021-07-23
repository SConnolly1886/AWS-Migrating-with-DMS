# Migrating the Database server using AWS Database Migration Service (DMS).
An RDS instance was created using the CloudFormation template so these next steps will be that much easier! All that needs to be done is to create a DMS Subnet group, create a replication instance (smaller instance for example), DMS source endpoint, DMS destination endpoint, migrate, and cutover the app instance!


![migration](https://user-images.githubusercontent.com/62077185/126720099-a1156139-2cb0-4d1d-bbd9-b514aaa1ed6f.png)

## Step 1 - Create the DMS Subnet Group
In the AWS DMS console select the following:
- `Create Subnet Group`
- name `DMSSubnetGroup`
- description `DMS Subnet Group`
- for vpc select `VPC`
- `Add Subnets` add `DBSubnetA`, `DBSubnetB`, `DBSubnetC`
- `Create subnet group` 

Great! The DB subnets have been selected as DMS Subnets. 

## Step 2 - Create the replication instance
In the DMS Console:
- `create Replication Instance` 
- name `ONPREMAWS`
- description `migrating db from on-prem to aws`
- `Instance Class` choose `dms.t2.micro` 
- for `VPC` select `VPC` 
- uncheck `MultiAZ` and `Publicly Accessing`  
- ensure `DMSSubnetGroup` is selected in `Replication subnet group`  
- security group select `SGDatabase`
- `Create`

Now for the endpoints!

## Step 3 - DMS Source Endpoint (On-Prem DB)
- `Create Endpoint` 
- select `Source Endpoint` for `Endpoint type` 
- `Endpoint configuration` set `Endpoint identifier` to `DBOnpremises`
- `Source Engine` set `mariadb`
- `Server name` use the privateIPv4 address of `onpremDB` (CloudFormation outputs)
- port `3306` (MariaDB)
- username `wordpressdb`
- DBPassword for password
- `Create endpoint`  

# Step 4 - - DMS Destination Endpoint (RDS)
- `Create Endpoint`
- select `Target Endpoint` for `Endpoint type`   
- select `RDS DB Instance` 
- select  `wordpressdb` in dropdown
- DBPassword for password
- `Create endpoint` 

Great the endpoints have been created. Now all that's left is the migration!

## Step 5 - Migrating the DB
Move to `migration tasks`
- `Create task` 
- `Task identifier` enter `ONPREMTOAWSWORDPRESS`
- `Replication instance` pick the replication instance you just created 
- `Source database endpoint` select `DBOnpremises`
- `Target database endpoint` select `wordpressdb`  
- `Migration type` select `migrate existing data` (small amount of data, no need to pick and choose)
- `Table mappings` select `Guided UI`  
- `Add new selection rule` 
- in `Schema` box select `Enter a Schema`
- in `Schema Name` type `wordpressdb`
- `Create Task`  

Great! This will start the replication. It will do a full load from the `DBOnpremises` (due to small size) to the RDS instance.
This will run until you see `Load complete`  

And just like that the data has been migrated to the AWS RDS instance. 

## Step 6 - Cutover the application instance from On-Prem to AWS
In RDS Console:
select `Databases` -->  `a4lwordpressdb` 
Under `Endpoint & Port` copy the `endpoint` dns name.

Connect back to the AWS Web Server instance
Escalate priviledges and go to the directory `cd /var/www/html`
Now we will need to edit the `wp-config.php` to change the DB Host

Using an editor (VI) of your choice edit the `wp-config.php` file. Replace the DB_HOST with the IP address of the RDS host you just copied. Save and exit. 

Great now only a few more changes need to be made and this will be done by running the script below. It's similar to the script used in the pre-baked on-prem web server ami. 
This script will update the wordpress db with the new DNS name. 

```
#!/bin/bash
source <(php -r 'require("/var/www/html/wp-config.php"); echo("DB_NAME=".DB_NAME."; DB_USER=".DB_USER."; DB_PASSWORD=".DB_PASSWORD."; DB_HOST=".DB_HOST); ')
SQL_COMMAND="mysql -u $DB_USER -h $DB_HOST -p$DB_PASSWORD $DB_NAME -e"
OLD_URL=$(mysql -u $DB_USER -h $DB_HOST -p$DB_PASSWORD $DB_NAME -e 'select option_value from wp_options where option_id = 1;' | grep http)
HOST=$(curl http://169.254.169.254/latest/meta-data/public-hostname)
$SQL_COMMAND "UPDATE wp_options SET option_value = replace(option_value, '$OLD_URL', 'http://$HOST') WHERE option_name = 'home' OR option_name = 'siteurl';"
$SQL_COMMAND "UPDATE wp_posts SET guid = replace(guid, '$OLD_URL','http://$HOST');"
$SQL_COMMAND "UPDATE wp_posts SET post_content = replace(post_content, '$OLD_URL', 'http://$HOST');"
$SQL_COMMAND "UPDATE wp_postmeta SET meta_value = replace(meta_value,'$OLD_URL','http://$HOST');"
```

Amazing! Now to make sure everything is running as it should stop both of the simulated on-prem servers. Open the IPv4 address of the AWS Web Server and...

![wp1](https://user-images.githubusercontent.com/62077185/126720106-ab2dfca9-8cc1-4302-9ab9-057e6969f287.JPG)

It works! In a few steps an on-premises database has been migrated to AWS

# Finished 
The CloudFormation template made things a lot easier. Check out the diagram of the finished setup below. 

![finallayout](https://user-images.githubusercontent.com/62077185/126720096-5e000103-9077-43f4-b812-bf0bb8065600.png)

All you need to do is delete the stack to undo all the changes you've made. Enjoy!



