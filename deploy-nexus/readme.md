# Demo Project:
Automate Nexus Deployment

## Technologies used:
Ansible, Nexus, DigitalOcean, Java, Linux

## Project Description:
Create Server on DigitalOcean  
Write Ansible Playbook that creates Linux user for Nexus,configure server, installs and deploys Nexus and verifies  
that it is running successfully.

---

## Steps to Create Droplet on DigitalOcean

1. Go to **DigitalOcean**.  
2. Click **Create Droplet**.  
   - Region: **Frankfurt**  
   - Plan: **Regular CPU**  
   - Pricing: **$18/mo, 2 GB**  
   - Authentication: **SSH Keys**  
3. Click **Create Droplet**.  
4. Copy the **IP Address** of the droplet.  
5. In the **hosts file**, replace `droplet_ip` with the copied IP address.  

> ⚠️ To prevent insufficient memory errors, ensure you choose a server with sufficient memory.

### Nexus Installation Automation Notes

- In **nexus.sh** are all the commands that were manually executed to install Nexus on a droplet.  
  It’s used as a reference to do the exact same thing using a playbook.  

- To download Nexus, we use the module **`get_url`**, which is used to download HTTP, HTTPS, or FTP files to the remote server.  
  - For Windows, there’s a separate module: **`win_get_url`**.  

- We added `inventory=hosts` into the **ansible.cfg** file, in order to avoid passing `-i hosts` every time when running `ansible-playbook`.  

### Handling Nexus Installation with Ansible

- We’re saving the result of executing the **get_url** module.  
- To avoid SSH into the server to get the Nexus version name that was installed, we retrieve it automatically from **download_result**.  
- In the result, there’s a **dest** attribute that contains the name of the Nexus version.  

### Using the Find Module
- **find** module: returns a list of files based on specific criteria.  
- We’re using it to change the Nexus module name.  

### Nexus Deployment Details

- In the **path** attribute, there’s the full path and the folder name.  
- Renaming the folder should happen only once, and for this we’re using the **stat** module, which retrieves file or file system status.  
- `stat_result.stat.exists` → gives information if the folder exists or not.  

### Running the Playbook
```bash
ansible-playbook deploy-nexus.yaml
```

### Connecting to the Server

1. Connect to the server:  
```bash
ssh root@droplet_ip
```

2. List contents in /opt/:
`ls /opt/`
> → You’ll see the Nexus tar file.
3. Verify ownership: `ls -l /opt/`
> Nexus and Sonatype should be owned by the nexus user.

## Managing Nexus User and Permissions

- To create the **nexus group**, we use the **`group`** module.  
- We use the **`file`** module to assign the **nexus user** recursively to the Nexus folder and everything inside it.  
### Verification Steps
After running:  
```bash
ansible-playbook -i hosts deploy-nexus.yaml
```

Check the following:
```bash
ls -l /opt/
su - nexus
ls -l /opt/nexus
```

### Verifying Nexus User in nexus.rc
1. Open the **nexus.rc** file:  
```bash
vim /opt/nexus/bin/nexus.rc
```

2. Check the commented line where the nexus user needs to be added.
3. You can verify in your terminal that the line was successfully added.

### Option 1: Using `blockinfile`

```yaml
blockinfile:
  path: /opt/nexus/bin/nexus.rc
  block: |
    run_as_user="nexus"
```

### Option 2: Using `lineinfile`
Exactly how it's used in the script.
- Ensures a particular line is in a file or replaces an existing line using regex.
- Useful when you want to change a single line in a file only.

## Verifying Nexus Process

1. Connect to the server:  
```bash
ssh root@droplet_ip
```

2. Check if Nexus is running:
```bash
ps aux | grep nexus
```

> You should see the Nexus process running.
