# Install SSM agent:

# Download and install
sudo snap install amazon-ssm-agent --classic

# Start and enable
sudo systemctl start snap.amazon-ssm-agent.amazon-ssm-agent.service
sudo systemctl enable snap.amazon-ssm-agent.amazon-ssm-agent.service

# Verify it's running
sudo systemctl status snap.amazon-ssm-agent.amazon-ssm-agent.service

Then attach the IAM instance profile to both EC2s.
This is required — SSM agent needs IAM permissions to talk to AWS:

Go to EC2 Console → select your instance → Actions → Security → Modify IAM role
If you don't have a role yet, create one:

IAM → Roles → Create role → EC2 use case
Attach policy: AmazonSSMManagedInstanceCore
Name it ec2-ssm-role
Assign ec2-ssm-role to instances

# For snap install:
sudo systemctl restart snap.amazon-ssm-agent.amazon-ssm-agent.service

# Check logs to confirm it connected:
sudo tail -f /var/log/amazon/ssm/amazon-ssm-agent.log


# Switch to SSH (Recommended) GIT Https to SSH
Step 1 — Generate SSH key on golden EC2:
bash# Start your golden EC2 and SSH in
ssh-keygen -t ed25519 -C "golden-ami-deploy" -f ~/.ssh/gitlab_deploy -N ""

# Copy the PUBLIC key
cat ~/.ssh/gitlab_deploy.pub
Step 2 — Add public key to GitLab:
GitLab → your repo → Settings → Repository → Deploy Keys → Add new deploy key
Title:  golden-ami-deploy
Key:    paste the pub key here
Access: Read only ✅ (no need for write)
Step 3 — Configure SSH on golden EC2:
bash# Create SSH config
cat > ~/.ssh/config << 'EOF'
Host gitlab.com
    HostName gitlab.com
    User git
    IdentityFile ~/.ssh/gitlab_deploy
    StrictHostKeyChecking no
EOF

chmod 600 ~/.ssh/config
chmod 600 ~/.ssh/gitlab_deploy
Step 4 — Change remote URL from HTTPS to SSH:
bashcd /var/www/florist_multibranch

# Check current remote
git remote -v

# Change to SSH
git remote set-url origin git@gitlab.com:your-username/your-repo.git

# Test it works
git fetch
Step 5 — Verify git pull works without any password:
bashgit pull origin main
# Should pull without asking anything
Once this works, stop the golden EC2 and bake a new AMI — the SSH key and config will be baked in.

