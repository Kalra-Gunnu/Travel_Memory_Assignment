🚀 Deploying a MERN App on AWS EC2 with Load Balancing & Cloudflare Domain

✅ 1. Launch EC2 Instance
1. Use Ubuntu as the OS.
2. Connect to your instance using PowerShell:
    ```bash
    ssh -i <your-key>.pem ubuntu@<EC2-Public-IP>
    ```

🔧 2. Initial Setup
1. Run the following commands:
    ```bash
    sudo apt update
    sudo apt install -y git nodejs npm nginx
    ```
📁 3. Clone the Repository
1. Clone the Repository
    ```bash
    sudo git clone https://github.com/UnpredictablePrashant/TravelMemory
    cd TravelMemory/backend
    ```
⚙️ 4. Backend Setup
1. Create a .env file with:
    ```txt
    DB_URI=<Your-MongoDB-URI>
    PORT=3001
    ``` 
2. Install dependencies:
    ```bash
    sudo npm install
    ```
3. Add to package.json under scripts:
    ```json
    "start": "dotenv -e .env node index.js"
    ```

🌐 5. Configure Nginx for Reverse Proxy
1. Edit /etc/nginx/sites-enabled/default to redirect traffic from port 80 to 3001:
    ```nginx
    server {
        listen 80;
        root /var/www/travel-frontend;
        server_name yourdomain.com www.yourdomain.com;

        location /trip {
            proxy_pass http://localhost:3001;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
        }
        location / {
            try_files: $uri /index.html
        }
    }
    ```
 
2. Then run:
    ```bash
    sudo systemctl reload nginx
    sudo systemctl restart nginx
    ```

♻️ 6. Run with PM2
1. Install PM2
    ```bash
    sudo npm install -g pm2
    ```
2. create ecosystem.config.js manually
    ```json
    module.exports = {
        apps: [
            {
                name: "backend-app",
                script: "index.js",
                watch: false,
                env: {
                    NODE_ENV: "production",
                    PORT: 3001
                },
                env.file: ".env",
            },
        ],
    };
    ```
3. Then start with
    ```bash 
    pm2 start ecosystem.config.js
    ```
4. Backend will now be available at:
    ```bash
    http://<EC2-IP>/trip
    ```

🧑‍💻 7. Frontend Setup
1. Go to frontend directory
    ```bash
    cd ../frontend
    ```
2. Create .env:
    ```txt
    REACT_BACKEND_URI=http://<EC2-IP>/trip
    ```
3. Edit src/url.js accordingly.
4. Build the app:
    ```bash
    sudo npm install
    sudo npm run build
    ```
5. Deploy it with:
    ```bash
    sudo cp -r build /var/www/travel-frontend
    ```
6. App now runs at:
    ```bash
    http://<EC2-IP>
    ```
________________________________________

📦 8. Create AMI & Launch More Instances
1. Create an AMI from this configured instance.
2. Launch additional instances using this AMI.
 
🧭 9. Create Application Load Balancer (ALB)
1. Add at least 2 Availability Zones.
2. Register all EC2 instances in ALB’s target group.
3. Add HTTP (80) and HTTPS (443) listeners.
 ____________________________________

🔗 10. Reconfigure Each Instance for ALB
1. In each instance:
    - Backend:
        ```bash
        pm2 start ecosystem.config.js
        ```
    - Frontend- Update .env and src/url.js:
        ```bash
        REACT_BACKEND_URI=http://<ALB-DNS-Name>
        ```
    - Rebuild:
        ```bash
        sudo npm run build
        sudo cp -r build /var/www/travel-frontend
        ```
________________________________________
🌐 11. Domain & Cloudflare Setup
1. Buy a domain (e.g., from GoDaddy).
2. Add domain to Cloudflare.
3. Update nameservers on domain provider to Cloudflare’s nameservers.
4. Request SSL Certificate from AWS ACM:
    - Domain name: *.yourdomain.com
    - ACM provides a CNAME record → add it in Cloudflare DNS (DNS Only).
5. Once validated, certificate will be issued.
6. Create the A name and CNAME record for the application
________________________________________
🔁 12. Update Frontend for HTTPS Domain
1. Update .env and src/url.js in frontend to:
    ```bash
    REACT_BACKEND_URI=https://www.yourdomain.com
    ```
2. Rebuild and redeploy:
    ```bash
    sudo rm -rf build /var/www/travel-frontend
    sudo npm run build
    sudo cp -r build /var/www/travel-frontend
    ```
________________________________________
✅ Final Step: Access the App
1. Go to:
    ```bash
    https://www.yourdomain.com
    ```
________________________________________
📝 Notes
1. Use pm2 save and pm2 startup to ensure PM2 apps run after reboot.
2. Use HTTPS to avoid Mixed Content issues (HTTP backend call on HTTPS frontend).
3. All frontend builds must be redone when .env or src/url.js changes.

