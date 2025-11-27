# AWS Cloud Development Environment Setup Guide
## For 2012 MacBook Pro with VS Code Remote-SSH

This guide will help you set up a complete cloud-based development environment on AWS that you can access from your 2012 MacBook Pro using VS Code.

---

## Table of Contents
1. [AWS Account Setup](#step-1-aws-account-setup)
2. [Create EC2 Instance](#step-2-create-ec2-instance)
3. [Configure SSH Keys](#step-3-configure-ssh-keys)
4. [Set Up Elastic IP](#step-4-set-up-elastic-ip)
5. [Install VS Code Remote-SSH Extension](#step-5-install-vs-code-remote-ssh-extension)
6. [Configure SSH Connection](#step-6-configure-ssh-connection)
7. [Connect and Set Up Development Tools](#step-7-connect-and-set-up-development-tools)
8. [Daily Usage and Cost Management](#step-8-daily-usage-and-cost-management)

---

## Step 1: AWS Account Setup

### 1.1 Create AWS Account
1. Go to https://aws.amazon.com
2. Click "Create an AWS Account"
3. Follow the prompts (you'll need a credit card, but we'll stay in free tier where possible)
4. Complete identity verification

### 1.2 Set Up Billing Alerts (Important!)
1. Go to AWS Console → Billing Dashboard
2. Click "Billing preferences" in left menu
3. Enable "Receive Free Tier Usage Alerts"
4. Enter your email
5. Set up a budget:
   - Go to "Budgets" in left menu
   - Click "Create budget"
   - Choose "Monthly cost budget"
   - Set amount to $10-15
   - Add your email for alerts

---

## Step 2: Create EC2 Instance

### 2.1 Launch Instance
1. Log into AWS Console (https://console.aws.amazon.com)
2. Search for "EC2" in the top search bar
3. Click "Launch Instance"

### 2.2 Configure Instance Settings

**Name and Tags:**
- Name: `dev-environment` (or whatever you prefer)

**Application and OS Images:**
- Click "Quick Start"
- Select **Ubuntu**
- Choose **Ubuntu Server 24.04 LTS** (free tier eligible)

**Instance Type:**
- Select **t3.micro** (1 vCPU, 1 GB RAM) or **t3.small** (2 vCPU, 2 GB RAM)
- t3.micro: ~$7.50/month if running 24/7
- t3.small: ~$15/month if running 24/7
- **Note:** You can start/stop to save money (only pay for hours used)

**Key Pair (Important!):**
1. Click "Create new key pair"
2. Name it: `dev-environment-key`
3. Key pair type: **RSA**
4. Private key format: **pem** (for Mac/Linux)
5. Click "Create key pair"
6. **SAVE THIS FILE!** It will download as `dev-environment-key.pem`
7. Move it to a safe location:
   ```bash
   # Open Terminal on your Mac
   mkdir -p ~/.ssh
   mv ~/Downloads/dev-environment-key.pem ~/.ssh/
   chmod 400 ~/.ssh/dev-environment-key.pem
   ```

**Network Settings:**
1. Click "Edit" next to Network settings
2. Leave "Create security group" selected
3. Security group name: `dev-environment-sg`
4. Description: `Security group for dev environment`
5. Keep the SSH rule (allows port 22)
6. Make sure "Allow SSH traffic from" is checked
7. You can select "My IP" for better security (recommended)

**Configure Storage:**
- Change size to **20 GB** (or more if you need it)
- Keep "gp3" as storage type
- This costs about $2/month for 20GB

### 2.3 Launch
1. Review your settings in the Summary panel
2. Click "Launch instance"
3. Wait for instance to show "Running" state (takes 1-2 minutes)

### 2.4 Note Your Instance Details
1. Click on your instance ID
2. Copy and save:
   - **Instance ID** (e.g., i-1234567890abcdef0)
   - **Public IPv4 address** (e.g., 54.123.45.67)
   - **Instance state** (should say "Running")

---

## Step 3: Configure SSH Keys

### 3.1 Set Correct Permissions (Mac Terminal)
```bash
# Make sure your key has the right permissions
chmod 400 ~/.ssh/dev-environment-key.pem

# Verify permissions
ls -la ~/.ssh/dev-environment-key.pem
# Should show: -r--------
```

### 3.2 Test SSH Connection
```bash
# Replace YOUR_PUBLIC_IP with your EC2 instance's public IP
ssh -i ~/.ssh/dev-environment-key.pem ubuntu@YOUR_PUBLIC_IP

# Example:
# ssh -i ~/.ssh/dev-environment-key.pem ubuntu@54.123.45.67
```

If successful, you'll see:
```
Welcome to Ubuntu 24.04 LTS
...
ubuntu@ip-xxx-xxx-xxx-xxx:~$
```

Type `exit` to disconnect for now.

**If you get an error:**
- "Permission denied" → Check your key permissions (step 3.1)
- "Connection refused" → Check security group allows SSH from your IP
- "Connection timeout" → Check your internet connection and security group

---

## Step 4: Set Up Elastic IP

An Elastic IP keeps your instance's public IP address the same (even after stopping/starting).

### 4.1 Allocate Elastic IP
1. In EC2 Console, go to "Elastic IPs" (left menu under "Network & Security")
2. Click "Allocate Elastic IP address"
3. Click "Allocate"
4. Note the allocated IP address

### 4.2 Associate with Instance
1. Select your new Elastic IP
2. Click "Actions" → "Associate Elastic IP address"
3. Select your instance from dropdown
4. Click "Associate"

### 4.3 Update Your Notes
- Write down your **Elastic IP** (this won't change now)
- You'll use this IP for all future connections

**Important:** Elastic IPs are FREE while your instance is running, but cost $0.005/hour (~$3.60/month) if the instance is stopped but the IP is still allocated. To avoid charges when not using your instance for extended periods, you can release the Elastic IP (you'll get a new one when you need it again).

---

## Step 5: Install VS Code Remote-SSH Extension

### 5.1 Open VS Code
1. Launch VS Code on your MacBook

### 5.2 Install Extension
1. Click the Extensions icon (or press Cmd+Shift+X)
2. Search for "Remote - SSH"
3. Look for the one by Microsoft (should be first result)
4. Click "Install"

### 5.3 Verify Installation
- You should see a small green icon in the bottom-left corner of VS Code
- Click it to see remote connection options

---

## Step 6: Configure SSH Connection

### 6.1 Create SSH Config File
1. Open Terminal on your Mac
2. Create or edit SSH config:
   ```bash
   nano ~/.ssh/config
   ```

3. Add this configuration (replace YOUR_ELASTIC_IP with your actual IP):
   ```
   Host aws-dev
       HostName YOUR_ELASTIC_IP
       User ubuntu
       IdentityFile ~/.ssh/dev-environment-key.pem
       ServerAliveInterval 60
       ServerAliveCountMax 3
   ```

4. Save and exit:
   - Press Ctrl+O (to save)
   - Press Enter (to confirm)
   - Press Ctrl+X (to exit)

**Config Explanation:**
- `Host aws-dev` - The name you'll use to connect (you can change this)
- `HostName` - Your Elastic IP address
- `User ubuntu` - Default user for Ubuntu EC2 instances
- `IdentityFile` - Path to your SSH key
- `ServerAliveInterval` - Keeps connection alive
- `ServerAliveCountMax` - Prevents timeout

### 6.2 Test Configuration
```bash
# You should now be able to connect using just the host name
ssh aws-dev

# If successful, you'll be connected to your EC2 instance
# Type 'exit' to disconnect
```

---

## Step 7: Connect and Set Up Development Tools

### 7.1 Connect via VS Code
1. Open VS Code
2. Press **Cmd+Shift+P** (Command Palette)
3. Type "Remote-SSH: Connect to Host"
4. Select "aws-dev" from the list
5. A new VS Code window will open
6. Wait for VS Code to install the VS Code Server on your EC2 instance (this happens automatically on first connect - takes 1-2 minutes)

### 7.2 Verify Connection
- Bottom-left corner should show: **SSH: aws-dev**
- This means you're connected!

### 7.3 Open Terminal in VS Code
1. Go to Terminal → New Terminal (or press Ctrl+` )
2. You're now in a terminal on your EC2 instance

### 7.4 Update System
```bash
# Update package lists
sudo apt update

# Upgrade installed packages
sudo apt upgrade -y
```

### 7.5 Install Development Tools

**Basic essentials:**
```bash
# Build tools
sudo apt install -y build-essential curl wget git

# Install Git and configure
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

**Choose your language/tools:**

**For Python Development:**
```bash
# Install Python 3 and pip
sudo apt install -y python3 python3-pip python3-venv

# Verify installation
python3 --version
pip3 --version

# Create a virtual environment for your projects
mkdir -p ~/projects
cd ~/projects
python3 -m venv venv
source venv/bin/activate
```

**For Node.js/JavaScript Development:**
```bash
# Install Node.js (using NodeSource repository for latest LTS)
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt install -y nodejs

# Verify installation
node --version
npm --version

# Install common global packages
sudo npm install -g yarn typescript ts-node
```

**For Go Development:**
```bash
# Install Go
sudo snap install go --classic

# Verify installation
go version

# Set up Go workspace
mkdir -p ~/go/{bin,src,pkg}
echo 'export GOPATH=$HOME/go' >> ~/.bashrc
echo 'export PATH=$PATH:$GOPATH/bin' >> ~/.bashrc
source ~/.bashrc
```

**For Rust Development:**
```bash
# Install Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source ~/.bashrc

# Verify installation
rustc --version
cargo --version
```

**For Java Development:**
```bash
# Install OpenJDK
sudo apt install -y openjdk-17-jdk maven

# Verify installation
java --version
mvn --version
```

**Docker (if needed):**
```bash
# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add your user to docker group
sudo usermod -aG docker ubuntu

# Log out and back in for group changes to take effect
exit
# Then reconnect via VS Code
```

### 7.6 Install VS Code Extensions (on remote)

When connected to your EC2 instance:
1. Click Extensions in VS Code
2. Install extensions you need (they'll install on the remote):
   - Python (Microsoft)
   - ESLint
   - Prettier
   - GitLens
   - Docker (if you installed Docker)
   - Any language-specific extensions you use

### 7.7 Create Your Project Directory
```bash
# Create a projects folder
mkdir -p ~/projects
cd ~/projects

# Clone a repository or start a new project
git clone https://github.com/yourusername/your-repo.git
# OR
mkdir my-new-project
cd my-new-project
```

### 7.8 Open Your Project in VS Code
1. File → Open Folder
2. Navigate to your project directory
3. Click OK

You're now ready to code!

---

## Step 8: Daily Usage and Cost Management

### 8.1 Connecting to Your Environment

**Every time you want to code:**
1. Make sure your EC2 instance is running (check AWS Console)
2. Open VS Code
3. Press Cmd+Shift+P
4. Type "Remote-SSH: Connect to Host"
5. Select "aws-dev"
6. Start coding!

**Reconnecting:**
- If you close VS Code, just repeat the above steps
- VS Code remembers your remote environment

### 8.2 Managing Your EC2 Instance

**To Stop Instance (save money when not using):**
1. Go to EC2 Console
2. Select your instance
3. Instance State → Stop Instance
4. **Note:** You only pay for hours the instance is running ($0.01/hour for t3.micro)

**To Start Instance:**
1. Go to EC2 Console
2. Select your instance
3. Instance State → Start Instance
4. Wait 1-2 minutes for it to start
5. Connect via VS Code

**Cost Breakdown:**
- **t3.micro running 24/7:** ~$7.50/month
- **t3.micro running 8 hours/day:** ~$2.50/month
- **Storage (20GB):** ~$2/month
- **Elastic IP (while running):** Free
- **Elastic IP (while stopped):** ~$3.60/month (or release it to avoid charges)

**Best Practice:**
- Stop your instance when done for the day
- Start it when you need it (takes 1-2 minutes)
- This can reduce costs to $4-5/month total

### 8.3 Backup Strategy

**Option 1: Git (Recommended)**
```bash
# Commit and push your work regularly
git add .
git commit -m "Your commit message"
git push origin main
```

**Option 2: EBS Snapshots**
1. EC2 Console → Volumes (under Elastic Block Store)
2. Select your instance's volume
3. Actions → Create Snapshot
4. Snapshots cost ~$0.05/GB/month

**Option 3: Download Files**
```bash
# From your Mac Terminal, download files from EC2
scp -r aws-dev:~/projects/my-project ~/Desktop/backup/
```

### 8.4 Upgrading Your Instance

**If you need more power:**
1. Stop your instance
2. Select instance → Actions → Instance Settings → Change Instance Type
3. Choose a larger size (t3.small, t3.medium, etc.)
4. Start instance

### 8.5 Monitoring Costs

**Check your AWS bill:**
1. AWS Console → Billing Dashboard
2. Review "Month-to-Date Costs"
3. Check for any unexpected charges

**Set up Cost Anomaly Detection:**
1. Billing Dashboard → Cost Anomaly Detection
2. Create a monitor to alert you of unusual spending

---

## Troubleshooting

### Can't Connect via SSH
1. Check instance is running (AWS Console)
2. Verify security group allows SSH from your IP
3. Confirm you're using the correct Elastic IP
4. Test connection from Terminal: `ssh aws-dev`

### VS Code Remote Connection Fails
1. Make sure Remote-SSH extension is installed
2. Check ~/.ssh/config is correct
3. Try connecting via Terminal first to rule out SSH issues
4. Check VS Code output: View → Output → Select "Remote-SSH"

### Instance Running Slow
1. Check instance type (t3.micro has limited CPU)
2. Check disk space: `df -h`
3. Check memory: `free -h`
4. Consider upgrading to t3.small or larger

### Lost SSH Key
- Unfortunately, without the key, you can't access your instance
- You'd need to create a new instance
- **Prevention:** Back up your .pem file to a secure location

### Forgot to Stop Instance
- Stop it as soon as you remember
- Costs are hourly, so you'll only be charged for the time it ran
- Set up billing alerts to prevent this

---

## Advanced Tips

### 8.6 Create an Alias for Easy Starting/Stopping

Add these to your Mac's `~/.zshrc` or `~/.bash_profile`:

```bash
# Replace i-xxxxxxxxx with your instance ID
alias ec2-start='aws ec2 start-instances --instance-ids i-xxxxxxxxx'
alias ec2-stop='aws ec2 stop-instances --instance-ids i-xxxxxxxxx'
alias ec2-status='aws ec2 describe-instances --instance-ids i-xxxxxxxxx --query "Reservations[0].Instances[0].State.Name"'
```

Then you can just type:
```bash
ec2-start    # Start your instance
ec2-stop     # Stop your instance
ec2-status   # Check if it's running
```

(Requires AWS CLI installed: `brew install awscli` and configured with your credentials)

### 8.7 Auto-shutdown Script

To prevent forgetting to stop your instance, create a cron job:

```bash
# On your EC2 instance
crontab -e

# Add this line to shut down at midnight every night
0 0 * * * sudo shutdown -h now
```

### 8.8 Port Forwarding for Web Development

If you're developing web apps:

1. **In VS Code Terminal on remote:**
   ```bash
   # Start your dev server (example: React)
   npm start
   ```

2. VS Code will automatically detect the port and ask if you want to forward it
3. Click "Forward Port" or "Open in Browser"
4. Your web app will open in your local browser!

**Manual port forwarding:**
```bash
# From your Mac Terminal
ssh -L 3000:localhost:3000 aws-dev

# Now localhost:3000 on your Mac = localhost:3000 on EC2
```

---

## Conclusion

You now have a fully functional cloud development environment! Your 2012 MacBook Pro is just handling the VS Code interface, while all the heavy work happens in the cloud.

**Next Steps:**
1. Install your preferred extensions in VS Code (on the remote)
2. Clone your repositories
3. Start coding!

**Remember:**
- Stop your instance when you're done to save money
- Push your code to Git regularly
- Monitor your AWS costs

**Resources:**
- AWS EC2 Documentation: https://docs.aws.amazon.com/ec2/
- VS Code Remote-SSH: https://code.visualstudio.com/docs/remote/ssh
- AWS Free Tier: https://aws.amazon.com/free/

Happy coding! 🚀
