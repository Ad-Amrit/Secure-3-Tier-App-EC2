# Secure-3-Tier-App-EC2

This guide outlines deploying a secure web application on an Amazon EC2 instance using Nginx as a web server.

### Step 1: Create EC2 Instance and connect it to the Linux terminal
![Screenshot (263)](https://github.com/user-attachments/assets/f310549b-0aeb-4a7f-acd8-1cb7dfd35605)

### Step 2: Install Nginx, Node.js, and NVM

- Run following commands to install, enable and start Nginx.

```jsx
sudo apt update
sudo apt install nginx
sudo systemctl enable nginx
sudo systemctl start nginx
```
![Screenshot (265)](https://github.com/user-attachments/assets/af0afdf1-b8a2-406c-9358-bfffae5174da)

- Install NVM (Node Version Manager)

```jsx
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash
```

- Load NVM into your shell

```jsx
source ~/.nvm/nvm.sh
```

- Install a desired Node.js version

```jsx
nvm install 22
```

### Step 3: Clone Git Repository, Set Up Project, and Build for Deployment

**Clone Git Repository**

- Navigate to the following directory

```jsx
cd /var/www/html/
```

- Clone the git repository

```jsx
git clone https://github.com/your-username/your-repository.git
Example: sudo git clone https://github.com/MonkeyDmagnas/Demo-wanderlust.git
```

**Note:** If you're using a private repository, you'll need to provide your GitHub credentials when prompted.

**Install dependencies**

- Navigate to the frontend directory

```jsx
cd /var/www/html/frontend
```

- Install project dependencies

```jsx
npm install
```

- Build the frontend application

```jsx
npm run build
```

This command will create an optimized version of your frontend code in a `dist` or `build` folder.

**Install MongoDB**

- Execute following commands to install MongoDB

```jsx
sudo apt-get install gnupg curl
curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg \
   --dearmor
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
sudo apt-get update
sudo apt-get install -y mongodb-org
```

- Enable and Start MongoDB

```jsx
sudo systemctl enable mongod
sudo systemctl start mongod
```

- Verify MongoDB Status

```jsx
sudo systemctl status mongod
```
![Screenshot (266)](https://github.com/user-attachments/assets/ae885baa-7c46-4b93-9476-62dda9c668df)

**Import MongoDB Data**

- Navigate to backend directory

```jsx
cd /var/www/html/Demo-wanderlust/backend
mongoimport --db wanderlust --collection posts --file ./data/sample_posts.json --jsonArray
```

**Start Frontend and Backend with PM2**

- Install PM2

```jsx
npm install -g pm2
```

- Start the frontend application

```jsx
cd /var/www/html/Demo-wanderlust/frontend
pm2 start npm --name wanderlust-frontend -- run dev -- --host
```

- Start the backend application

```jsx
cd /var/www/html/Demo-wanderlust/backend
pm2 start npm --name wanderlust-backend -- start -- --host
```

### Step 4: Configure Nginx

- Navigate to the following directory

```jsx
cd /etc/nginx/sites-available/
```

- Create a Nginx configuration file . (You can edit existing default file)

```jsx
vi wanderlust.com
```

In my case, I delete default file from both sites-available and sites-enable directory and create new nginx configuration file inside sites-available.

- Add following configuration inside [wanderlust.com](http://wanderlust.com)

```jsx
server {
  listen 80;
  listen [::]:80;

  listen 443 ssl;
  server_name ec2-34-228-113-105.compute-1.amazonaws.com;

  ssl_certificate /var/www/html/Demo-wanderlust/frontend/self-signed.crt;
  ssl_certificate_key /var/www/html/Demo-wanderlust/frontend/private.key;

  location / {
    proxy_pass http://localhost:5173;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_cache_bypass $http_upgrade;
  }

  location /api {
    proxy_pass http://localhost:5000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_cache_bypass $http_upgrade;
  }

  # Proxy requests to the MongoDB database
  location /mongodb {
    proxy_pass http://localhost:27017;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
  }
}
```

- Link the Configuration

```jsx
sudo ln -s /etc/nginx/sites-available/wanderlust.com /etc/nginx/sites-enabled/wanderlust.com
```

- Reload Nginx

```jsx
sudo nginx -t
sudo systemctl reload nginx
```

### Step 5: Configure Nginx for HTTPS

root@ip-172-31-82-198:~# mkdir openssl
root@ip-172-31-82-198:~# cd openssl/
root@ip-172-31-82-198:~/openssl# openssl req \
-newkey rsa:2048 -nodes -keyout private.key \
-out wanderlust.csr
![Screenshot (271)](https://github.com/user-attachments/assets/5f49a233-d85d-4274-9a8c-64bab41adeb6)
![Screenshot (266)](https://github.com/user-attachments/assets/59f0e14e-9cc4-4a66-b6fc-8ef8fc51476a)
![Screenshot (270)](https://github.com/user-attachments/assets/e932a644-cbc5-4f88-a370-e6bcb0084adc)

---
