**Jenkins Install:** 
```
https://www.jenkins.io/doc/book/installing/linux/
```
**Trivy Install:** 
```
sudo apt-get install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update -y
sudo apt-get install trivy -y
```

**Run sonarqube**
```
docker run -itd --name SonarQube-Server -p 9000:9000 sonarqube:lts-community
```

**Nginx server setup:**
```
server {
    server_name 72.60.111.17.nip.io;  //change to your domain

    # Serve React frontend
    location / {
        proxy_pass http://127.0.0.1:30080/;   # frontend container
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    # Auth routes
    location /auth/ {
        proxy_pass http://127.0.0.1:30800;   # backend container
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    # API routes
    location /api/ {
        proxy_pass http://127.0.0.1:30800;   # backend container
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    # Optional: WebSocket support for backend (e.g., STOMP)
    location /ws/ {
        proxy_pass http://127.0.0.1:30800;   # backend container
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

**Install cert-bot: (for certificate)**
```
sudo apt update
sudo apt install certbot python3-certbot-nginx -y


sudo ln -s /etc/nginx/sites-available/chattingo.conf /etc/nginx/sites-enabled/  
sudo nginx -t
sudo systemctl restart nginx

# Issue Certificate
sudo certbot --nginx -d <example.com> # your domain

# Verify SSL
https://example.com     # your domain
```

**nip.io:**
```
https://nip.io/
```

**Install Node on server**
```
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

```

**Install docker-compose on server**
```
sudo apt install docker-compose-v2
```


**Commands for debugging**
```
sudo usermod -aG docker $USER
newgrp docker
docker ps
```
