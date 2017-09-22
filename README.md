# Django Trump
<img src="djtrump/static/djtrump/app.png">

## Description
A web app that shows Donald Trump and a random quote of his. Quotes can be personalized. The app makes use of [whatdoestrumpthink](https://whatdoestrumpthink.com/) API.

It is a sample django app intended for learning purposes only:

[How to deploy a Django app to DigitalOcean](http://rahmonov.me/posts/deploy-a-django-app-to-digitalocean/)

# CI and CD
## installing system-wide dependencies
``` 
apt-get update
apt-get -y upgrade

apt-get install -y nginx
apt-get install -y supervisor
apt-get install -y python-virtualenv
apt-get install -y postgresql postgresql-contrib
```

## Configure Database
```
sudo su - postgres
psql
postgres=# CREATE DATABASE djtrumpprod;
postgres=# CREATE USER djtrumpuser WITH password 'djtrump';
postgres=# GRANT ALL PRIVILEGES ON DATABASE djtrumpprod TO djtrumpuser;
postgres=# \q
postgres@djtrump:~$ exit
```

## setting up our project and its environment
```
git clone https://github.com/rahmonov/djtrump.git

virtualenv djtrumpenv --python=$(which python3.5)
source djtrumpenv/bin/active

cd djtrump
pip install -r requirements.txt
```
Now we should migrate but there is one more thing that we need to do before that. If you go to the settings folder, there are two files: base.py and prod.py. Basically, base.py contains all the configurations and prod.py overrides those needed in the production environment. For example, DATABASES config is overridden in prod.py. That's why, we need to tell our environment to use this prod.py and not the default base.py. This is done by setting DJANGO_SETTINGS_MODULE env variable to prod.py path. Open ~/.bash_profile and add this:

```
export DJANGO_SETTINGS_MODULE=djtrump.settings.prod
```
Save and quit. Then, source this file for our changes to take effect:
```
source ~/.bash_profile
```
Now, try to migrate. Most probably, it will fail and say something like this:
```
FATAL:  Peer authentication failed for user "djtrumpuser"
```
That's because, postgresl uses peer authentication by default, which is it will succeed if the user with the same name as the postgres user uses it. In our case, there is no djtrumpuser user in postgres and thus it fails. To fix it, go to /etc/postgresql/9.5/main/pg_hba.conf and change the line that says this:
```
local   all     all      peer
```
to this:
```
local   all     all      md5
```
Save and quit. This way, postgres will try to use password to authenticate the user. Now, restart postgresql for our changes to take effect:
```
sudo service postgresl restart
```
Go ahead and migrate:
```
python manage.py migrate
```
It works now. Cool! Try to run the development server and it will work.
# configuring nginx
Create a new file: /etc/nginx/sites-available/djtrump and add the following:
```
server {
    listen 80;
    server_name your_ip;

    location = /favicon.ico { access_log off; log_not_found off; }

    location /static/ {
            alias /root/djtrump/static/;
    }

    location / {
            include proxy_params;
            proxy_pass http://your_ip:8030;
    }
}
```
Replace your_ip with the IP address of your server. We know what this is doing from the previous tutorials. Basically, it is serving the static files from /root/djtrump/static/ and redirecting http requests to gunicorn which should be running on port 8030.

Now, let's enable this file by linking it to the sites-enabled folder:
```
sudo ln -s /etc/nginx/sites-available/djtrump /etc/nginx/sites-enabled
```
Restart nginx:
```
sudo service nginx restart
```
There are two more things that we need to do before nginx works. First, we need to put all our static files in the folder /root/djtrump/static/ and run gunicorn on port 8030 as we promised in nginx config file.

First, run this to gather all static files in that folder:

```
python manage.py collectstatic --noinput
```
Now, run gunicorn:

```
gunicorn --workers 3 --bind 0.0.0.0:8030 djtrump.wsgi
```

Go ahead and type in the browser the IP of your address. You will see that the app is running. Congratulations!

Please note that if you cloned the app to the user's home directory, you may face issues with static files (Permission denied error). One of the ways to solve it to run nginx as root. To do that, open /etc/nginx/nginx.conf and change the line that says:
```
user www-data;
```
to this:
```
user root;
```
and restart the nginx:
```
sudo service nginx restart
```
# configuring supervisor
