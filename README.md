# deploy-flask-gunicorn
Deploy Flask on VPS with Gunicorn

### Update the local package
```bash
sudo apt update
```

### Install Nginx
```bash
sudo apt install nginx
```

### Install Package Python
```bash
sudo apt install python3-pip python3-dev build-essential libssl-dev libffi-dev python3-setuptools
```

### Creating a Python Virtual Environment
```bash
sudo apt install python3-venv
```

### Make dir or Clone project from github

### Create Virtual Environment
```bash
python3 -m venv myprojectenv
```

### Active the env
```bash
source myprojectenv/bin/activate
```

## Setting Flask Application

### Install Package
```bash
pip install wheel
```

### Install requirements.txt
```bash
pip install -r requirements.txt
```

### Allow Access Port you are using
```bash
sudo ufw allow 5000
```

### Test Your Flask App
```bash
python app.py
```

### You will receive output like the following
```bash
Output
 * Serving Flask app 'myproject' (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on all addresses.
   WARNING: This is a development server. Do not use it in a production deployment.
 * Running on http://your_server_ip:5000/ (Press CTRL+C to quit)
```
### Visit Your Website
```bash
http://your_server_ip:5000
```

## Creating the WSGI Entry Point

### Create a new file using your preferred text editor and name it
```bash
nano ~/myproject/wsgi.py
```

### import the Flask instance from your application and then run it
```bash
from myproject import app

if __name__ == "__main__":
    app.run()
```

## Configuring Gunicorn
### Next, you can check that Gunicorn can serve the application correctly by passing it the name of your entry point.
```bash
gunicorn --bind 0.0.0.0:5000 wsgi:app
```

### You will receive output like the following:
```bash
Output
[2021-11-19 23:07:57 +0000] [8760] [INFO] Starting gunicorn 20.1.0
[2021-11-19 23:07:57 +0000] [8760] [INFO] Listening at: http://0.0.0.0:5000 (8760)
[2021-11-19 23:07:57 +0000] [8760] [INFO] Using worker: sync
[2021-11-19 23:07:57 +0000] [8763] [INFO] Booting worker with pid: 8763
[2021-11-19 23:08:11 +0000] [8760] [INFO] Handling signal: int
[2021-11-19 23:08:11 +0000] [8760] [INFO] Shutting down: Master
```
### Visit Your Website
```bash
http://your_server_ip:5000
```

### Your are done with your virtual environment
```bash
deactivate
```

### create .service 
```bash
sudo nano /etc/systemd/system/myproject.service
```

###  Add a description of your service here
```bash
[Unit]
Description=Gunicorn instance to serve myproject
After=network.target

[Service]
User=sammy
Group=www-data
WorkingDirectory=/home/sammy/myproject
Environment="PATH=/home/sammy/myproject/myprojectenv/bin"
ExecStart=/home/sammy/myproject/myprojectenv/bin/gunicorn --workers 3 --bind unix:myproject.sock -m 007 wsgi:app

[Install]
WantedBy=multi-user.target
```

### Now start the Gunicorn service you created
Step 1
```bash
sudo systemctl start myproject
```
Step 2
```bash
sudo systemctl enable myproject
```
Step 3
```bash
sudo systemctl status myproject
```

```bash
Output
● myproject.service - Gunicorn instance to serve myproject
   Loaded: loaded (/etc/systemd/system/myproject.service; enabled; vendor preset
   Active: active (running) since Fri 2021-11-19 23:08:44 UTC; 6s ago
 Main PID: 8770 (gunicorn)
    Tasks: 4 (limit: 1151)
   CGroup: /system.slice/myproject.service
       	├─9291 /home/sammy/myproject/myprojectenv/bin/python3.6 /home/sammy/myproject/myprojectenv/bin/gunicorn --workers 3 --bind unix:myproject.sock -m 007 wsgi:app
       	├─9309 /home/sammy/myproject/myprojectenv/bin/python3.6 /home/sammy/myproject/myprojectenv/bin/gunicorn --workers 3 --bind unix:myproject.sock -m 007 wsgi:app
       	├─9310 /home/sammy/myproject/myprojectenv/bin/python3.6 /home/sammy/myproject/myprojectenv/bin/gunicorn --workers 3 --bind unix:myproject.sock -m 007 wsgi:app
       	└─9311 /home/sammy/myproject/myprojectenv/bin/python3.6 /home/sammy/myproject/myprojectenv/bin/gunicorn --workers 3 --bind unix:myproject.sock -m 007 wsgi:app
…
```

## Configuring Nginx to Proxy Requests
```bash
sudo nano /etc/nginx/sites-available/myproject
```

```bash
server {
    listen 80;
    server_name your_domain www.your_domain;

    location / {
        include proxy_params;
        proxy_pass http://unix:/home/sammy/myproject/myproject.sock;
    }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/myproject /etc/nginx/sites-enabled
```

```bash
sudo nginx -t
```

```bash
sudo systemctl restart nginx
```
Delete Access Port
```bash
sudo ufw delete allow 5000
```

Then allow full access to the Nginx server:

```bash
sudo ufw allow 'Nginx Full'
```
### Visit Your Website
```bash
http://yourdomain
```

## Securing the Application

### First, install Certbot using snap
```bash
sudo snap install --classic certbot
```

```bash 
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

```bash
sudo certbot --nginx -d your_domain -d www.your_domain
```

```
sudo ufw delete allow 'Nginx HTTP'
```
### Visit Your Website
```bash
https://yourdomain
```







