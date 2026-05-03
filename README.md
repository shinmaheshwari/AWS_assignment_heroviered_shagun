# 🌍 Travel Memory MERN Application
## Manual Deployment on AWS EC2 with Load Balancer and Cloudflare

---

## 📌 Project Overview

The Travel Memory Application is a full‑stack web application developed using the MERN stack (MongoDB, Express.js, React.js, Node.js). The application allows users to create, view, and manage their travel experiences by adding details such as destination, hotels, expenses, and trip memories.

This assignment focuses on the manual deployment of the application on Amazon EC2, implementing scalability using a Load Balancer, and connecting the application to a custom domain using Cloudflare, while following industry best practices.

---

## 📂 Project Repository

**GitHub Repository:**
- 👉 [https://github.com/UnpredictablePrashant/TravelMemory](https://github.com/UnpredictablePrashant/TravelMemory)

---

## 🎯 Objectives

- ✅ Set up the backend running on Node.js
- ✅ Configure the frontend designed with React
- ✅ Ensure efficient communication between frontend and backend
- ✅ Deploy the full application on an EC2 instance
- ✅ Facilitate load balancing by creating multiple EC2 instances
- ✅ Configure an Application Load Balancer (ALB)
- ✅ Connect a custom domain through Cloudflare
- ✅ Enable HTTPS access

---

## 🛠️ Technology Stack

### Application
- **Frontend:** React.js
- **Backend:** Node.js, Express.js
- **Database:** MongoDB Atlas

### Infrastructure
- **Cloud Provider:** AWS
- **Compute:** EC2
- **Load Balancer:** Application Load Balancer (ALB)
- **Web Server:** Nginx
- **Process Manager:** PM2
- **DNS & SSL:** Cloudflare

---

## 🧱 Architecture Overview

```
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
```

---

## ✅ Prerequisites

### Required Accounts
- AWS Account
- MongoDB Atlas Account
- Cloudflare Account
- GitHub Account

### Software Requirements
- Node.js (v18+)
- npm
- Git
- Nginx
- PM2

---

## 🚀 Deployment Steps

### 1️⃣ Backend Setup (Node.js)

**Clone the repository and install dependencies:**
```bash
git clone https://github.com/UnpredictablePrashant/TravelMemory.git
cd TravelMemory/backend
npm install
```

**Create `.env` file:**
```env
PORT=3000
MONGO_URI=mongodb+srv://<username>:<password>@cluster.mongodb.net/travelmemory
JWT_SECRET=your_secret_key
```

**Add a health‑check endpoint in `index.js`:**
```javascript
app.get("/api/health", (req, res) => {
  res.status(200).json({ status: "ok" });
});
```

**Run backend using PM2:**
```bash
npm install -g pm2
pm2 start index.js --name travelmemory-backend
pm2 startup
pm2 save
```

---

### 2️⃣ Frontend Setup (React)

**Install dependencies:**
```bash
cd ../frontend
npm install
```

**Configure backend URL in `src/url.js`:**
```javascript
const baseUrl = "https://api.shagunthetech.in";
export default baseUrl;
```

**Build frontend:**
```bash
npm run build
```

---

### 3️⃣ EC2 and Nginx Setup

**Install required packages on TM‑Master‑Server:**
```bash
sudo apt update -y
sudo apt install -y nginx nodejs npm git
```

**Deploy frontend build files:**
```bash
sudo rm -rf /var/www/html/*
sudo cp -r build/* /var/www/html/
```

**Configure Nginx:**
```nginx
server {
  listen 80;
  server_name shagunthetech.in www.shagunthetech.in;
  
  location / {
    root /var/www/html;
    index index.html;
    try_files $uri /index.html;
  }
}
```

**Restart Nginx:**
```bash
sudo nginx -t
sudo systemctl restart nginx
```

---

### 4️⃣ Scaling the Backend Using Load Balancer

Three EC2 instances were used:

| Instance Name | Purpose |
|---|---|
| **TM‑Master‑Server** | Frontend (React + Nginx) |
| **travelMemory‑backend‑01** | Backend (Node.js + PM2) |
| **travelMemory‑backend‑02** | Backend (Node.js + PM2) |

**Steps:**
1. An AMI was created from a configured backend server
2. Two backend EC2 instances were launched using the AMI
3. A Target Group was created on port 3000
4. Health check configured at `/api/health`
5. Backend EC2 instances registered with the target group
6. An Application Load Balancer was created to distribute traffic across backend instances

---

### 5️⃣ Domain & SSL Configuration (Cloudflare)

**Cloudflare DNS Setup:**

| Record Type | Name | Points To | Status |
|---|---|---|---|
| A | shagunthetech.in | TM‑Master‑Server Public IP | ✅ Proxied |
| CNAME | api | ALB DNS Name | ✅ Proxied |

**SSL Settings:**
- **SSL/TLS Mode:** Flexible

---

## ✅ Testing & Verification

- ✅ Frontend accessible at: **https://shagunthetech.in**
- ✅ Backend health check verified at: **https://api.shagunthetech.in/api/health**
- ✅ User authentication tested successfully
- ✅ "Add Experience" form submission verified
- ✅ Data stored correctly in MongoDB Atlas
- ✅ Load balancing confirmed across travelMemory‑backend‑01 and travelMemory‑backend‑02
- ✅ HTTPS enabled via Cloudflare

---

## 🧪 Challenges Faced

### 1. Backend service not persisting after SSH logout
**Problem:** The backend stopped running after closing the SSH session.
**Solution:** Configured PM2 as a process manager and enabled it to start on system boot.

### 2. Target Group instances marked as unhealthy
**Problem:** Backend EC2 instances were initially unhealthy due to a port mismatch and incorrect environment configuration.
**Solution:** Explicitly set `PORT=3000` and corrected the health‑check path.

### 3. PM2 process not running on AMI‑based EC2 instances
**Problem:** AMIs do not preserve running processes.
**Solution:** Backend services were manually started on travelMemory‑backend‑01 and travelMemory‑backend‑02, and saved using PM2.

### 4. Frontend not communicating with backend APIs
**Problem:** The frontend initially used an incorrect backend URL.
**Solution:** Updated `url.js` to use the API subdomain.

### 5. Cloudflare HTTPS requests failing
**Problem:** HTTPS requests failed due to an SSL configuration mismatch.
**Solution:** Switched Cloudflare SSL mode to Flexible for proper SSL termination.

### 6. Incorrect routing between frontend and backend domains
**Problem:** API traffic was mistakenly routed to the frontend server.
**Solution:** Used an A record for the frontend and a CNAME record for the backend ALB.

### 7. Difficulty validating load balancing behavior
**Problem:** Load balancing was challenging to verify.
**Solution:** Monitored PM2 logs on multiple backend instances and observed distributed request handling.

### 8. Silent failures during form submission
**Problem:** The "Add Experience" form initially failed due to missing authentication tokens.
**Solution:** Logged in correctly and verified network requests.

---

## ✅ Deliverables Status

- ✅ Fully functional MERN application
- ✅ Backend running on multiple EC2 instances
- ✅ Application Load Balancer configured
- ✅ Custom domain configured via Cloudflare
- ✅ HTTPS enabled
- ✅ Architecture diagram created using draw.io
- ✅ Manual deployment fully documented

---

## 🏁 Conclusion

The Travel Memory application was successfully deployed using a manual cloud deployment approach on AWS EC2. The final architecture provides scalability through load balancing, secure access via Cloudflare SSL, and reliable data storage using MongoDB Atlas. All objectives and evaluation criteria of the assignment have been fulfilled.

---

## 📞 Support & Contact

For more information about this project, please visit the [original repository](https://github.com/UnpredictablePrashant/TravelMemory).

---

**Last Updated:** May 3, 2026
