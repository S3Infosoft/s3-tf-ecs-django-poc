# s3-tf-ecs-django-poc
# Project Setup

Let's start by setting up a quick Django project.
Create a new project directory along with a new Django project:

$ mkdir django-ecs-terraform && cd django-ecs-terraform
$ mkdir app && cd app
$ python3.8 -m venv env
$ source env/bin/activate

(env)$ pip install django==3.1
(env)$ django-admin.py startproject hello_django .
(env)$ python manage.py migrate
(env)$ python manage.py runserver

Navigate to http://localhost:8000/ to view the Django welcome screen. Kill the server once done, and then exit from the virtual environment. Go ahead and remove it as well. We now have a simple Django project to work with.

ADD requirements.txt and Dockerfile in the app directory

# Deploying Django to AWS ECS with Terraform

1. Install terraform
2. Sign up for an AWS account
3. Create two ECR repositories, django-app and nginx
4. Create a terraform folder in the project root directory:
Add the terraform files which Sets up the following AWS infrastructure:

Networking:
   VPC
   Public and private subnets
   Routing tables
   Internet Gateway
   Key Pairs
Security Groups
Load Balancers, Listeners, and Target Groups
IAM Roles and Policies
ECS:
   Task Definition (with multiple containers)
   Cluster
   Service
Launch Config and Auto Scaling Group
RDS
Health Checks and Logs

Add a new folder in "terraform" called "policies". Then, add the following role and policy definitions
ecs-role.json
ecs-instance-role-policy.json
ecs-service-role-policy.json
Add a "templates" folder to the "terraform" folder, and then add a new template file called django_app.json.tpl

5. For Django Health Check

Add the middleware to app/hello_django/middleware.py
Add the class to the middleware config in settings.py:
'hello_django.middleware.HealthCheckMiddleware',  # new

6. Update python files in the app:

Add DATABASES in the settings.py file

Set the STATIC_ROOT in settings.py file

6. Create a nginx folder in the project root directory: Add Dockerfile related to nginx and nginx.config files

7. Create a "deploy" folder in the project root. Then, add an update-ecs.py file to that newly created folder.Create and activate a new virtual environment. Then, install Boto3 and Click. Run the script like so:
python update-ecs.py --cluster=production-cluster --service=production-service

8. Build the Django and Nginx Docker images and push them up to ECR:

$ cd app
$ docker build -t <AWS_ACCOUNT_ID>.dkr.ecr.us-west-1.amazonaws.com/django-app:latest .
$ docker push <AWS_ACCOUNT_ID>.dkr.ecr.us-west-1.amazonaws.com/django-app:latest
$ cd ..

$ cd nginx
$ docker build -t <AWS_ACCOUNT_ID>.dkr.ecr.us-west-1.amazonaws.com/nginx:latest .
$ docker push <AWS_ACCOUNT_ID>.dkr.ecr.us-west-1.amazonaws.com/nginx:latest
$ cd ..

9.Update the variables in terraform/variables.tf.

10.Set the following environment variables, init Terraform, create the infrastructure:

$ cd terraform
$ export AWS_ACCESS_KEY_ID="YOUR_AWS_ACCESS_KEY_ID"
$ export AWS_SECRET_ACCESS_KEY="YOUR_AWS_SECRET_ACCESS_KEY"

$ terraform init
$ terraform apply
$ cd ..

11.Terraform will output an ALB domain

12.Open the EC2 instances overview page in AWS. Use ssh ec2-user@<ip> to connect to the instances until you find one for which docker ps contains the Django container. Run docker exec -it <container ID> python manage.py migrate.

13.Now we can see app in ALB domain http://your.domain.com/admin

14.You can also run the following script to bump the Task Definition and update the Service:

$ cd deploy
$ python update-ecs.py --cluster=production-cluster --service=production-service



