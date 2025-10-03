# Demo Project: Ansible Integration in Jenkins

## 🚀 Technologies Used
- Ansible  
- Jenkins  
- DigitalOcean  
- AWS  
- Boto3  
- Docker  
- Java  
- Maven  
- Linux  
- Git  

## 📖 Project Description
This project demonstrates **automated infrastructure and configuration management** by integrating Ansible with Jenkins CI/CD pipelines.

### Steps

1. **Server Setup**
   - Create and configure a dedicated server for **Jenkins**.
   - Create and configure a dedicated server for **Ansible Control Node**.

2. **Ansible Playbook**
   - Write an Ansible playbook to configure **two EC2 instances** as managed nodes.

3. **Jenkins Configuration**
   - Add SSH key credentials in Jenkins for:
     - **Ansible Control Node server**
     - **Ansible Managed Node servers**
   - Configure Jenkins to execute the Ansible playbook on the remote Control Node server as part of a **CI/CD pipeline**.

4. **Jenkinsfile Workflow**
   The Jenkinsfile is configured to perform the following tasks:
   a. Connect to the remote **Ansible Control Node server**.  
   b. Copy the **Ansible playbook and configuration files** to the Control Node.  
   c. Copy the **SSH keys** for the Managed Node servers to the Control Node.  
   d. Install **Ansible, Python3, and Boto3** on the Control Node server.  
   e. Execute the playbook **remotely on the Control Node**, which configures the two EC2 Managed Nodes.

---

## Setting up Ansible Control Node on DigitalOcean

### Step 1: Create a Droplet
1. Go to **DigitalOcean** and create a new droplet.
2. Configuration:
   - **Region:** Frankfurt
   - **OS:** Ubuntu
   - **Plan:** Regular CPU, $18/mo
3. Rename the droplet: `ansible-server`

---

### Step 2: SSH into the Server
```bash
ssh root@<droplet_ip>
```

### Step 3: Install Ansible
```bash
apt update
apt install ansible-core -y
```

### Step 4: Install AWS Python Modules
```bash
apt install python3-boto3 -y
```

> This installs both Boto3 and Botocore, required for dynamic inventory with AWS.

### Step 5: Configure AWS Credentials

To use the **dynamic inventory** in Ansible, AWS credentials must be set up on the `ansible-server`.

1. Create the AWS configuration directory and navigate to it:
```bash
mkdir ~/.aws
cd ~/.aws
```

2. Create the credentials file:
`vim credentials`

3. Copy your AWS credentials from your local machine: `cat ~/.aws/credentials`
> Paste the content into the credentials file on the Ansible server.

4. Verify the credentials file exists and is readable:
```bash
cd ~
ls ~/.aws/credentials
```

5. Exit server: `exit`

## Launch EC2 Instances for Ansible Managed Nodes

### Step 1: Launch Instances
1. Go to **AWS Management Console → EC2 → Launch Instances**.  
2. Configuration:
   - **Number of instances:** 2  
   - Leave all other settings as **default**.

### Step 2: Key Pair
1. Create a **new key pair**:
   - Name: `ansible-jenkins`
   - Download the `.pem` file.  
2. This key will be used by **Ansible** to connect to these EC2 instances as managed nodes.

---

### Step 3: Application Repository
For this project, we are using the sample Java Maven application:  
[Java Maven App Repository](https://github.com/gabidinica/java-maven-app)

Update Jenkinsfile
- Make sure the **Jenkinsfile** is updated accordingly to execute the Ansible playbook remotely.
- On the **ANSIBLE_SERVER**, replace placeholders with your **public IP** where required.

---

### Organize Ansible Files
1. Inside your project folder, create a new folder for Ansible:
```bash
mkdir Ansible
```

2. Copy all Ansible files into this folder.
3. Configure ansible.cfg: In ansible.cfg, specify the path to the .pem key file that was downloaded when creating the EC2 instances. This allows Ansible to connect to the managed nodes via SSH.
4. Playbook for Docker: my-playbook.yaml contains plays for:installing Docker and installing Docker Compose.
Use `ssh-agent` to Connect to Remote Server
- `ssh-agent` allows you to use an SSH key to connect to a remote server and copy all necessary files.

- Access Jenkins Dashboard and Manage Plugins
Open your browser and go to:
`http://<droplet-ip>:8080`
 - Navigate to: Dashboard → Manage Jenkins → Plugins -> Check for SSH Agent that's installed.

## Add SSH Credentials in Jenkins

### Step 1: Open Jenkins Credentials
1. Navigate to: **Manage Jenkins → Credentials → Global → Add Credentials**  

### Step 2: Configure SSH Key
- **Kind:** SSH Username with private key  
- **ID / Name:** `ansible-server-key`  
- **Username:** `root`  
- **Private Key:** Enter directly  

### Step 3: Convert Local SSH Key (if needed)
On your local computer, open a terminal and display your SSH private key:

```bash
cat ~/.ssh/id_ed25519
```

If necessary, convert it to PEM format for Jenkins:
```bash
ssh-keygen -p -f ~/.ssh/id_ed25519 -m pem -P "" -N ""
```

> Copy the converted key and paste it into the Jenkins private key field. Click Create to save credentials.

On your machine in terminal:

```bash
cat ~/Downloads/ansible-jenkins.pem
```

Copy the result.

In Jenkins, add a new credential:
- Kind: SSH Username with private key
- ID / Name: ec2-server-key
- Username: ec2-user
- Private Key: Enter directly → paste the copied key
- Click Create to save the credential.

## Create Jenkins Pipeline and Verify Ansible Files

### Step 1: Create Jenkins Pipeline
1. Go to **Jenkins Dashboard → All → Create Pipeline**  
2. Configure the pipeline:
   - **Name:** `ansible-pipeline`  
   - **Definition:** Pipeline  
   - **Pipeline from SCM:** Git  
   - **Repository URL:** Paste your project repository  
   - **Credentials:** GitHub account credentials  
   - **Branch:** Specify the branch name  
3. Click **Save**.  
4. Click **Build Now** to trigger the pipeline.

---

### Step 2: Verify Files on Ansible Server
On your local machine, SSH into the Ansible server:

```bash
ssh root@<droplet-ip>
ls
```

> You should see all the Ansible files along with the SSH key .pem file.

## Execute Ansible Playbook from Jenkins Pipeline

### Step 1: Install SSH Pipeline Plugin
1. Go to **Jenkins Dashboard → Manage Jenkins → Plugins**  
2. Search for **SSH Pipeline Steps** and install the plugin.  
   - This plugin allows Jenkins to execute command-line commands on a remote server.  

---

### Step 2: Execute Ansible Playbook
- In your Jenkins pipeline, add a **stage** to call the Ansible playbook that configures the EC2 instances.  
- Use the **SSH Pipeline Steps plugin** to execute commands on the remote Ansible Control Node.

---

### Step 3: Optional Automation Script
- You can use a script that **automates installation of Ansible and Boto3** on the remote server, so you don’t need to do it manually.  
- This step is optional but recommended for smoother pipeline execution.

