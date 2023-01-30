# notes_nginx-gunicorn-flask
Some steps to set up python-flask-gunicorn-nginx on AWS. Mostly copied from [source1](https://realpython.com/django-nginx-gunicorn/) and [source2](https://pyliaorachel.github.io/blog/tech/system/2017/07/07/flask-app-with-gunicorn-on-nginx-server-upon-aws-ec2-linux.html).
Other pages - [Flask notes]()

## AWS EC2 setup

* Create EC2 instance with Ubuntu.
* Create Elastic IP, associate with EC2 instance. Otherwise IP will change after system restart.
* Open ports 80 and 443 in EC2 security group. 
* Create domain in Route 53. When using domains from other vendors, it may take 60 days before you can transfer new domains.
* In Route 53, "create record" to link domain and the EC2 instance.

SSH to EC2 instance.<br />
Initial setup:
```
$ sudo apt update
$ sudo apt upgrade
$ sudo apt-get install fish git tig python3-pip python3-flask nginx curl
$ pip install virtualenv
```

## Gunicorn and Flask
Create project
```
$ mkdir myproject
$ cd myproject
```
Create Flask file ~/myproject/app.py:
```
from flask import Flask
application = Flask(__name__)

@application.route("/")
def index():
    return "Hello World!"

if __name__ == "__main__":
    application.run(host='0.0.0.0', port='8080')
```
Create ~/myproject/wsgi.py:
```
from app import application

if __name__ == "__main__":
    application.run()
``` 

Run Gunicorn (you will need port 8080 open in EC2 security groups to test without Nginx):
```
# create virtualenv
$ virtualenv venv

# activate virtualenv
$ source ./venv/bin/activate
$ pip install gunicorn flask

$ gunicorn --bind 0.0.0.0:8080 wsgi
```

Test serverip:8080

## Nginx setup
Replace "eazyprojectapp" with your domain everywhere below.

```
$ sudo apt-get install -y 'nginx=1.18.*'
$ nginx -v 
$ sudo systemctl start nginx
$ sudo systemctl status nginx
```
Create /etc/nginx/sites-available/eazyprojectapp:
```
server_tokens               off;
access_log                  /var/log/nginx/eazyprojectapp.access.log;
error_log                   /var/log/nginx/eazyprojectapp.error.log;

# This configuration will be changed to redirect to HTTPS later
server {
  server_name               .eazyprojectapp.com;
  listen                    80;
  location / {
    proxy_pass              http://localhost:8000;
    proxy_set_header        Host $host;
  }
}
```

Test config:
```
$ sudo service nginx configtest /etc/nginx/sites-available/eazyprojectapp
```
Enable domain:
```
$ cd /etc/nginx/sites-enabled
$ sudo ln -s ../sites-available/eazyprojectapp .
$ sudo systemctl restart nginx
```

Test new domain with HTTP.

## HTTPS
In /etc/nginx/nginx.conf, replace ssl_protocols with: 
```
ssl_protocols TLSv1.2 TLSv1.3;
```

You can use nginx -t to confirm that your Nginx supports version 1.3:
```
$ sudo nginx -t
```
Install and run Certbot
```
$ sudo snap install --classic certbot
$ sudo ln -s /snap/bin/certbot /usr/bin/certbot
$ sudo certbot --nginx --rsa-key-size 4096 --no-redirect
# (type both name.com and www.name.com)
```
/etc/nginx/sites-available/eazyprojectapp will be updated by Certbot

Restart Nginx
```
$ sudo systemctl reload nginx
```

Test https://eazyprojectapp.com. Redirect from HTTP to HTTPS may need to be configured separately.

## TBD
Missing: Upstart script for Gunicorn.

## Other commands

```
# ports in use:
$ sudo lsof -i -P -n | grep LISTEN

# stop gunicorn
$ pkill gunicorn
```

## Resources
https://realpython.com/django-nginx-gunicorn/

https://pyliaorachel.github.io/blog/tech/system/2017/07/07/flask-app-with-gunicorn-on-nginx-server-upon-aws-ec2-linux.html

![alt text](/images/nginx-gunicorn-flask.png)
