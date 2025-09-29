# Demo Project: Automate Node.js Application Deployment

## Technologies Used
- Ansible
- Node.js
- DigitalOcean
- Linux

## Project Description
Write Ansible Playbook that installs necessary technologies, creates Linux user for an application and deploys a NodeJS application with that user.  

**Key Steps:**
1. Create a server on DigitalOcean.  
2. Write an Ansible playbook that:
   - Installs the necessary technologies.
   - Creates a Linux user for the application.
   - Deploys a Node.js application under that user.
   - Start the application.
   - Verify the application is running successfully.

---

### Step 1: Create Droplets on DigitalOcean

1. Go to **DigitalOcean → Create → Droplet**.
2. Select the **Frankfurt** data center.
3. Choose **Ubuntu** as the operating system.
4. Under **CPU**, select **Regular**, then choose the **2GB** RAM option.
5. Configure your **SSH key** for secure access.
6. Increase the number of droplets to **2**.
7. Click **Create Droplet**.
8. Copy the public IP address of the droplet after it's created.

### Step 2: Configure the Ansible Hosts File

1. Open the Ansible `hosts` inventory file.
2. Paste the droplet IP addresses into the file.
3. Configure each entry with your own SSH keys for passwordless access.

### Step 3: Configure the Ansible Playbook

1. Open the playbook `deploy-node.yaml`.
2. Replace `droplet-ip` with the IP address of your droplet(s).
3. For reference on all the parameters used in the playbook (e.g., for the `apt` module), check the official Ansible documentation:  
   [Ansible Apt Module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_module.html#ansible-collections-ansible-builtin-apt-module)

### Step 4: Install Required Packages

We will install a list of packages on the server, such as `nodejs` and `npm`.  

**Example syntax in Ansible:**

```yaml
- name: Install a list of packages
  ansible.builtin.apt:
    pkg:
      - nodejs
      - npm
```

### Step 5: Deploy Node.js Application

We will copy and unpack the Node.js application tar file as part of the deployment.

1. Navigate to the `nodejs-app` folder in your terminal:
```bash
cd nodejs-app
```

2. Create a tar file of your Node.js app using npm pack:
```bash
npm pack
```

> This will generate a .tgz tar file, which we will copy to the target server and unpack as part of the Ansible playbook Deploy Node.js App.

### Step 6: Configure Playbook with Local Tar File

1. Open the Ansible playbook responsible for deploying the Node.js app (e.g., `deploy-node.yaml`).
2. Locate the task that copies the tar file from your local machine to the server.
3. Replace the placeholder `<your-local-path-to-tar-file>` with the **full path to the tar file** you generated with `npm pack`.

Example:

```yaml
- name: Copy Node.js app tar file
  ansible.builtin.copy:
    src: /Users/yourusername/nodejs-app/nodejs-app-1.0.0.tgz
    dest: /home/appuser/
```

### Step 7: Execute the Ansible Playbook
Run the following command in your terminal to deploy the Node.js application:
```bash
ansible-playbook -i hosts deploy-node.yaml
```

### Step 8: Verify Deployment on the Droplet

1. SSH into your droplet:
```bash
ssh root@<droplet-ip>
```

2. Check that the tar file has been copied: `ls` > You should see .tgz file here
3. Verify the contents of the Node.js application have been unpacked: `ls package`
> You should see your application files here.

### Optional: Run Node.js Server Asynchronously with Ansible

If you want to start the Node.js server without waiting for it to finish, you can use Ansible's asynchronous mode:
```yaml
- name: Start Node.js server asynchronously
  ansible.builtin.shell: node server.js
  async: 1000   # Maximum runtime in seconds
  poll: 0       # Do not wait for completion
```

**Note**:
- Debug module print statements during execution. app_status is a dictionary and we can print out standard output lines.

### Executing Tasks with a Different User

To run tasks as a different user (e.g., a non-root application user), follow these steps:
1. **Create a new Linux user** on the droplet for running the app.  
2. **Run tasks using the new user** in your playbook:
```yaml
- name: Start Node.js app as a specific user
  ansible.builtin.shell: node server.js
  become: true           # allows switching to another user
  become_user: appuser   # replace with the desired user
```


