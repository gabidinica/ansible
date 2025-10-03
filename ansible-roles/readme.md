# Demo Project:
Structure Playbooks with Ansible Roles

## Technologies used:
Ansible, Docker, AWS, Linux

## Project Description:
Break up large Ansible Playbooks into smaller manageable files using Ansible Roles

---

## Understanding Ansible Roles

- A **Role** is like a package for tasks.  
- Roles are like functions: they extract common logic and can be reused in different places with different parameters.  

### Structure
- The first folder is `roles/`.  
- Inside `roles/`, we create a folder for each role. Example: `create_user/`.  
- Inside each role directory, we have a folder called `tasks/`.  
- Inside each `tasks/` folder, we must have a `main.yaml` file.  

## Using Roles in a Playbook

To use the roles inside `deploy-docker-with-roles.yaml`, we define the `roles` section and then specify the role name.
---

## Testing the Playbook with Terraform

1. Create the EC2 servers with Terraform:
```bash
terraform apply
```

2. To run the playbook with dynamic inventory:

```bash
ansible-playbook deploy-docker-with-roles.yaml
```

- In the hosts file, we have set the inventory to use inventory_aws_ec2.yaml.

## Managing Variables in Ansible Roles

- In the `vars/` folder are the variables used in starting containers.  
- In `main.yaml`, replace `docker_username` with your **Docker Hub username**.  
- The variables defined in the `vars/` folder will **overwrite** the ones defined in `project-vars`.  

### Default Values
- Default values for variables can be defined in another folder called `defaults/`.  


