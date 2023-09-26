# Node.js Deployment

> Steps to deploy a Node.js app to AWS Lightsail using PM2, NGINX as a reverse proxy and an SSL from LetsEncrypt

## 1. Sign up for AWS
[AWS Lightsail](https://lightsail.aws.amazon.com/)

## 2. Create an instance and connect to the server

## 3. Install Node/NPM
```
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -

sudo apt install nodejs

node --version
```

## 4. Clone your project from Github
There are a few ways to get your files on to the server, I would suggest using Git
```
git clone yourproject.git
```

### 5. Install dependencies and test app
```
cd yourproject
npm install
npm start (or whatever your start command)
# stop app
ctrl+C
```
## 6. Setup PM2 process manager to keep your app running
```
sudo npm i pm2 -g
pm2 start app (or whatever your file name)

# Other pm2 commands
pm2 show app
pm2 status
pm2 restart app
pm2 stop app
pm2 logs (Show log stream)
pm2 flush (Clear logs)

# To make sure app starts when reboot
pm2 startup ubuntu
```

## 8. Install NGINX and configure
```
sudo apt install nginx

sudo nano /etc/nginx/sites-available/default
```
Add the following to the location part of the server block
```
    server_name yourdomain.com www.yourdomain.com;

    location / {
        proxy_pass http://localhost:5000; #whatever port your app runs on
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
```
```
# Check NGINX config
sudo nginx -t

# Restart NGINX
sudo service nginx restart
```

### You should now be able to visit your IP with no port (port 80) and see your app. Now let's add a domain

## 9. Add domain in Digital Ocean
In Digital Ocean, go to networking and add a domain

Add an A record for @ and for www to your droplet


## Register your domain/subdomain
I prefer Namecheap for domains. 
- Point your A record to your AWS Lightsail Instance Public IP
  e.g Type: A, Host: domain, Value: 127.0.0.1
It may take a bit to propogate

10. Add SSL with LetsEncrypt
```
sudo apt-get update
sudo apt-get install software-properties-common
sudo apt-add-repository ppa:certbot/certbot -y
sudo apt-get update -y
sudo apt-get install certbot -y
# sudo apt-get install python-certbot-nginx
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

# Only valid for 90 days, test the renewal process with
certbot renew --dry-run
```
Now visit https://yourdomain.com and you should see your Node app

## Issues
  - Running ``` sudo service nginx restart``` issue
    - Error: 
      ```
      Job for nginx.service failed because the control process exited with error code.
      See "systemctl status nginx.service" and "journalctl -xe" for details.
      ```
    - Fix: ```sudo /opt/bitnami/ctlscript.sh stop ```


## Reference
[AWS Lightsail Ngnix Doc](https://lightsail.aws.amazon.com/ls/docs/en_us/articles/amazon-lightsail-using-lets-encrypt-certificates-with-nginx)
