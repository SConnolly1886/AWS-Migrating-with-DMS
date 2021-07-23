# Migrating the Wordpress Content
The on-premises web server has currently been set up with a sample web page. The database for wordpress is on the on-premises database server but we will deal with that later. Using the public IPv4 of the on-premises web server it should display the sample page provided. 

Lovely!

![wp1](https://user-images.githubusercontent.com/62077185/126720106-ab2dfca9-8cc1-4302-9ab9-057e6969f287.JPG)


### Changes on AWS Web Server
Now in order to transfer over the files to AWS, SCP will be used. The first thing you need to do is connect into the AWS Web Server. This can be done with instance connect to simulate a SSH connection. The SSH config file needs to be edited. This can be found at `/etc/ssh/sshd_config`. 
Use whatever editor you prefer to edit the file. (I prefer VI). 

Edit the `/etc/ssh/sshd_config` file and change:
- `PasswordAuthentication no` to `#PasswordAuthentication no`
- `#PasswordAuthentication yes` to `PasswordAuthentication yes`

Save the file. Great! This will allow for temporary password authentication. **Note the AWS Web Server security group allows connections on SSH port 22**

Next, run the command `passwd ec2-user`. When prompted for a password use the `DBPassword` provided in the CloudFormation template.

Now restart the SSHD service to put those changes into affect. Run the command  `systemctl restart sshd`

### Changes on On-Premise Web Server 
Connect to the simulated on-prem web server using SSM Session Manager. Run `sudo bash` for elevated priviledges. 

Move to the web folder `cd /var/www/`

Go to the CloudFormation outputs and copy the PrivateIP of the AWS web server and use it in the command below

Run `scp -rp html ec2-user@PRIVATEIPOFTHEAWSWEBSERVER:/home/ec2-user` answer `yes` to the authenticity warning. If successful, it would have transferred over the wordpress files on the on-prem server to the AWS web server. 


### Now back to the AWS Web Server
Connect to the instance again if you haven't closed the old connection. Escalate priviledges and move to the `/home/ec2-usr` directory where the files were copied to. 

Run `ls -la` to check for the html directory and the run `cd html`. Great now copy all of those files to the webroot folder using the following command:
`cp * -R /var/www/html/`

Now all that needs to be done is to change the permissions on the files you've moved. Run the commnands below to change the permissions:
```
usermod -a -G apache ec2-user   
chown -R ec2-user:apache /var/www
chmod 2775 /var/www
find /var/www -type d -exec chmod 2775 {} \;
find /var/www -type f -exec chmod 0664 {} \;
```

Great now if everything is working as it should....use  the public IPv4 on the AWS Web Server to ensure the sample web page has been copied!

![wp1](https://user-images.githubusercontent.com/62077185/126720106-ab2dfca9-8cc1-4302-9ab9-057e6969f287.JPG)

Fantastic! On to the next step...

