---
layout: post
title: Running flask with virtualenv, uwsgi, and nginx
date: 2013-10-17 22:40:57.000000000 -07:00
---
My website used to be hosted on Heroku, but I recently changed to a virtual private server. Figuring out how to serve my flask site with virtualenv, uwsgi, and nginx was frustrating and it almost made me regret switching. There are plenty of articles on this same topic, but none of them worked for me on the first try. Here are my notes for Ubuntu 12.04 in case someone else is experiencing similar issues.

First make sure that your vps has the latest updates:

    sudo apt-get update
    sudo apt-get upgrade

Now install python and virtualenv:

    sudo apt-get install build-essential python-dev python-pip
    sudo pip install virtualenv

Make a folder for your website:

    sudo mkdir -p /var/www/mysite
    sudo chown -R <your user id> /var/www/mysite
    cd /var/www/mysite

Setup virtualenv and install flask:

    virtualenv .env --no-site-packages
    source .env/bin/activate
    pip install flask

Place your flask app in this folder. Make sure that your host is set to `0.0.0.0` and that your app is under `if __name__ == '__main__':`. If your app is in a function, uwsgi will not be able to call it.

Now is a good time to test your app with the flask development server to see if everything is working so far. If everything runs smoothly, install nginx and uwsgi:

    deactivate
    sudo apt-get install nginx uwsgi uwsgi-plugin-python

Next we must create a socket file for nginx to communicate with uwsgi:

    cd /tmp/
    touch mysite.sock
    sudo chown www-data mysite.sock

By changing the owner of `mysite.sock` to `www-data`, nginx will be able to write to the socket. Now all we have to do is add our configuration files for nginx and uwsgi. First delete the default configuration for nginx:

    cd /etc/nginx/sites-available
    sudo rm default

Create a new configuration file `mysite` and add the following:

    server {
        listen 80;
        server_tokens off;
        server_name www.mysite.com mysite.com;

         location / {
             include uwsgi_params;
             uwsgi_pass unix:/tmp/mysite.sock;
         }

         location /static {
             alias /var/www/mysite/static;
         }

         ## Only requests to our Host are allowed
         if ($host !~ ^(mysite.com|www.mysite.com)$ ) {
            return 444;
         }
    }

In order to enable the site, we must link our configuration file to `/etc/nginx/sites-enabled/`:

    sudo ln -s /etc/nginx/sites-available/mysite /etc/nginx/sites-enabled/mysite

The process is similar for uwsgi. Create the file `/etc/uwsgi/apps-available/mysite.ini` and add the following:

    [uwsgi]
    vhost = true
    socket = /tmp/mysite.sock
    venv = /var/www/mysite/.env
    chdir = /var/www/mysite
    module = app
    callable = app

Module is the name of your python script and callable is the name of your flask instance. So if your flask site was in a file called `mysite.py` that looked like this:

    from flask import Flask
    my_app = Flask(__name__)

    @my_app.route('/')
    def hello_world():
        return 'Hello World!'

    if __name__ == '__main__':
        my_app.run(host='0.0.0.0')

Your mysite.ini file would be:

    module = mysite
    callable = my_app

Link the configuration file to the enabled-apps folder:

    sudo ln -s /etc/uwsgi/apps-available/mysite.ini /etc/uwsgi/apps-enabled/mysite.ini

Finally, restart nginx and uwsgi:

    sudo service nginx restart
    sudo service uwsgi restart

Thats it. If you notice any errors in my guide, please feel free to email me. Here are some tips in case you get stuck:

* Check that your flask site runs under virtualenv without any errors.
* Ensure that you can run your site with just uwsgi from the command line.
* If you run uwsgi with sudo you will change the owner of `mysite.sock` to root and this will create errors for nginx. Make sure that you change the owner back to `www-data`.
* If uwsgi cannot find your app, you probably have an issue with file permissions. In order to serve the site uwsgi must have executable permissions for your python script and your .env folder.
* The logs for nginx and uwsgi are `/var/log/nginx/error.log` and `/var/log/uwsgi/app/mysite.log` respectively. If nginx is working properly, you will want to look at `/var/log/nginx/access.log`.
