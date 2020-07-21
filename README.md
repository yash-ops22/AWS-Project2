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
Launching our instance using the above created security
group and using the key we had created earlier. Also, installing 
the httpd server into it so that we can launch our site....


      resource "aws_instance" "aws-os" {
                        ami             =  "ami-0732b62d310b80e97"
                        instance_type   =  "t2.micro"
                        key_name        =  "cloudkey2"
                        subnet_id       =  "subnet-d26d68ba"
                        security_groups = ["${aws_security_group.sgcloud2.id}"]


                       connection {
                          type     = "ssh"
                          user     = "ec2-user"
                          private_key = file("C:/Users/win 10/Downloads/cloudkey2.pem")
                          host     = aws_instance.aws-os.public_ip
                        }

                        provisioner "remote-exec" {
                          inline = [
                            "sudo yum install amazon-efs-utils -y",
                            "sudo yum install httpd  php git -y",
                            "sudo systemctl restart httpd",
                            "sudo systemctl enable httpd",
                            "sudo setenforce 0",
                            "sudo yum -y install nfs-utils"
                          ]
                        }

                        tags = {
                          Name = "osaws"
                        }
                      }

     

# Step 3:
We have to create an volume or storage,  In our first
aws task we had used EBS, here we are using EFS.
Creating the EFS Volume......

     resource "aws_efs_file_system" "amazonefs" {
       creation_token = "my-cloudefs"

       tags = {
         Name = "amazonefs"
       }
     }

     resource "aws_efs_mount_target" "efsmount" {
        file_system_id = aws_efs_file_system.amazonefs.id
        security_groups = [aws_security_group.sgcloud2.id]
        subnet_id      =   "subnet-d26d68ba"

     }

We cannot use this storage directly, so Mounting this EFS to our 
instance that we have launched, mounting this EFS into directory
/var/www/html of the httpd server.
Also, cloning the gihthub repo into the /var/www/html folder
and downloading the files into it.......

     
    resource "null_resource" "mount"  {
                depends_on = [aws_efs_mount_target.efsmount]
                connection {
                type     = "ssh"
                user     = "ec2-user"
                private_key = file("C:/Users/win 10/Downloads/cloudkey2.pem")
                host     = aws_instance.aws-os.public_ip
              }
              provisioner "remote-exec" {
                  inline = [
                    "sudo mount -t nfs -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport ${aws_efs_file_system.amazonefs.id}.efs.ap-south-         1.amazonaws.com:/ /var/www/html",
                       "sudo rm -rf /var/www/html/*",
                       "sudo git clone https://github.com/yash-ops22/awscloud2.git /var/www/html/",
                       "sudo sed -i 's/url/${aws_cloudfront_distribution.amazoncloudfront.domain_name}/g' /var/www/html/index.html"
                        ]
                    }
                  }


# Step 4:
Creating the S3 bucket, so that we can upload files or
images into it.......

     resource "aws_s3_bucket" "buckets3yashucloud22" {
        bucket = "yashu22"
        acl    = "private"

        tags = {
          Name        = "buckets3"
        }
      }
       locals {
          s3_origin_id = "myS3Origin"
        }
        
Uploading the images into our S3 bucket and making it
publically accesible....
     
     resource "aws_s3_bucket_object" "objectawsbucket" {
          bucket = "${aws_s3_bucket.buckets3yashucloud22.id}"
        key    = "picture"
          source = "C:/Users/win 10/Downloads/aws1.jpg"
          acl    = "public-read"
        } 

# Step 5:


