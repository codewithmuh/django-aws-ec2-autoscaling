# Django AWS EC2 Auto Scaling Deployment

This repository contains scripts and instructions for deploying a Django project on AWS EC2 with Auto Scaling and Load Balancer, along with other essential components such as RDS for database management, SSL certificate setup, Route53 configuration, and connecting RDS with PgAdmin for database management.

![1i](https://github.com/codewithmuh/django-aws-ec2-autoscaling/assets/51082957/8042ab6b-e1fb-46b6-b9cb-fdd22d05291e)

## Prerequisites

Before you begin, ensure you have the following:

- An AWS account with appropriate permissions to create EC2 instances, RDS databases, Load Balancers, etc.
- A Django project ready to be deployed.
- Basic understanding of AWS services and deployment concepts.

## Steps Covered in Video:

1. Project Setup with Nginx and Gunicorn
2. WhiteNoise and Gunicorn Concept
3. AWS RDS Database Setup and its security group
4. Pgadmin setup, Connect PgAdmin with RDS To view the database (Optional)
5. Running Project Locally To make sure No bugs.
6. Create an ec2 Template with metadata, keeping the pem file for future use.
7. create an autoscaling group with a load balancer, target group, security group
8. View the application In the browser at load balancer DNS
9. SSH explanation if required for future Code updates.
10. Domain Registration at AWS Route53
11. SSL cert, AWS Certificate manager, attaching to load balancer
12. What Next 
# Complete Youtube Video Tutorial:


This project is licensed under the MIT License, granting you the freedom to use, modify, and distribute the code.

## Acknowledgements
Special thanks to codewithmuh for creating this incredible Django Starter Kit and simplifying the development process.

## Support
<a href="https://www.buymeacoffee.com/codewithmuh" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/default-yellow.png" alt="Buy Me A Coffee" height="41" width="174"></a>


