# Deployment Guide: car-craft-lab-backend

This guide covers deployment using GitHub Actions and AWS EC2.

## 1. Prerequisites
- AWS EC2 instance (Ubuntu recommended)
- Node.js, npm, and PM2 installed on EC2
- SSH access to EC2
- GitHub repository secrets configured:
  - `EC2_PRODUCTION_HOST`, `EC2_SSH_PRIVATE_KEY`, `SLACK_WEBHOOK_URL`, etc.

## 2. GitHub Actions Workflow
The project uses `.github/workflows/deploy.yml` to automate deployment:
- Runs tests and linting
- Deploys to EC2 via SSH on push to `main`
- Installs dependencies, updates environment, and restarts with PM2

## 3. Manual Deployment Steps (Shell Script)
You can also deploy manually using SSH and the following script:

```sh
#!/bin/bash
set -e
cd /var/www/backend/car-craft-lab-backend

echo "Pulling latest code..."
git pull origin main

echo "Installing dependencies..."
NODE_ENV=production npm ci

echo "Updating environment variables..."
cat <<EOF > .env
NODE_ENV=production
PORT=5000
MONGODB_URI=your-mongodb-uri
JWT_SECRET=your-jwt-secret
CORS_ORIGIN=https://car-craft-lab-main.vercel.app
EOF

echo "Ensuring PM2 is installed..."
if ! command -v pm2 &> /dev/null; then
  npm install -g pm2
fi

echo "Restarting app with PM2..."
pm2 start ecosystem.config.js || pm2 restart backend
pm2 save

echo "Deployment complete."
```

## 4. Useful Commands
- Start app: `pm2 start ecosystem.config.js`
- Restart app: `pm2 restart backend`
- View logs: `pm2 logs backend`
- Check status: `pm2 status`

## 5. Troubleshooting
- Ensure `.env` is correct and up to date
- Check PM2 logs for errors
- Make sure all secrets are set in GitHub repository settings

---
For more details, see the comments in `.github/workflows/deploy.yml` and the shell script above.
