# aws-lb-db-backend-frontend
![Untitled-2023-08-02-1735](https://github.com/reduanmasud/aws-lb-db-backend-frontend/assets/59122533/d335bfd7-296d-492d-ab56-58b03dc70900)

## First, goto AWS and Create A VPC with below configurations
**Resources to create:** `VPC and more`  
**Name tag auto-generation:** `devops`  
**IPv4 CIDR block:** `10.10.0.0/23`  
**IPv6 CIDR block:** `No IPv6 CIDR block`  
**Tenancy:** `Default`  
**Number of Availability Zones (AZs):** `1`  
**Number of public subnets:** `1`  
**Number of private subnets:** `1`  
**NAT gateways ($):** `In 1 AZ`  
**VPC endpoints:** `S3 Gateway`  
**DNS optionsInfo:**  
✅ Enable DNS hostnames  
✅ Enable DNS resolution  

## Create an EC2 for the load balancer; follow below configurations
### Name and tags
**Name:** : `loadbalancer`
### Application and OS Images (Amazon Machine Image)
**Operating System:** `Ubuntu`  
![image](https://github.com/reduanmasud/aws-lb-db-backend-frontend/assets/59122533/4034956f-43dc-47fe-8869-c9014eb1c621)  
### Instance type 
**Keep it Default** No need to change  
### Key pair (login)
Create, select, and follow screenshot  
![image](https://github.com/reduanmasud/aws-lb-db-backend-frontend/assets/59122533/0a5aa152-e2df-4a6d-bd2d-7d83039b615d)  

![image](https://github.com/reduanmasud/aws-lb-db-backend-frontend/assets/59122533/07d73e50-7862-4c67-8e98-c7771fa8a316)  

### Network Settings
**Edit Network Setting as follow**  
**VPC - required:** `devops-vpc` the VPC you have created   
**Subnet:** `public subnet` Select a public subnet from there  
**Auto-assign public IP:** `Enable`  
**Firewall (security groups):** `Create Security Group`  
**Security group name - required:** `my-app-security-group`  
**Description - required:**  Keep it as default or you can change  
No add few Security Group as follow  
**For HTTP:-**   
Type: `HTTP`  
Sourse type: `Anywhere`  
**For HTTPS:-**  
Type: `HTTP`  
Sourse type: `Anywhere`   
**For App purposes:**  
Type: `Custom TCP`  
Port Range: `3000`  
Sourse type: `Anywhere`   
> **Keep in mind we may need many rules**   


## Create an EC2 for the Backend
### Name and tags
**Name:** : `backend`
### Application and OS Images (Amazon Machine Image)
**Operating System:** `Ubuntu`  
![image](https://github.com/reduanmasud/aws-lb-db-backend-frontend/assets/59122533/4034956f-43dc-47fe-8869-c9014eb1c621)  
### Instance type 
**Keep it Default** No need to change  
### Key pair (login)
`No Need to create any key pair Just Select the previous one`
![image](https://github.com/reduanmasud/aws-lb-db-backend-frontend/assets/59122533/07d73e50-7862-4c67-8e98-c7771fa8a316)  
### Network Settings
**Edit Network Setting as follow**  
**VPC - required:** `devops-vpc` the VPC you have created   
**Subnet:** `private subnet` Select a private subnet from there  
**Auto-assign public IP:** `Disable`  
**Firewall (security groups):** `Select Existing Security Group`  
**Common security groups:** `my-app-security-group`  


## Create an EC2 for the FrontEnd
### Name and tags
**Name:** : `frontend`
### Application and OS Images (Amazon Machine Image)
**Operating System:** `Ubuntu`  
![image](https://github.com/reduanmasud/aws-lb-db-backend-frontend/assets/59122533/4034956f-43dc-47fe-8869-c9014eb1c621)  
### Instance type 
**Keep it Default** No need to change  
### Key pair (login)
`No Need to create any key pair Just Select the previous one`
![image](https://github.com/reduanmasud/aws-lb-db-backend-frontend/assets/59122533/07d73e50-7862-4c67-8e98-c7771fa8a316)  
### Network Settings
**Edit Network Setting as follow**  
**VPC - required:** `devops-vpc` the VPC you have created   
**Subnet:** `private subnet` Select a private subnet from there  
**Auto-assign public IP:** `Disable`  
**Firewall (security groups):** `Select Existing Security Group`  
**Common security groups:** `my-app-security-group`  


***
Now Let's setup servers
***  
Update all servers:
```sh 
sudo apt update
```   
Install Node.js on both the backend and the frontend. Follow : https://github.com/nodesource/distributions   
Install Nginx in all servers:  
```sh 
sudo apt install nginx
```
## Configure nginx.conf to each of the servers
In Every server, you can use this command to edit
```
sudo vim /etc/nginx/nginx.conf
```
After editing, check nginx & reload    
```
sudo nginx -t
sudo nginx -s reload
```

### Backend
edit nginx.conf file and  delete all and add these
```conf
events {}
http {

    upstream backend {
            server localhost:3000;
        }

    server {
        listen 80;
        location / {
            proxy_pass http://backend;
        }
    }
}
```

### Front End
edit nginx.conf file and  delete all and add these
```conf
events {}
http {

    upstream backend {
            server 10.10.1.28:80;
            #      ^^^^^^^^^^^^^ Your Backend Server IP
    }

    upstream frontend {
            server localhost:81;
    }

    server {
        listen 3000;
        location / {
            proxy_pass http://backend;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_read_timeout 86400;
        }
    }

    server {
        listen 80;

        location / {
            proxy_pass http://frontend;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_read_timeout 86400;
        }
    }
}

```


### Load Balancer
edit nginx.conf file, delete all and add these
```conf
events {}
http {

    upstream backend {
            server 10.10.1.28:80;
            #      ^^^^^^^^^^^^^ Your Backend Server IP
    }

    upstream frontend {
            server 10.10.1.15:80;
            #      ^^^^^^^^^^^^^ Your Frontend Server IP
    }

    server {
        listen 80;
        location / {
            proxy_pass http://frontend;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_read_timeout 86400;
        }
        location /api {
            proxy_pass http://backend;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_read_timeout 86400;
        }
    }

    server {
        listen 3000;
        location / {
            proxy_pass http://backend/;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_read_timeout 86400;
        }
    }
}
```
## Backend App setup
clone this repo and goto `backend` folder and run
```
npm i
```
Then run 
```
sudo npm install pm2 -g
```
Then Start the backend server
```
pm2 start "node app.js" --name backend
```


## Front-End App setup
Install yarn
```
sudo npm install --global yarn
```
Then Init Project
```
yarn
```
Now change `frontend/src/App.tsx`'s const host = "Your Backend IP"; `remove localhost`

Now run this command
```
yarn dev --host --port 81
```


