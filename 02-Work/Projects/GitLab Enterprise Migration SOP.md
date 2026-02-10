## Ubuntu 20.04 (Omnibus) → Ubuntu 22.04 (Docker Compose)

**Version:** 1.0
**GitLab Version:** 18.8.2 EE
**Database:** RDS PostgreSQL 16.10
**Backup Size:** ~105GB
**Users:** 200+

---

## Table of Contents
1. [Pre-Migration Preparation](#pre-migration-preparation)
2. [Phase 1: New Server Setup](#phase-1-new-server-setup)
3. [Phase 2: Testing Phase](#phase-2-testing-phase)
4. [Phase 3: Final Migration](#phase-3-final-migration)
5. [Phase 4: Cutover & Verification](#phase-4-cutover--verification)
6. [Rollback Procedures](#rollback-procedures)
7. [Post-Migration Tasks](#post-migration-tasks)

---

## Pre-Migration Preparation

### 1.1 Documentation Gathering
[Clone gitlab-automation scripts](https://gitlab.761link.net/enterprise/gitlab-automation/-/tree/docker?ref_type=heads)
```
git clone https://gitlab.761link.net/enterprise/gitlab-automation.git
```

Collect and document the following from the current Ubuntu 20.04 server:
- [ ] Current GitLab external URL: `gitlab.761link.net`
- [ ] Current server IP: `10.51.30.165`
- [ ] RDS PostgreSQL endpoint: `In rb file` (store securely)
- [ ] RDS database name: `In rb file`
- [ ] RDS username: In rb file
- [ ] RDS password: `In rb file`
**S3 Configuration Details:**
- [ ] AWS Account: `sunet-enterprise-govcloud`
- [ ] Backup bucket name: `ent-gitlab-uploads`
- [ ] Packages bucket name: `ent-gitlab-packages`
- [ ] LFS bucket name: `ent-gitlab-lfs`
- [ ] Registry bucket name: `ent-gitlab-registry`
- [ ] AWS Access Key ID: `In rb file` (store securely)
- [ ] AWS Secret Access Key: `in rb file` (store securely)

### 1.2 Backup Current Configuration Files

On Ubuntu 20.04 server:

```bash
# Create backup directory
sudo mkdir -p /root/gitlab-migration-backup
cd /root/gitlab-migration-backup

# Backup GitLab configuration
sudo cp /etc/gitlab/gitlab.rb gitlab.rb.backup
sudo cp /etc/gitlab/gitlab-secrets.json gitlab-secrets.json.backup

# Backup SSL certificates
sudo mkdir -p ssl-certs
sudo cp -r /etc/ssl/certs/your-gitlab-cert* ./ssl-certs/
# Note: Copy all relevant certificate files and private keys

# Copy these files to a secure location OFF the server from 192.168.120.2
sudo scp robert.smith@10.51.30.165:/home/robert.smith/gitlab.rb.backup .
sudo scp robert.smith@10.51.30.165:/home/robert.smith/gitlab-secrets.json.backup .
sudo scp -r robert.smith@10.51.30.165:/home/robert.smith/ssl-certs/ .
```

### 1.3 Verify Current Backup

```bash
# Check latest backup in S3
aws s3 ls s3://ent-gitlab-uploads/ --recursive | sort | tail -n 10

# Note the latest backup timestamp: ex 1770256882_2026_02_04_18.8.2-ee_gitlab_backup.tar

# Create a pre-migration backup (incremental — skips object-store blobs since those stay in S3)
sudo gitlab-backup create SKIP=uploads,artifacts,lfs,registry,packages BACKUP=manual-pre-migration

# Verify upload to S3
aws s3 ls s3://ent-gitlab-uploads/ --recursive | sort | tail -n 10
```

### 1.4 Document Current Settings

```bash
# Get current GitLab version
sudo gitlab-rake gitlab:env:info

# Document all external URLs and settings
sudo gitlab-rake gitlab:check

# Save output to file
sudo gitlab-rake gitlab:env:info > /root/gitlab-migration-backup/gitlab-info.txt

# Record user and project counts for post-migration verification
sudo gitlab-rails runner "puts 'Users: ' + User.count.to_s"
sudo gitlab-rails runner "puts 'Projects: ' + Project.count.to_s"
```

**Counts from old server (fill in for verification later):**
- User count: `_____________________`
- Project count: `_____________________`

---

## Phase 1: New Server Setup

### 2.1 Prepare Ubuntu 22.04 Server
#### Already configured on 10.51.30.170

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install required packages
sudo apt install -y curl ca-certificates gnupg lsb-release awscli

# Install Docker
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Verify Docker installation
sudo docker --version
sudo docker compose version

# Add user to docker group (optional)
sudo usermod -aG docker $USER
# Log out and back in for group changes to take effect
```

### 2.2 Create Directory Structure
No longer needed Jaleels configure-gitlab-server.sh does this. 

```bash
# Create GitLab directories 
###sudo mkdir -p /etc/gitlab/{config,logs,data,data/backups}
```

### 2.3 Copy SSL Certificates

Transfer your SSL certificates from the old server:

```bash
# From old server, copy certificates
scp -r robert.smith@10.51.30.165:/home/robert.smith/ssl-certs/* /tmp/

# On new server, move to GitLab config directory (nginx reads from /etc/gitlab inside the container)
sudo mkdir -p /etc/gitlab/config/ssl
sudo mv /tmp/your-gitlab-cert.crt /etc/gitlab/config/ssl/
sudo mv /tmp/your-gitlab-cert.key /etc/gitlab/config/ssl/
sudo chmod 600 /etc/gitlab/config/ssl/*.key
sudo chmod 644 /etc/gitlab/config/ssl/*.crt
```

**Certificate files copied:**
- [ ] Certificate file: `_____________________`
- [ ] Private key file: `_____________________`

### 2.4 Copy gitlab-secrets.json

**Critical:** The secrets file encrypts stored credentials (CI variables, 2FA keys, etc.). Without the original secrets file, encrypted data in the database is unrecoverable.
Run this on 10.51.30.170

```bash
# Copy from old server
sudo scp robert.smith@10.51.30.165:/home/robert.smith/gitlab-secrets.json.backup /tmp/gitlab-secrets.json

# Move to GitLab config directory
sudo mv /tmp/gitlab-secrets.json /etc/gitlab/config/
sudo chmod 600 /etc/gitlab/config/gitlab-secrets.json
```

### 2.5 Configure gitlab.rb

Copy the `gitlab.rb` from the old server and adapt it for Docker. The `gitlab.rb` lives at `/etc/gitlab/config/gitlab.rb` on the host (mounted to `/etc/gitlab/gitlab.rb` inside the container).
Run this on 10.51.30.170

```bash
# Copy from old server
scp robert.smith@10.51.30.165:/home/robert.smith/gitlab.rb.backup /etc/gitlab/config/gitlab.rb
```

**Verify the following sections are present and correct in `/etc/gitlab/config/gitlab.rb`:**

- [ ] `external_url` is set to `https://gitlab.761link.net`
- [ ] PostgreSQL disabled (`postgresql['enable'] = false`) and RDS connection configured
- [ ] S3 object storage configured (uploads, artifacts, LFS, packages, registry)
- [ ] S3 backup upload configured (`backup_upload_connection`, `backup_upload_remote_directory`)
- [ ] LDAP configuration present and correct
- [ ] SSL certificate paths point to `/etc/gitlab/ssl/` (the container-internal path)
- [ ] `gitlab_sshd['enable'] = false` (SSH is not exposed in this setup)
- [ ] `prometheus_monitoring['enable'] = false` (optional, if not using built-in monitoring)

> **Note:** All configuration is managed through `gitlab.rb`, not through the `GITLAB_OMNIBUS_CONFIG` environment variable in docker-compose. The compose file is kept minimal. This makes configuration changes easier to manage, diff, and back up.

### 2.6 Create Docker Compose Files

#### Production Compose: `/etc/gitlab/docker-compose.yml`

Used for normal operation. Binds to the server's IP, enforces resource limits, and restarts automatically.

```yaml
services:
  gitlab:
    image: gitlab/gitlab-ee:18.8.2-ee.0
    container_name: gitlab
    restart: always
    hostname: 'gitlab.761link.net'
    ports:
      - '${IP_ADDRESS}:443:443'
    volumes:
      - '$GITLAB_HOME/config:/etc/gitlab'
      - '$GITLAB_HOME/logs:/var/log/gitlab'
      - '$GITLAB_HOME/data:/var/opt/gitlab'
    security_opt:
      - no-new-privileges
    cpus: "6.0"
    cpu_shares: 2048
    mem_limit: 28g
    pids_limit: 300
    healthcheck:
      test: ["CMD", "curl", "-kfsS", "http://localhost/-/health"]
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 5m
```

**Environment variables** (set by `start-gitlab.sh` or export manually):
- `GITLAB_HOME` — Host path for GitLab data (default: `/etc/gitlab`)
- `IP_ADDRESS` — Server IP to bind port 443 to (auto-detected by `start-gitlab.sh`)

#### Startup/Restore Compose: `/etc/gitlab/docker-compose-startup.yml`

Used for first startup and backup restores. Does **not** restart on failure, and blocks CI runner job requests so runners don't pick up jobs during restore.

```yaml
# Used for first gitlab startup or gitlab restore
services:
  gitlab:
    image: gitlab/gitlab-ee:18.8.2-ee.0
    container_name: gitlab
    restart: "no"
    hostname: 'gitlab.761link.net'
    ports:
      - '443:443'
    volumes:
      - '$GITLAB_HOME/config:/etc/gitlab'
      - '$GITLAB_HOME/logs:/var/log/gitlab'
      - '$GITLAB_HOME/data:/var/opt/gitlab'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        nginx['custom_gitlab_server_config'] = "
          location ^~ /api/v4/jobs/request {
            return 503;
          }
        "
```

> **Why two compose files?** The startup compose uses `restart: "no"` so the container won't restart in a loop if reconfigure fails. It also blocks runner API requests so CI jobs don't execute against a partially-restored instance. Once restore is verified, switch to the production compose.

### 2.7 Deploy Scripts

Copy the backup, restore, and configure scripts to the new server:

```bash
# Copy scripts to the new server
sudo scp scripts/create-gitlab-backup.sh robert.smith@10.51.30.170:/tmp/
sudo scp scripts/restore-latest-gitlab-backup.sh robert.smith@10.51.30.170:/tmp/
sudo scp scripts/configure-gitlab-server.sh robert.smith@10.51.30.170:/tmp/
sudo  scp start-gitlab.sh robert.smith@10.51.30.170:/tmp/

# On the new server — place scripts
sudo mv /tmp/create-gitlab-backup.sh /etc/gitlab/
sudo mv /tmp/restore-latest-gitlab-backup.sh /etc/gitlab/
sudo mv /tmp/configure-gitlab-server.sh /etc/gitlab/
sudo mv /tmp/start-gitlab.sh /etc/gitlab/
sudo chmod 750 /etc/gitlab/*.sh
```

### 2.8 Run Server Configuration Script

```bash
cd /etc/gitlab
sudo ./configure-gitlab-server.sh
```

This script will:
- Create required directories (`/etc/gitlab`, `/etc/gitlab/data/backups`)
- Install the backup script to `/usr/local/bin/create-gitlab-backup.sh`
- Add a `gitlab-container-shell` alias to root's `.bashrc`
- Install a cron job for weekly backups (Thursday 3:00 PM)
- Disable `gitlab_sshd` in `gitlab.rb` if it exists

---

## Phase 2: Testing Phase

### 3.1 Start GitLab Container (Startup Mode)

Use the startup compose file for the initial bring-up. This blocks CI runner requests and won't auto-restart on failure.

```bash
cd /etc/gitlab
export GITLAB_HOME="/etc/gitlab"

# Start container in startup/restore mode
sudo -E docker compose -f docker-compose-startup.yml up -d

# Monitor startup (this will take 5-10 minutes)
sudo docker compose -f docker-compose-startup.yml logs -f

# Wait for "gitlab Reconfigured!" message
# Press Ctrl+C to exit logs when ready
```

### 3.2 Verify Container Health

```bash
# Check container status
sudo docker ps

# Container should be "running" (healthy)

# Check GitLab health
sudo docker exec -it gitlab gitlab-rake gitlab:check

# Check database connectivity
sudo docker exec -it gitlab gitlab-rake gitlab:db:configure
```

### 3.3 Restore Backup for Testing

```bash
# Download latest backup from S3 to the backups directory
aws s3 cp s3://ent-gitlab-uploads/LATEST_BACKUP_FILE.tar /etc/gitlab/data/backups/

# Verify the file is in place
ls -lh /etc/gitlab/data/backups/

# Run a dry-run first to validate everything without modifying data
cd /etc/gitlab
sudo RESTORE_MODE=overwrite ./restore-latest-gitlab-backup.sh latest

# If using RDS with an existing database, set overwrite mode:
# sudo RESTORE_MODE=overwrite ./restore-latest-gitlab-backup.sh latest

# To auto-create required PostgreSQL extensions if missing:
# sudo RESTORE_MODE=overwrite AUTO_CREATE_EXTENSIONS=true ./restore-latest-gitlab-backup.sh latest
```

The restore script will:
1. Validate the backup archive integrity
2. Check GitLab version compatibility between backup and container
3. Test database connectivity
4. Check for required PostgreSQL extensions (`pg_trgm`, `btree_gist`)
5. Stop puma and sidekiq
6. Start Redis
7. Restore the backup
8. Reconfigure GitLab
9. Disable sshd to prevent start timeouts
10. Start all services
11. Run final sanity checks

**This will take a significant amount of time depending on backup size (~105GB).**

Monitor progress in another terminal:
```bash
# Check which tables are being restored and row counts
docker exec gitlab gitlab-psql -d gitlabhq_production -c "
SELECT
  now(),
  relname,
  n_live_tup
FROM pg_stat_user_tables
ORDER BY n_live_tup DESC
LIMIT 10;
"

# Confirm the restore process isn't stuck (look for ruby, pg_restore, psql)
docker exec gitlab ps -eo pid,pcpu,pmem,cmd | sort -k2 -r | head
```

### 3.4 Switch to Production Compose and Verify

After restore completes successfully:

```bash
cd /etc/gitlab

# Stop the startup container
sudo docker compose -f docker-compose-startup.yml down

# Start with the production compose using start-gitlab.sh
sudo ./start-gitlab.sh

# Monitor startup
sudo docker logs -f gitlab
```

### 3.5 Test Access and Functionality

**Add test entry to `/etc/hosts` on your local machine:**
```
10.51.30.170    gitlab.761link.net
```

Access GitLab in browser:
- [ ] GitLab UI loads correctly at `https://gitlab.761link.net`
- [ ] Can log in with existing credentials
- [ ] LDAP authentication works
- [ ] Can view existing projects
- [ ] Can clone a repository via HTTPS
- [ ] Can view CI/CD pipelines
- [ ] Can view issues/merge requests
- [ ] Artifacts are accessible
- [ ] LFS files are accessible

```bash
# Test git clone
git clone https://gitlab.761link.net/test-group/test-repo.git

# Check for any errors in logs
sudo docker logs gitlab 2>&1 | grep -i error | tail -20
sudo docker logs gitlab 2>&1 | grep -i warning | tail -20
```

### 3.6 Performance Verification

```bash
# Check resource usage
sudo docker stats --no-stream

# Check GitLab environment info
sudo docker exec -it gitlab gitlab-rake gitlab:env:info

# Verify user and project counts match old server
sudo docker exec -it gitlab gitlab-rails runner "puts 'Users: ' + User.count.to_s"
sudo docker exec -it gitlab gitlab-rails runner "puts 'Projects: ' + Project.count.to_s"
```

**Document any issues found:**
```
Issue 1: _____________________
Resolution: _____________________

Issue 2: _____________________
Resolution: _____________________
```

### 3.7 Stop Test Environment

Once testing is complete and satisfactory:

```bash
cd /etc/gitlab
sudo docker compose down

# Keep backups and data intact for final migration
```

---

## Phase 3: Final Migration

### 4.1 Prepare for Downtime

**Scheduled Maintenance Window:**
- Start time: _________________ (Date/Time)
- Expected duration: 4-6 hours
- End time: _________________ (Date/Time)

### 4.2 Put Old Server in Maintenance Mode

On Ubuntu 20.04 server:

```bash
# Enable maintenance mode
sudo gitlab-ctl deploy-page up

# Verify maintenance page is showing
curl -I https://gitlab.761link.net

# Stop all GitLab services to prevent new data
sudo gitlab-ctl stop

# Verify all stopped
sudo gitlab-ctl status
```

### 4.3 Create Final Backup

```bash
# On old server — create final migration backup (ONLY SKIP S3 buckets)
sudo gitlab-backup create SKIP=uploads,artifacts,lfs,registry,packages BACKUP=final-migration

# Wait for completion, then verify
ls -lh /var/opt/gitlab/backups/

# Note final backup filename: _____________________

# Verify upload to S3 (should happen automatically via gitlab.rb config)
aws s3 ls s3://ent-gitlab-uploads/ | grep final-migration
```

### 4.4 Verify RDS Database State
#### We can skip this, but good test to confirm we are on RDS and not bundled DB. 
```bash
# Connect to RDS and verify last update times
sudo gitlab-rails dbconsole

# In PostgreSQL console:
SELECT schemaname, tablename, last_vacuum, last_analyze
FROM pg_stat_user_tables
ORDER BY last_analyze DESC LIMIT 10;

\q
```

### 4.5 Make sure we still have gitlab.rb and secrets backed up. 

---

## Phase 4: Cutover & Verification

### 5.1 Start New Server (Startup Mode)

```bash
# On Ubuntu 22.04 server
cd /etc/gitlab

# Ensure gitlab-secrets.json is in place
ls -l /etc/gitlab/config/gitlab-secrets.json

# Start container in startup/restore mode
export GITLAB_HOME="/etc/gitlab"
sudo -E docker compose -f docker-compose-startup.yml up -d

# Monitor startup
sudo docker compose -f docker-compose-startup.yml logs -f
```

### 5.2 Restore Final Backup

```bash
# Download final backup from S3
aws s3 cp s3://ent-gitlab-uploads/final-migration_gitlab_backup.tar /etc/gitlab/data/backups/

# Restore using the restore script
cd /etc/gitlab
sudo RESTORE_MODE=overwrite AUTO_CREATE_EXTENSIONS=true ./restore-latest-gitlab-backup.sh final-migration_gitlab_backup.tar
```

### 5.3 Switch to Production Compose

```bash
cd /etc/gitlab

# Stop the startup container
sudo docker compose -f docker-compose-startup.yml down

# Start with production compose
sudo ./start-gitlab.sh

# Wait for healthy status
sudo docker logs -f gitlab
```

### 5.4 Verification Checks

```bash
# Full system check (run on 10.51.30.170 while still testing)
sudo docker exec -it gitlab gitlab-rake gitlab:check

# Check all services
sudo docker exec -it gitlab gitlab-ctl status

# Verify data integrity
sudo docker exec -it gitlab gitlab-rake gitlab:artifacts:check
sudo docker exec -it gitlab gitlab-rake gitlab:lfs:check

# Check user count
sudo docker exec -it gitlab gitlab-rails runner "puts 'Users: ' + User.count.to_s"
# Expected count: _________________ (from section 1.4)

# Check project count
sudo docker exec -it gitlab gitlab-rails runner "puts 'Projects: ' + Project.count.to_s"
# Expected count: _________________ (from section 1.4)
```

### 5.5 Create AMIs and IP Cutover

**CRITICAL: This is the point of no return. No Elastic IPs are available — the IP cutover is performed by terminating the old instance and relaunching the new AMI with the freed private IP (`10.51.30.165`).**

#### Step 1: Create AMI of Old Server (10.51.30.165) — Safety Backup

Before touching the old server, snapshot it so you can recover it if needed.
You can also do this through the UI. Just please document the AMI name. 

```bash
# Get the instance ID of the OLD server (10.51.30.165) — run from any host with AWS CLI access
OLD_INSTANCE_ID=$(aws ec2 describe-instances \
  --filters "Name=private-ip-address,Values=10.51.30.165" \
  --query "Reservations[0].Instances[0].InstanceId" \
  --output text)
echo "Old instance ID: $OLD_INSTANCE_ID"

# Create AMI of old server (no-reboot to avoid downtime on the source)
aws ec2 create-image \
  --instance-id "$OLD_INSTANCE_ID" \
  --name "gitlab-20-04-pre-cutover-$(date +%Y%m%d)" \
  --description "GitLab Ubuntu 20.04 old server backup before IP cutover" \
  --no-reboot

# Note the AMI ID returned — do NOT proceed until this AMI is in "available" state
# Old server AMI ID: _____________________

# Poll until available (may take 10-20 minutes)
aws ec2 wait image-available --image-ids ami-XXXXXXXXXXXXXXXXX
echo "Old server AMI ready"
```

#### Step 2: Create AMI of New Server (10.51.30.170) — Deploy Image
You can also do this through the UI. Just please document the AMI name. 

```bash
# Get the instance ID of the NEW server (10.51.30.170)
NEW_INSTANCE_ID=$(aws ec2 describe-instances \
  --filters "Name=private-ip-address,Values=10.51.30.170" \
  --query "Reservations[0].Instances[0].InstanceId" \
  --output text)
echo "New instance ID: $NEW_INSTANCE_ID"

# Stop containers cleanly before AMI creation to ensure filesystem consistency
ssh robert.smith@10.51.30.170 "cd /etc/gitlab && sudo docker compose down"

# Create AMI of new server
aws ec2 create-image \
  --instance-id "$NEW_INSTANCE_ID" \
  --name "gitlab-22-04-deploy-$(date +%Y%m%d)" \
  --description "GitLab Ubuntu 22.04 image to deploy on 10.51.30.165"

# Note the AMI ID — do NOT proceed until this AMI is in "available" state
# New server AMI ID: _____________________

# Poll until available
aws ec2 wait image-available --image-ids ami-XXXXXXXXXXXXXXXXX
echo "New server AMI ready — safe to proceed with cutover"
```

#### Step 3: IP Cutover — Terminate Old Instance and Relaunch from New AMI

> **Important:** Private IPs in AWS are tied to an ENI. To assign `10.51.30.165` to a new instance, the original instance holding that IP must be **terminated** (not just stopped). Confirm the old-server AMI (Step 1) is `available` before proceeding.

```bash
# 1. Put old GitLab in maintenance mode (if not already stopped from section 4.2) Its better to just have it stopped the whole time. 
ssh robert.smith@10.51.30.165 "sudo gitlab-ctl deploy-page up && sudo gitlab-ctl stop"

# 2. Gather subnet and security group from the old instance before termination
#    (collect these BEFORE step 3 below — left here for reference)
# Subnet ID:          _____________________
# Security Group IDs: _____________________
# Key pair name:      _____________________
# IAM instance profile (if any): _____________________

# 3. Terminate the old instance to release the 10.51.30.165 private IP
## Go into the UI and delete, we probably have termination protection on. 
aws ec2 terminate-instances --instance-ids "$OLD_INSTANCE_ID"


# Wait for termination — the private IP is not available until the instance is fully terminated
aws ec2 wait instance-terminated --instance-ids "$OLD_INSTANCE_ID"
echo "Old instance terminated — 10.51.30.165 is now free"

# 4. Launch new instance from the 10.51.30.170 AMI, assigning 10.51.30.165 as the private IP
aws ec2 run-instances \
  --image-id ami-XXXXXXXXXXXXXXXXX \
  --instance-type <INSTANCE_TYPE> \
  --subnet-id subnet-XXXXXXXXX \
  --security-group-ids sg-XXXXXXXXX \
  --private-ip-address 10.51.30.165 \
  --key-name <KEY_PAIR_NAME> \
  --iam-instance-profile Name=<PROFILE_NAME> \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=gitlab-prod}]'

# Note the new instance ID
# New instance ID (at 10.51.30.165): _____________________

# Wait for the instance to be running
aws ec2 wait instance-running --instance-ids i-XXXXXXXXXXXXXXXXX
echo "New GitLab instance is running at 10.51.30.165"
```

**Final server IP:** `10.51.30.165` (same as the old server — no DNS change needed)

### 5.6 Post-Cutover Verification

```bash
# SSH into the new instance at 10.51.30.165 and start GitLab
ssh robert.smith@10.51.30.165 "cd /etc/gitlab && sudo ./start-gitlab.sh"

# Monitor startup
ssh robert.smith@10.51.30.165 "sudo docker logs -f gitlab"

# From external machine, test connectivity (no DNS change needed — IP is the same as old server)
curl -I https://gitlab.761link.net

# Should return 200 OK

# Test git operations
git clone https://gitlab.761link.net/test-group/test-repo.git
cd test-repo
echo "test" > test.txt
git add test.txt
git commit -m "Post-migration test"
git push origin main
```

**Verification Checklist:**
- [ ] GitLab UI accessible
- [ ] Users can log in successfully
- [ ] LDAP authentication working
- [ ] Git clone via HTTPS works
- [ ] Git push works
- [ ] Can create new projects
- [ ] Can create new issues/MRs
- [ ] CI/CD pipelines trigger correctly
- [ ] Artifacts download successfully
- [ ] LFS files accessible
- [ ] Container Registry accessible
- [ ] Webhooks firing correctly
- [ ] Email notifications sending

### 5.7 Re-enable Backups
**I think cron job create-gitlab-backup.sh script needs updatign to SKIP=uploads,artifacts,lfs,registry,packages**
The cron job was already installed by `configure-gitlab-server.sh` in section 2.8. Verify it exists and run a manual test:

```bash
# Verify cron job exists
sudo crontab -l | grep gitlab

# Expected: 0 15 * * 4 /usr/local/bin/create-gitlab-backup.sh >> /var/log/gitlab-backup.log 2>&1

# Test backup manually
sudo /usr/local/bin/create-gitlab-backup.sh

# Verify backup created
ls -lh /etc/gitlab/data/backups/

# Verify backup uploaded to S3 (if configured in gitlab.rb)
aws s3 ls s3://ent-gitlab-uploads/ --recursive | tail -n 5
```

---

## Rollback Procedures

### In Case of Critical Issues During Migration

#### Rollback Option 1: Revert to Old Server (Before IP Cutover)

```bash
# On Ubuntu 20.04 server
sudo gitlab-ctl start

# Disable maintenance mode
sudo gitlab-ctl deploy-page down

# Verify services
sudo gitlab-ctl status

# Test access
curl -I https://gitlab.761link.net
```

#### Rollback Option 2: Restore Old Server from AMI (After IP Cutover)

If critical issues are found after cutover and the new instance at `10.51.30.165` cannot be recovered:

```bash
# 1. Stop GitLab on the current (failed) instance at 10.51.30.165
ssh robert.smith@10.51.30.165 "cd /etc/gitlab && sudo docker compose down" || true

# 2. Terminate the current instance to free 10.51.30.165
CURRENT_INSTANCE_ID=$(aws ec2 describe-instances \
  --filters "Name=private-ip-address,Values=10.51.30.165" \
  --query "Reservations[0].Instances[0].InstanceId" \
  --output text)
aws ec2 terminate-instances --instance-ids "$CURRENT_INSTANCE_ID"
aws ec2 wait instance-terminated --instance-ids "$CURRENT_INSTANCE_ID"

# 3. Relaunch from the OLD server AMI (gitlab-20-04-pre-cutover-* created in step 5.5)
#    This restores the Ubuntu 20.04 Omnibus instance at 10.51.30.165
aws ec2 run-instances \
  --image-id ami-XXXXXXXXXXXXXXXXX \
  --instance-type <INSTANCE_TYPE> \
  --subnet-id subnet-XXXXXXXXX \
  --security-group-ids sg-XXXXXXXXX \
  --private-ip-address 10.51.30.165 \
  --key-name <KEY_PAIR_NAME> \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=gitlab-rollback}]'

# 4. Once running, re-enable GitLab on the restored instance
ssh robert.smith@10.51.30.165 "sudo gitlab-ctl start && sudo gitlab-ctl deploy-page down"

# 5. Verify access
curl -I https://gitlab.761link.net
```

#### Rollback Option 3: Restore Previous Backup on New Server

```bash
cd /etc/gitlab

# Download the previous backup from S3
aws s3 cp s3://ent-gitlab-uploads/PREVIOUS_BACKUP_FILE.tar /etc/gitlab/data/backups/

# Switch to startup compose
sudo docker compose down
export GITLAB_HOME="/etc/gitlab"
sudo -E docker compose -f docker-compose-startup.yml up -d

# Wait for container to be running, then restore
sudo RESTORE_MODE=overwrite ./restore-latest-gitlab-backup.sh PREVIOUS_BACKUP_FILE.tar

# Switch back to production compose
sudo docker compose -f docker-compose-startup.yml down
sudo ./start-gitlab.sh
```

---

## Post-Migration Tasks

### 7.1 Update Runner Configuration

If runners need to re-register or update URL:

```bash
# On each runner server, update gitlab-runner config
sudo vi /etc/gitlab-runner/config.toml

# Verify url = "https://gitlab.761link.net"
# Restart runner
sudo gitlab-runner restart
sudo gitlab-runner verify 
```

### 7.2 Update Documentation

- [ ] Conops? 
- [ ] Enter docs that need to be updated. 

### 7.3 Monitor

```bash
# Check logs daily
sudo docker logs --tail=100 gitlab

# Monitor resource usage
sudo docker stats --no-stream

# Check for errors
sudo docker exec -it gitlab gitlab-rake gitlab:check

# Quick health check
sudo docker exec gitlab curl -kfsS http://localhost/-/health

# Monitor RDS performance
# Check AWS CloudWatch metrics for database
```

**Issues Log (First Week):**
```
Day 1: _____________________
Day 2: _____________________
Day 3: _____________________
Day 4: _____________________
Day 5: _____________________
Day 6: _____________________
Day 7: _____________________
```

### 7.5 Optimize New Environment
Jaleel are these needed in your docker compose file? 
```bash
# After confirming stability, review Docker resource limits in docker-compose.yml:
# - cpus: "6.0"
# - mem_limit: 28g
# - pids_limit: 300
# - shm_size (not currently set — add if shared memory issues occur)

# Consider enabling additional monitoring or CloudWatch logs if needed
```

---

## Appendix A: Useful Commands

### Docker Management
```bash
# View logs
sudo docker logs -f gitlab

# Enter container shell
sudo docker exec -it gitlab bash
# Or use the alias installed by configure-gitlab-server.sh:
gitlab-container-shell

# Restart specific service
sudo docker exec -it gitlab gitlab-ctl restart puma

# Check service status
sudo docker exec -it gitlab gitlab-ctl status

# Reconfigure GitLab
sudo docker exec -it gitlab gitlab-ctl reconfigure

# Health check
sudo docker exec gitlab curl -kfsS http://localhost/-/health
```

### GitLab Console
```bash
# Enter Rails console
sudo docker exec -it gitlab gitlab-rails console

# Common console commands:
# User.find_by_username('username')
# Project.find_by_full_path('group/project')
# ApplicationSetting.current.update(signup_enabled: false)
```

### Backup Commands
**review the skips in backup**
```bash
# Incremental backup (skips object-store blobs)
sudo /usr/local/bin/create-gitlab-backup.sh

# Full backup (includes everything)
BACKUP_TYPE=Full sudo /usr/local/bin/create-gitlab-backup.sh

# List local backups
ls -lh /etc/gitlab/data/backups/

# Restore latest backup (dry-run first)
DRY_RUN=true sudo /etc/gitlab/restore-latest-gitlab-backup.sh latest

# Restore latest backup (actual)
sudo RESTORE_MODE=overwrite /etc/gitlab/restore-latest-gitlab-backup.sh latest

# Restore specific backup
sudo RESTORE_MODE=overwrite /etc/gitlab/restore-latest-gitlab-backup.sh 1770256882_2026_02_04_18.8.2-ee_gitlab_backup.tar
```

### Database Commands
```bash
# Connect to PostgreSQL
sudo docker exec -it gitlab gitlab-rails dbconsole

# Check database size
sudo docker exec -it gitlab gitlab-rails runner "puts ActiveRecord::Base.connection.execute('SELECT pg_size_pretty(pg_database_size(current_database()));').first['pg_size_pretty']"
```

### Start/Stop
```bash
# Production start (using start-gitlab.sh)
cd /etc/gitlab && sudo ./start-gitlab.sh

# Production stop
cd /etc/gitlab && sudo docker compose down

# Startup/restore mode start
cd /etc/gitlab && export GITLAB_HOME="/etc/gitlab" && sudo -E docker compose -f docker-compose-startup.yml up -d

# Startup/restore mode stop
cd /etc/gitlab && sudo docker compose -f docker-compose-startup.yml down
```

---

## Appendix B: Troubleshooting

### Issue: GitLab won't start
```bash
# Check logs
sudo docker logs gitlab | tail -50

# Check config syntax
sudo docker exec -it gitlab gitlab-ctl reconfigure

# Verify gitlab-secrets.json is present
ls -l /etc/gitlab/config/gitlab-secrets.json

# Verify gitlab.rb is present
ls -l /etc/gitlab/config/gitlab.rb
```

### Issue: Can't connect to RDS
```bash
# Test connection from container
sudo docker exec -it gitlab bash
apt update && apt install -y postgresql-client
psql -h RDS_ENDPOINT -U DB_USER -d DB_NAME

# Check security groups in AWS
# Ensure the new instance IP (10.51.30.165) is allowed in the RDS security group
# Note: After cutover the instance runs at 10.51.30.165 — remove the old 10.51.30.170 rule if present
```

### Issue: S3 objects not accessible
```bash
# Test S3 access from the host
aws s3 ls s3://ent-gitlab-uploads/

# Check from inside the container
sudo docker exec -it gitlab gitlab-rails console
Gitlab::AppLogger.info(Gitlab.config.backup.upload.connection)
```

### Issue: LDAP not working
```bash
# Test LDAP connection
sudo docker exec -it gitlab gitlab-rake gitlab:ldap:check

# Debug LDAP
sudo docker exec -it gitlab gitlab-rails console
Gitlab::Auth::Ldap::Config.servers
```

### Issue: Container restarts in a loop
```bash
# Switch to startup compose (restart: "no") to debug
cd /etc/gitlab
sudo docker compose down
export GITLAB_HOME="/etc/gitlab"
sudo -E docker compose -f docker-compose-startup.yml up -d

# Check reconfigure output
sudo docker logs -f gitlab
```

### Issue: Restore script fails version check
```bash
# The restore script checks that backup major.minor version matches the running container
# Ensure the container image tag matches the backup version
# Current image: gitlab/gitlab-ee:18.8.2-ee.0
# If the backup is from a different minor version, you need the matching image
```

---

## Sign-off

**Date Completed:** _____________________
**Production Status:** ☐ Success ☐ Issues (document below)
**Issues/Notes:**
```
_____________________________________________________
_____________________________________________________
_____________________________________________________
```
