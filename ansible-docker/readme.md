# Demo Project: Ansible & Docker

## Technologies Used
- Ansible  
- AWS  
- Docker  
- Terraform  
- Linux  

## Project Description
- Create an **AWS EC2 Instance** with Terraform  
- Write an **Ansible playbook** to:  
  - Install necessary technologies like Docker and Docker Compose  
  - Copy the `docker-compose` file to the server  
  - Start the Docker containers configured inside the `docker-compose` file  

---

# Notes on Docker Deployment

- **Obs:** We have two `deploy-docker` files:  
  1. One specifically for **EC2 instances**, since we are creating a new EC2 instance.  
  2. A more **generic version** that also handles creating a new user.  

- To create the EC2 instance using **Terraform**, we use the project from:  
[Terraform AWS Infrastructure GitHub](https://github.com/gabidinica/terraform/tree/main/automate-aws-infrastructure)  
## Steps to Create EC2 Instance with Terraform

1. **Clone the project locally**.  
2. Open `main.tf` and remove the following lines:  
```hcl
user_data = file("entry-script.sh")
user_data_replace_on_change = true
```

3. Initialize Terraform:
`terraform init`

4. Apply Terraform configuration and confirm when prompted:
`terraform apply`

5. Verify the EC2 instance:
- Check in your AWS Console that the EC2 instance is running.
- Copy the public IP address from the terminal where the instance was created.

### Configuring Ansible Hosts and Installing Docker

1. In the **hosts** file, under the `docker_server` group, paste the **public IP address** of the EC2 instance.  
2. Connect to the EC2 instance via terminal:  
```bash
ssh ec2-user@public-ip-address
```

3. Yum module: Use this module to install Docker.
> Note: To install packages, you need to switch to the root user instead of the default ec2-user.

 Execute the playbook (1st play):  
```bash
ansible-playbook deploy-docker.yaml
```

### 2nd play: Install Docker Compose

uname -m runs as a shell command and the output is registered under a variable, which will be used for the download URL.
After the playbook runs, connect to the server and verify Docker Compose:
```bash
docker-compose
```

### 3rd Play: Start Docker Daemon
- Use the **systemd module** to control systemd services on remote hosts.  
- This play ensures the **Docker daemon** is started and running.  

### 4th Play: Add ec2-user to Docker Group

- Add the **ec2-user** to the **docker group** to allow running Docker commands without `sudo`.  
- Use the **`meta`** module to reset the connection immediately after adding the user to the Docker group.  
- To pull Docker images, use the **`docker_image`** module.  

### 5th Play: Start Docker Containers

- Use the **`docker-compose-full.yaml`** file to start the containers.  
- In `src`, add the **full path** from your local machine where the `docker-compose.yaml` file is located.  
- Before pulling images, perform **Docker login** using the **`docker_login`** module:  
  - Replace `username` with your Docker Hub username.  
  - Add your Docker Hub **password** into the `project-vars` file.  

- In the terminal, list files to check for the `docker-compose.yaml` file:  
```bash
ls
```

- After executing the playbook/script, verify running containers:
`docker ps`

