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
