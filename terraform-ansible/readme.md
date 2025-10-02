# Demo Project: Ansible Integration in Terraform

## Technologies Used
- Ansible  
- Terraform  
- AWS  
- Docker  
- Linux  

## Project Description
- Create an **Ansible playbook** for Terraform integration.  
- Adjust **Terraform configuration** to execute the Ansible playbook automatically.  
  - Once Terraform provisions a server, it will automatically execute the Ansible playbook to configure the server.  

---

### Integrating Ansible with Terraform

- In the **Terraform** folder, in `main.tf`, under `aws_instance` after `tags`, the **provisioner** with the Ansible command is added.  

- **Local-exec provisioner**: Invokes a local executable after a resource is created.  

- Navigate to the **Ansible** folder.  
  - Update your **local path** as needed for the playbook execution.  

### Dynamic Inventory with Terraform and Ansible

- Use the `--inventory` option to provide the **new IP address** of the newly created server dynamically.  
  - This replaces the need to manually update the `hosts` file.  

- In the command, specify:  
  - **User**  
  - **Private key**  

### deploy-server-new-user.yaml Configuration

- In `deploy-server-new-user.yaml`, `hosts: all` is used because the **inventory** (an IP address) is passed dynamically from `main.tf` in the Terraform folder.  

1. **Initialize and apply Terraform**:  
```bash
terraform init
terraform apply
```

2. Connect to EC2 instance:
```bash
ssh ec2-user@ec2-public-ip
```

3. Verify Docker containers:
```bash
sudo docker ps
```

> The containers from the docker-compose file should be running.

- Ensure that the **port is open** and actually allows **OpenSSH** connections.  

- Execute Terraform:  
```bash
terraform apply
```

> Verify in the AWS EC2 Console that the instance is running.

- null_resource: Use this in Terraform if you want to define a task separately from resource creation.
