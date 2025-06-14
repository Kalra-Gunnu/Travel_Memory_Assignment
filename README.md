🚀 Deploying a MERN App on AWS EC2 with Load Balancing & Cloudflare Domain
✅ 1. Launch EC2 Instance
    Use Ubuntu as the OS.
    Connect to your instance using PowerShell:
        ssh -i <your-key>.pem ubuntu@<EC2-Public-IP>
🔧 2. Initial Setup
    Run the following commands:
        sudo apt update
        sudo apt install -y git nodejs npm nginx
📁 3. Clone the Repository
    sudo git clone https://github.com/UnpredictablePrashant/TravelMemory
    cd TravelMemory/backend
⚙️ 4. Backend Setup
    Create a .env file with:
        DB_URI=<Your-MongoDB-URI>
        PORT=3001
    Install dependencies:
        sudo npm install
    Add to package.json under scripts:
        "start": "dotenv -e .env node index.js"
🌐 5. Configure Nginx for Reverse Proxy
    Edit /etc/nginx/sites-enabled/default to redirect traffic from port 80 to 3001:
 
    Then run:
        sudo systemctl reload nginx
        sudo systemctl restart nginx
♻️ 6. Run with PM2
    sudo npm install -g pm2
    create ecosystem.config.js manually
 
    pm2 start ecosystem.config.js
 
    Backend will now be available at:
    http://<EC2-IP>/trip
 
🧑‍💻 7. Frontend Setup
    cd ../frontend
    Create .env:
        REACT_BACKEND_URI=http://<EC2-IP>/trip
 
    Edit src/url.js accordingly.
    Build the app:
        sudo npm install
        sudo npm run build
    Deploy it with:
        sudo cp -r build /var/www/travel-frontend
    App now runs at:
        http://<EC2-IP>
________________________________________
📦 8. Create AMI & Launch More Instances
    Create an AMI from this configured instance.
    Launch additional instances using this AMI.
 
🧭 9. Create Application Load Balancer (ALB)
    Add at least 2 Availability Zones.
    Register all EC2 instances in ALB’s target group.
    Add HTTP (80) and HTTPS (443) listeners.
 ____________________________________
🔗 10. Reconfigure Each Instance for ALB
    In each instance:
        Backend:
            pm2 start ecosystem.config.js
        Frontend:
            Update .env and src/url.js:
                REACT_BACKEND_URI=http://<ALB-DNS-Name>
        Rebuild:
            sudo npm run build
            sudo cp -r build /var/www/travel-frontend
________________________________________
🌐 11. Domain & Cloudflare Setup
    Buy a domain (e.g., from GoDaddy).
    Add domain to Cloudflare.
    Update nameservers on domain provider to Cloudflare’s nameservers.
    Request SSL Certificate from AWS ACM:
        Domain name: *.yourdomain.com
        ACM provides a CNAME record → add it in Cloudflare DNS (DNS Only).
    Once validated, certificate will be issued.
    Create the A name and CNAME record for the application
________________________________________
🔁 12. Update Frontend for HTTPS Domain
    Update .env and src/url.js in frontend to:
        REACT_BACKEND_URI=https://www.yourdomain.com
    Rebuild and redeploy:
        sudo rm -rf build /var/www/travel-frontend
        sudo npm run build
        sudo cp -r build /var/www/travel-frontend
________________________________________
✅ Final Step: Access the App
    Go to:
        https://www.yourdomain.com
 
________________________________________
📝 Notes
    Use pm2 save and pm2 startup to ensure PM2 apps run after reboot.
    Use HTTPS to avoid Mixed Content issues (HTTP backend call on HTTPS frontend).
    All frontend builds must be redone when .env or src/url.js changes.

