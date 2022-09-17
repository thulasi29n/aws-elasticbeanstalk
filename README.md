# aws-elasticbeanstalk

Implementation of a Scalable Web Application using the services of AWS Elastic Beanstalk, DynamoDB, CloudFront and Edge Location
--------------------------------------------------------------------------------
Problem Statement:

An application that needs to support the high demand of large number of users accessing application simultaneously.
This application will be used in a large conference that had more than 10,000 people, in-person and online, with participants from all over the world.
This event will be a broadcast online and in person,more than 10,000 people in the audience will register their e-mails to guarantee their participation in the event.
-------------------------------------------------------------------------------
High level Solution:

We used in AWS, Elastic Beanstalk services to deploy the web application, DynamoDB and to store emails, CloudFront to cache static and dynamic files in an Edge Location close to the user.
------------------------------------------------------------------------
solution -architecture
![sol-arch](https://user-images.githubusercontent.com/26733874/190853290-09175829-26a0-4c38-8b22-49164c97b3ba.png)


**Implementation Steps:** (Planning document is attached in the project- Can be used for reference)

1. Create DynamoDB 
     -- name : users
     -- primary key : email

2. In Elastic BeanStalk 
      2.1) platform : python 
      2.2) Upload Application code from S3.(if you have any working web app, that can be used. My version of app available in repo) 
      2.3) preset config -- High Availablity 
      2.4) Software,Instance,capacity -- you can provide details as you require. ( refer xlsx for more reference) 
      2.5) Security -- Create a ssh key and select the one you created. 
      2.6) Networking -- Select defualt VPC and Subnets. 
      2.7) Auto scaling -- choose auto scaling params as "CPU Utilization"- "Percentage" - "1 min Duration" - "50-max" - "40-min" 
      2.8) Load Balancer -- choose settings as default 
      2.9) create APP.

3. Once the deploy completes, access the application and simulate a record insertion -- you will encounter error.

4. Investigate error logs -- ( you will most probably get Access denied exception)

5. Fix Role Issue 
     5.1) IAM Role -- Create a new role and assign permission 'AmazonDynamoDBFullAccess'. 
        ( This name of the role can be copied bean syalk instance and can be mapped) (refer screenshots pdf for more info)

6. Try to insert the registry once again. It should go through with out any issues this time.

7. Create Cloud Front Distribution 
     7.1) Allowed HTTP methods: GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE

8. SSH into EC2 instance --> ssh ec2-user@<PUBLIC_IP> -i <ssh_private_key> 
     8.1) Install stress tool to run stress test sudo amazon-linux-extras install epel -y sudo yum install stress -y 
     8.2) run stress test --> stress -c 4 
     8.3) check instance --> you will see 100% CPU usage.

9. Explore the resources created by the AWS Elastic Beanstalk (EC2, ELB, Auto Scaling Group, Cloud Watch resources) and 
      also the auto scaling process ---> Observe the instance status in BeanStalk 
   9.1) instance which is in stress whill show as degraded in the monitoring. 
   9.2) new instance will be created and added to autoscaling group. 
   9.3) newly created instance will be attached to load balancer.

10. Test Application --> you will not notice any change in the response.

11. SSH into same EC2 instance as in step 8 --> kill PID of the stress test.

12. you will now observe, Bean Stalk removing 3rd instance and LB having 2 instances.

13. Once you finish exploring it, 
    13.1) Please remove the Elastic Beanstalk application 
    13.2) Elastic Beanstalk environment 
    13.3) Disable and delete the CloudFront distribution and 
    13.4) Finally delete the DynamoDB users table
