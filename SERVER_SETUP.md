# Server Setup Guide for Digital Ocean Droplet

## Prerequisites
- Droplet IP: `138.68.240.85`
- SSH key already added to Digital Ocean
- Domain: `deadentertainment.xyz`

## Step 1: Connect to Your Droplet

```bash
ssh root@138.68.240.85
```

Or if using the custom key:
```bash
ssh -i ~/.ssh/id_ed25519_deadentertainment root@138.68.240.85
```

## Step 2: Run Initial Setup Script

On the droplet, run:
```bash
bash <(curl -s https://raw.githubusercontent.com/your-repo/deadentertainment.xyz/main/deploy.sh)
```

Or manually run the commands from `deploy.sh`:
```bash
# Update system
sudo apt-get update && sudo apt-get upgrade -y

# Install Node.js 20.x
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# Install PM2
sudo npm install -g pm2

# Install Nginx
sudo apt-get install -y nginx

# Install Git
sudo apt-get install -y git
```

## Step 3: Deploy Your Code

### Option A: Using Git (Recommended)

```bash
# Clone your repository
cd /var/www
git clone YOUR_REPO_URL deadentertainment.xyz
cd deadentertainment.xyz

# Install dependencies
npm install

# Create .env file
nano .env
# Add: CONTRACTOR_PORTAL_PASSWORD=your-secure-password

# Build the application
npm run build

# Start with PM2
pm2 start npm --name "deadentertainment" -- start

# Save PM2 configuration
pm2 save

# Setup PM2 to start on boot
pm2 startup
# Follow the instructions it gives you
```

### Option B: Using SCP (Upload Files)

From your local machine:
```bash
# From the project directory
scp -r -i ~/.ssh/id_ed25519_deadentertainment . root@138.68.240.85:/var/www/deadentertainment.xyz
```

Then on the server:
```bash
cd /var/www/deadentertainment.xyz
npm install
# Create .env file
npm run build
pm2 start npm --name "deadentertainment" -- start
pm2 save
pm2 startup
```

## Step 4: Configure Nginx

```bash
# Copy nginx configuration
sudo nano /etc/nginx/sites-available/deadentertainment.xyz
# Paste the contents from nginx.conf

# Create symlink
sudo ln -s /etc/nginx/sites-available/deadentertainment.xyz /etc/nginx/sites-enabled/

# Remove default site
sudo rm /etc/nginx/sites-enabled/default

# Test configuration
sudo nginx -t

# Reload Nginx
sudo systemctl reload nginx
```

## Step 5: Configure Firewall

```bash
# Allow HTTP and HTTPS
sudo ufw allow 'Nginx Full'
# Or individually:
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Enable firewall
sudo ufw enable

# Check status
sudo ufw status
```

## Step 6: Setup SSL (Optional but Recommended)

```bash
# Install Certbot
sudo apt-get install certbot python3-certbot-nginx -y

# Get SSL certificate
sudo certbot --nginx -d deadentertainment.xyz -d www.deadentertainment.xyz

# Auto-renewal is set up automatically
```

## Step 7: Add Fonts and Forms

Upload fonts to the server:
```bash
# Create fonts directory if needed
mkdir -p /var/www/deadentertainment.xyz/public/fonts

# Upload font files (use SCP or SFTP)
# BlackChant-Regular.ttf
# DigitalTech-Regular.ttf
```

Upload tax forms:
```bash
# Create forms directory if needed
mkdir -p /var/www/deadentertainment.xyz/public/forms

# Upload PDF files
# w9.pdf
# 1099.pdf
```

## Step 8: Verify Everything Works

1. Check PM2 status:
   ```bash
   pm2 status
   pm2 logs deadentertainment
   ```

2. Check Nginx status:
   ```bash
   sudo systemctl status nginx
   ```

3. Test the site:
   - Visit: http://138.68.240.85 (should work immediately)
   - Visit: http://deadentertainment.xyz (once DNS propagates)

## Useful Commands

```bash
# View PM2 logs
pm2 logs deadentertainment

# Restart application
pm2 restart deadentertainment

# Stop application
pm2 stop deadentertainment

# View Nginx logs
sudo tail -f /var/log/nginx/error.log
sudo tail -f /var/log/nginx/access.log

# Reload Nginx after config changes
sudo systemctl reload nginx
```

## Troubleshooting

### Application won't start
- Check logs: `pm2 logs deadentertainment`
- Verify .env file exists and has correct password
- Check Node.js version: `node -v` (should be 20.x)

### Nginx 502 Bad Gateway
- Check if app is running: `pm2 status`
- Check if app is listening on port 3000: `netstat -tlnp | grep 3000`
- Check Nginx error logs: `sudo tail -f /var/log/nginx/error.log`

### DNS not resolving
- Wait 24-48 hours for propagation
- Test directly: `dig @ns1.digitalocean.com deadentertainment.xyz`
- Check DNS propagation: https://dnschecker.org

