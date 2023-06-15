`ssh -i "dysgraphia.pem" ubuntu@15.206.124.111`


Opsis

`ssh root@143.110.182.98`

`7!OpsisEye`

Adzum

`ssh root@64.227.180.96`

`7!AdzumG`

`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`

`ssh -i "employee_management.pem" ubuntu@3.109.185.82`


# After logging in to server

'sudo apt update
sudo apt install python3-venv python3-dev libpq-dev postgresql postgresql-contrib nginx curl'

# Configure Database Postgresql

`sudo -u postgres psql`

`CREATE DATABASE dysgraphia;
CREATE USER dysgraphia WITH PASSWORD 'dysgraphia';`

`ALTER ROLE dysgraphia SET client_encoding TO 'utf8';
ALTER ROLE dysgraphia SET default_transaction_isolation TO 'read committed';
ALTER ROLE dysgraphia SET timezone TO 'UTC';`

`GRANT ALL PRIVILEGES ON DATABASE dysgraphia TO dysgraphia;`

`\q`

# Create directory and go to directory

`mkdir ~/dysgraphia
cd ~/dysgraphia`

# Create git and pull project from git

`git init
git remote add origin https://github.com/FinuAjas/dysgraphia.git 
git pull origin main`

# Create virtual environment and activate

'python3 -m venv venv
source venv/bin/activate'

# Install dependencies

`pip install django gunicorn psycopg2-binary`

. . .

Make sure you have following configuration in database settings of the project

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'projectname',
        'USER': 'username',
        'PASSWORD': 'password',
        'HOST': 'localhost',
        'PORT': '',
    }
}

. . .

# Install dependencies from requiremnets file

`pip install -r requirements.txt`

`python manage.py makemigrations
python manage.py migrate`

`python manage.py createsuperuser`

`python manage.py collectstatic`

`sudo ufw allow 8000
python manage.py runserver 0.0.0.0:8000`

# open link http://server_domain_or_IP:8000 and make sure site is working properly


# Configuring gunicorn


`gunicorn --bind 0.0.0.0:8000 dysgraphia_check_project.wsgi`

# Deactive virtualenv 

`deactivate`


`sudo nano /etc/systemd/system/gunicorn.socket`

# Add this code to gunicorn.socket file.

. . .

[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/run/gunicorn.sock

[Install]
WantedBy=sockets.target

. . .


`sudo nano /etc/systemd/system/gunicorn.service`

# Add this code to gunicorn.service file.
. . .

# AWS 

[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/home/ubuntu/dysgraphia
ExecStart=/home/ubuntu/dysgraphia/venv/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/run/gunicorn.sock \
          dysgraphia_check_project.wsgi:application

[Install]
WantedBy=multi-user.target

# Digital Ocean 

[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=root
Group=www-data
WorkingDirectory=/root/opsis
ExecStart=/root/opsis/venv/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/run/gunicorn.sock \
          opsis.wsgi:application

[Install]
WantedBy=multi-user.target

. . .

`sudo systemctl start gunicorn.socket
sudo systemctl enable gunicorn.socket
sudo systemctl status gunicorn.socket`

. . .

# Output

. . .

`file /run/gunicorn.sock`

. . .

# Output

. . .

`sudo journalctl -u gunicorn.socket
sudo systemctl status gunicorn`

. . .

# Output

. . .

`curl --unix-socket /run/gunicorn.sock localhost`

. . .

# Output

The index page will be shown as output

. . .


`sudo journalctl -u gunicorn
sudo systemctl daemon-reload
sudo systemctl restart gunicorn`

`sudo nano /etc/nginx/sites-available/dysgraphia`

# Add this code to project file.

. . .

server {
    listen 80;
    server_name  15.206.124.111;

    location = /favicon.ico { access_log off; log_not_found off; }

    location / {
        include proxy_params;
        proxy_pass http://unix:/run/gunicorn.sock;
    }
}

. . .

`sudo ln -s /etc/nginx/sites-available/dysgraphia /etc/nginx/sites-enabled`

`sudo nginx -t
sudo systemctl restart nginx`

`sudo ufw delete allow 8000
sudo ufw allow 'Nginx Full`

# Now the site will be available at ip address

# To Update changes in project

`sudo systemctl restart gunicorn
sudo systemctl daemon-reload
sudo systemctl restart gunicorn.socket gunicorn.service
sudo nginx -t && sudo systemctl restart nginx`

# Delete non empty directory

`sudo rm -r dysgraphia/`
