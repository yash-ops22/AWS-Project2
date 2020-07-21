# Task Overview

1. Create Security group which allow the port 80.
2. Launch EC2 instance.
3. In this Ec2 instance use the existing key or provided 
  key and security group which we have created in step 1.
4. Launch one Volume using the EFS service and attach it 
  in your vpc, then mount that volume into /var/www/html
5. Developer have uploded the code into github repo also 
   the repo has some images.
6. Copy the github repo code into /var/www/html
7. Create S3 bucket, and copy/deploy the images from github 
  repo into the s3 bucket and change the permission to
  public readable.
8. Create a Cloudfront using s3 bucket(which contains images)
and use the Cloudfront URL to  update in code in /var/www/html

# Solution

# Step 1:
First as mentioned we have to create a Security Group which
will allow port 80, also we have included port 22 to login into 
our instance.

    resource "aws_security_group" "sgcloud2" {

                name        = "sgcloud2"
           

                ingress {

                  from_port   = 80
                  to_port     = 80
                  protocol    = "tcp"
                  cidr_blocks = [ "0.0.0.0/0"]

                }

              
                ingress {

                  from_port   = 22
                  to_port     = 22
                  protocol    = "tcp"
                  cidr_blocks = [ "0.0.0.0/0"]

                }




                egress {

                  from_port   = 0
                  to_port     = 0
                  protocol    = "-1"
                  cidr_blocks = ["0.0.0.0/0"]
                }


                tags = {

                  Name = "sgcloud2"
                }
              }

# Step 2:
