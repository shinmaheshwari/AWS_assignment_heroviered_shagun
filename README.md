🌍 Travel Memory MERN Application
Manual Deployment on AWS EC2 with Load Balancer and Cloudflare

📌 Project Overview
The Travel Memory Application is a full‑stack web application developed using the MERN stack (MongoDB, Express.js, React.js, Node.js). The application allows users to create, view, and manage their travel experiences by adding details such as destination, hotels, expenses, and trip memories.
This assignment focuses on the manual deployment of the application on Amazon EC2, implementing scalability using a Load Balancer, and connecting the application to a custom domain using Cloudflare, while following industry best practices.

📂 Project Repository
GitHub Repository:
👉 https://github.com/UnpredictablePrashant/TravelMemory

🎯 Objectives

Set up the backend running on Node.js
Configure the frontend designed with React
Ensure efficient communication between frontend and backend
Deploy the full application on an EC2 instance
Facilitate load balancing by creating multiple EC2 instances
Configure an Application Load Balancer (ALB)
Connect a custom domain through Cloudflare
Enable HTTPS access


🛠️ Technology Stack
Application

Frontend: React.js
Backend: Node.js, Express.js
Database: MongoDB Atlas

Infrastructure

Cloud Provider: AWS
Compute: EC2
Load Balancer: Application Load Balancer (ALB)
Web Server: Nginx
Process Manager: PM2
DNS & SSL: Cloudflare


🧱 Architecture Overview
Browser (HTTPS)
   |
Cloudflare (DNS + SSL)
   |
   ├── shagunthetech.in
   |      → TM‑Master‑Server
   |         (React Frontend + Nginx)
   |
   └── api.shagunthetech.in
          → Application Load Balancer
                 |
                 ├── travelMemory‑backend‑01
                 |      (Node.js + PM2)
                 |
                 └── travelMemory‑backend‑02
                        (Node.js + PM2)
                           |
                       MongoDB Atlas


✅ Prerequisites
Required Accounts

AWS Account
MongoDB Atlas Account
Cloudflare Account
GitHub Account

Software Requirements

Node.js (v18+)
npm
Git
Nginx
PM2


🚀 Deployment Steps

1️⃣ Backend Setup (Node.js)
Clone the repository and install dependencies:
Shellgit clone https://github.com/UnpredictablePrashant/TravelMemory.gitcd TravelMemory/backendnpm installShow more lines
Create .env file:
Plain Textenv isn’t fully supported. Syntax highlighting is based on Plain Text.PORT=3000MONGO_URI=mongodb+srv://<username>:<password>@cluster.mongodb.net/travelmemoryJWT_SECRET=your_secret_keyShow more lines
Add a health‑check endpoint in index.js:
JavaScriptapp.get("/api/health", (req, res) => {  res.status(200).json({ status: "ok" });});Show more lines
Run backend using PM2:
Shellnpm install -g pm2pm2 start index.js --name travelmemory-backendpm2 startuppm2 saveShow more lines

2️⃣ Frontend Setup (React)
Install dependencies:
Shellcd ../frontendnpm installShow more lines
Configure backend URL in src/url.js:
JavaScriptconst baseUrl = "https://api.shagunthetech.in";export default baseUrl;Show more lines
Build frontend:
Shellnpm run buildShow more lines

3️⃣ EC2 and Nginx Setup
Install required packages on TM‑Master‑Server:
Shellsudo apt update -ysudo apt install -y nginx nodejs npm gitShow more lines
Deploy frontend build files:
Shellsudo rm -rf /var/www/html/*sudo cp -r build/* /var/www/html/Show more lines
Configure Nginx:
Nginx Configserver {  listen 80;  server_name shagunthetech.in www.shagunthetech.in;  location / {    root /var/www/html;    index index.html;    try_files $uri /index.html;  }}Show more lines
Restart Nginx:
Shellsudo nginx -tsudo systemctl restart nginxShow more lines

4️⃣ Scaling the Backend Using Load Balancer

Three EC2 instances were used:

TM‑Master‑Server: Frontend (React + Nginx)
travelMemory‑backend‑01: Backend (Node.js + PM2)
travelMemory‑backend‑02: Backend (Node.js + PM2)


An AMI was created from a configured backend server
Two backend EC2 instances were launched using the AMI
A Target Group was created on port 3000
Health check configured at /api/health
Backend EC2 instances registered with the target group
An Application Load Balancer was created to distribute traffic across backend instances


5️⃣ Domain & SSL Configuration (Cloudflare)
Cloudflare DNS setup:




















Record TypeNamePoints ToAshagunthetech.inTM‑Master‑Server Public IPCNAMEapiALB DNS Name
✅ Proxy enabled (orange cloud)
SSL settings:

SSL/TLS Mode: Flexible


✅ Testing & Verification

Frontend accessible at:
👉 https://shagunthetech.in
Backend health check verified at:
👉 https://api.shagunthetech.in/api/health
User authentication tested successfully
“Add Experience” form submission verified
Data stored correctly in MongoDB Atlas
Load balancing confirmed across travelMemory‑backend‑01 and travelMemory‑backend‑02
HTTPS enabled via Cloudflare


🧪 Challenges Faced


Backend service not persisting after SSH logout
The backend stopped running after closing the SSH session. This was resolved by configuring PM2 as a process manager and enabling it to start on system boot.


Target Group instances marked as unhealthy
Backend EC2 instances were initially unhealthy due to a port mismatch and incorrect environment configuration. Explicitly setting PORT=3000 and correcting the health‑check path resolved the issue.


PM2 process not running on AMI‑based EC2 instances
AMIs do not preserve running processes. Backend services had to be manually started on travelMemory‑backend‑01 and travelMemory‑backend‑02, and saved using PM2.


Frontend not communicating with backend APIs
The frontend initially used an incorrect backend URL. Updating url.js to use the API subdomain resolved the issue.


Cloudflare HTTPS requests failing
HTTPS requests failed due to an SSL configuration mismatch. Switching Cloudflare SSL mode to Flexible enabled proper SSL termination.


Incorrect routing between frontend and backend domains
API traffic was mistakenly routed to the frontend server. Using an A record for the frontend and a CNAME record for the backend ALB, as suggested in the assignment, fixed the routing.


Difficulty validating load balancing behavior
Load balancing was verified by monitoring PM2 logs on multiple backend instances and observing distributed request handling.


Silent failures during form submission
The “Add Experience” form initially failed due to missing authentication tokens. Logging in correctly and verifying network requests resolved the issue.



✅ Deliverables Status

✅ Fully functional MERN application
✅ Backend running on multiple EC2 instances
✅ Application Load Balancer configured
✅ Custom domain configured via Cloudflare
✅ HTTPS enabled
✅ Architecture diagram created using draw.io
✅ Manual deployment fully documented


🏁 Conclusion
The Travel Memory application was successfully deployed using a manual cloud deployment approach on AWS EC2. The final architecture provides scalability through load balancing, secure access via Cloudflare SSL, and reliable data storage using MongoDB Atlas. All objectives and evaluation criteria of the assignment have been fulfilled.
