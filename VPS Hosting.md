`ssh -i "adzum.pem" ubuntu@13.232.247.215`

Opsis

`ssh root@143.110.182.98`

7!OpsisEye

Adzum

`ssh root@64.227.180.96`

`7!AdzumG`


# After logging in to server

'sudo apt update
sudo apt install python3-venv python3-dev libpq-dev postgresql postgresql-contrib nginx curl'

# Configure Database Postgresql

`sudo -u postgres psql`
`CREATE DATABASE opsis;
CREATE USER opsis WITH PASSWORD 'opsis';`

`ALTER ROLE opsis SET client_encoding TO 'utf8';
ALTER ROLE opsis SET default_transaction_isolation TO 'read committed';
ALTER ROLE opsis SET timezone TO 'UTC';`

`GRANT ALL PRIVILEGES ON DATABASE opsis TO opsis;`

`\q`

# Create directory and go to directory

`mkdir ~/opsis
cd ~/opsis`

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

# Create git and pull project from git

`git init
git remote add origin https://github.com/FinuAjas/adzum.git 
git pull origin main`

# Install dependencies from requiremnets file

`pip install -r requirements.txt`

`python manage.py makemigrations
python manage.py migrate`

`python manage.py createsuperuser`

`python manage.py collectstatic`

`sudo ufw allow 8000`

`python manage.py runserver 0.0.0.0:8000`

# open link http://server_domain_or_IP:8000 and make sure site is working properly


# Configuring gunicorn


`gunicorn --bind 0.0.0.0:8000 businessportal.wsgi`

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
          businessportal.wsgi:application

[Install]
WantedBy=multi-user.target

. . .

`sudo systemctl start gunicorn.socket
sudo systemctl enable gunicorn.socket`

`sudo systemctl status gunicorn.socket`

. . .

# Output

. . .

`file /run/gunicorn.sock`

. . .

# Output

. . .

`sudo journalctl -u gunicorn.socket`

`sudo systemctl status gunicorn`

. . .

# Output

. . .

`curl --unix-socket /run/gunicorn.sock localhost`

. . .

# Output

The index page will be shown as output

. . .

`sudo systemctl status gunicorn`

# Output

. . .

`sudo journalctl -u gunicorn`

`sudo systemctl daemon-reload
sudo systemctl restart gunicorn`

`sudo nano /etc/nginx/sites-available/opsis`

# Add this code to project file.

. . .

server {
    listen 80;
    server_name 143.110.182.98;

    location = /favicon.ico { access_log off; log_not_found off; }

    location / {
        include proxy_params;
        proxy_pass http://unix:/run/gunicorn.sock;
    }
}

. . .

`sudo ln -s /etc/nginx/sites-available/opsis /etc/nginx/sites-enabled`

`sudo nginx -t`
`sudo systemctl restart nginx`

`sudo ufw delete allow 8000
sudo ufw allow 'Nginx Full`

# Now the site will be available at ip address

# To Update changes in project

`sudo systemctl restart gunicorn
sudo systemctl daemon-reload
sudo systemctl restart gunicorn.socket gunicorn.service
sudo nginx -t && sudo systemctl restart nginx`
