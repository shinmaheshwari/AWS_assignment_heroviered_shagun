
flowchart TB
    User[User<br/>(Web Browser - HTTPS)]

    Cloudflare[Cloudflare<br/>DNS + SSL]

    subgraph AWS["AWS Cloud"]
        subgraph VPC["VPC"]
            IGW[Internet Gateway]

            subgraph AZ1["Public Subnet (AZ-1)"]
                FE["EC2: TM-Master-Server<br/>Frontend<br/>React + Nginx"]
            end

            subgraph AZ2["Public Subnet (AZ-2)"]
                ALB["Application Load Balancer"]
                BE1["EC2: travelMemory-backend-01<br/>Node.js + PM2"]
                BE2["EC2: travelMemory-backend-02<br/>Node.js + PM2"]
            end
        end
    end

    MongoDB["MongoDB Atlas<br/>(Managed Database)"]

    User -->|HTTPS| Cloudflare
    Cloudflare -->|A Record<br/>shagunthetech.in| FE
    Cloudflare -->|CNAME<br/>api.shagunthetech.in| ALB
    IGW --> FE
    IGW --> ALB
    ALB --> BE1
    ALB --> BE2
    BE1 --> MongoDB
    BE2 --> MongoDB
