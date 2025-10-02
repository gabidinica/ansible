# Demo Project: Configure Dynamic Inventory

## Technologies Used
- Ansible
- Terraform
- AWS

## Project Description
- Created an EC2 instance using Terraform.
- Developed an Ansible AWS EC2 plugin to dynamically set the inventory of EC2 servers.
- Enabled Ansible to manage EC2 instances without hard-coding server addresses in the inventory file.

---

- Create 3 EC2 instances with Terraform
- Connect to these servers with Ansible without hard-coding the IP addresses

- In the Terraform folder, in `main.tf` we are adding 3 EC2 instances.
- Run `terraform init`.
- Run `terraform apply`.
- The instances can be checked in the AWS account under EC2 and refreshed to view the new instances.

## AWS EC2 Plugin
- Documentation: [AWS EC2 Plugin](https://docs.ansible.com/ansible/latest/collections/amazon/aws/aws_ec2_inventory.html)
- In `ansible.cfg`, the `aws_ec2` plugin is added so it can be used in the project.
- The plugin configuration file is `inventory_aws_ec2.yaml`. The file name should always contain the plugin name.

- To test in the terminal:  
```bash
ansible-inventory -i inventory_aws_ec2.yaml --list
```

> This command retrieves all EC2 instances and their data.

- To get only the names of the servers:
```bash
ansible-inventory -i inventory_aws_ec2.yaml --graph
```

## Handling DNS Names for EC2 Instances

- The list displayed contains the private DNS name. If the instances do not have DNS assigned, we get the private DNS.
- To fix this:
  1. Destroy the existing infrastructure:
     ```bash
     terraform destroy
     ```
  
  2. Inside the VPC resource, enable public DNS names by adding:
     ```hcl
     enable_dns_hostnames = true
     ```
  
 3. ```bash
    terraform apply —auto-approve
   ```

Run the command:  
  ```bash
  ansible-inventory -i inventory_aws_ec2.yaml --graph
  ```

> Now, the output displays the public DNS names of the EC2 instances.


- In `deploy-docker-new-user.yaml`, `hosts: aws_ec2` is used so that the **public DNS names** are utilized.
- After these updates, run the playbook:  
```bash
ansible-playbook -i inventory_aws_ec2.yaml deploy-docker-new-user.yaml
```

- If we decide to always use the inventory file `inventory_aws_ec2.yaml`, we can update `ansible.cfg` so that we **do not need to pass it as a parameter** every time.
- To clean up the infrastructure, run:  
```bash
terraform destroy
```

- To have 2 dev servers and 2 prod servers, update the `main.tf` file accordingly.
- Apply the changes with Terraform:  
```bash
terraform apply --auto-approve
```

## Filtering EC2 Instances in Ansible Inventory

- Check the new instances in EC2 on AWS.
- We can filter the servers in `inventory_aws_ec2.yaml` if we want only specific instances, e.g., by image ID.
- In our case, we filter by name, for example `dev*`, and run:  
```bash
ansible-inventory -i inventory_aws_ec2.yaml --list
```

- In `inventory_aws_ec2.yaml`, we can define keyed groups:  
```yaml
keyed_groups:
  - key: tags
    prefix: "tag"
```

> This filters servers by their tags.

- To filter only the dev servers, use the tag @tag_Name_dev_server as a value for hosts in the playbook.
- When executed, only the dev servers will be configured.
