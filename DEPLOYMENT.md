# Deployment Guide for Inves4Business (Hostinger VPS)

This guide outlines the steps to deploy your full-stack Inves4Business application to a Hostinger VPS using **Nginx**, **PM2**, and **SSL**.

## 1. Prepare the Application for Production

### Build the Frontend
Run the following command in your local environment to generate the production build:
```bash
npm run build
```
This will create a `dist/` folder containing your static frontend files.

### Environment Variables
Ensure you have all required secrets ready for the VPS:
- `GEMINI_API_KEY`
- `RAZORPAY_KEY_ID`
- `RAZORPAY_KEY_SECRET`
- `FIREBASE_SERVICE_ACCOUNT` (Single-line JSON string)

---

## 2. Set Up Hostinger VPS

### Install Dependencies
Connect to your VPS via SSH and install the necessary tools:
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Node.js (LTS)
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt-get install -y nodejs

# Install Nginx
sudo apt install nginx -y

# Install PM2 (Process Manager)
sudo npm install -g pm2
```

---

## 3. Deploy the Code

1. **Upload Files**: Use SCP or Git to move your project files to `/var/www/inves4business`.
2. **Install Packages**:
   ```bash
   cd /var/www/inves4business
   npm install --production
   ```
3. **Configure Environment**: Create a `.env` file in the root directory and add your production secrets.

---

## 4. Start the Application

We have optimized the build process to produce a single, bundled production server file for maximum compatibility.

1. **Build the project**:
   ```bash
   npm run build
   ```
   This generates the `dist/` folder (frontend) and `server-prod.js` (backend).

2. **Start the server**:
   ```bash
   pm2 start server-prod.js --name "inves-app"
   ```

If you are using **Hostinger's Managed Node.js Panel** (instead of a VPS):
1. Go to **Advanced** -> **Node.js**.
2. Set the **Entry File** to `server-prod.js`.
3. Set the **Root Directory** to the folder containing your `package.json`.
4. Run `npm install` and then `npm run build` from the panel's terminal (or your git deployment).
5. Click **Start/Restart**.

---

## 5. Configure Nginx as Reverse Proxy

Create an Nginx configuration file:
```bash
sudo nano /etc/nginx/sites-available/inves4business
```

Paste the following configuration (replace `yourdomain.com` with your actual domain):
```nginx
server {
    listen 80;
    server_name yourdomain.com;

    # Frontend Static Files
    location / {
        root /var/www/inves4business/dist;
        index index.html;
        try_files $uri /index.html;
    }

    # Backend API Proxy
    location /api {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Enable the site and restart Nginx:
```bash
sudo ln -s /etc/nginx/sites-available/inves4business /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

---

## 6. Secure with SSL (Certbot)

Install Certbot and obtain an SSL certificate:
```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d yourdomain.com
```

---

## 7. Final Verification
- Visit `https://yourdomain.com` to see your live app.
- Test the Login/Register flow.
- Ensure Razorpay payments are working by checking the `/api/payment/create` endpoint.
