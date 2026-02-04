💡Do not change the zone you are in.
# 3-tier-application-deployment
## VPC Creation

- Create a VPC.
- Create 2 subnets( Public and Private )
- Create 2 Route Table( Public and Private )
- Create Internet Gateway
- Create NAT Gateway
- Connect all the above in the Route Table.
- Create 4 Security Group
	-> Public (bastion)
	-> Private (Backend)
	-> Load Balancer (ALB)
	-> RDS (Database)
## Instance Creation
### Bastion Server

1. Create an Instance (bastion server/Frontend server ) with the created VPC and Public SG (bastion).
2. Create an Elastic IP and associate it with the bastion server.
3. Connect to the server and Install any web server, a sample is given below
```
sudo apt update -y
sudo apt install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx

Test:
http://<Elastic-ip>
```

### Backend Server

1. Create an Instance ( backend server ) with the created VPC and Private SG (backend).
2. Log in to the bastion server
3. Store your key-pair and add read permissions
4.  Check if the backend is accessible via ssh
```
chmod 400 filename.ext //modify permissions
ssh -i filename.pem ubuntu@your-ip
```

## Application Load Balancer(ALB)

-  Create a Target group with backend instance
⚠️ ALB should also be in the same region with at least two Availability Zones
-  Create the Application Load Balancer with the security Group

## RDS

1. Select Full configuration and select the engine ( MySQL or Aurora or PostgreSQL )
2. Select the Availability ( single instance or multi instance )
3. Enter username and password for your master DB.
4. Select the created VPC and RDS-sg Security Group.
5. Under Additional configuration give the name of the database in DB and create.
6. Note the Endpoint of the RDS.

## Deployment

### Backend
1. Go to your backend server and pull your backend code from GitHub.
2. Change the .env file with the Endpoint and the configurations of your RDS.
3. Install Docker in your machine.
4. Build an image of the backend.
5. Run the Backend as a Container in port 8080.
6. Verify it using the below command
```
docker ps
```

### Frontend
1. Go to your Bastion Server and install your package manager( npm )
2. Pull the frontend code from GitHub.
3. Add the endpoint of the ALB in your .env file.
4. Build your code and deploy it using nginx.


# Disaster Recovery

💡I am using Backup & restore.

| Frontend EC2 (Nginx) | Instance + config  | AMI + EBS snapshot            |
| -------------------- | ------------------ | ----------------------------- |
| Backend EC2 (Docker) | App + Docker setup | AMI + EBS snapshot            |
| Database (RDS MySQL) | Data               | Automated backups + snapshots |
| S3 (if any)          | Static files       | Versioning                    |

## RDS Backup
1. Go to your RDS database in your AWS console
2. Take a snapshot of the current state of the database.
3. Click on modify
4. Set backup retention to your desired no of days
5. Select apply immediately and save


## EC2 Backup
1. Create an image of both the bastion and the backend instances
2. It creates both AMI and EBS snapshot

## Testing

1. Terminate any one of the below 
	1. Backend EC2
	2. Frontend EC2
	3. RDS instance
2. Terminate the backend instance
3. Go to AMI's and select the backend ami and launch the instance with the same VPC, subnet and Security Group.
4. After boot check for the container's Status
```
docker ps
docker logs <container_id>
```
