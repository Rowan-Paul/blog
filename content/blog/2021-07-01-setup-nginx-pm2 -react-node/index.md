---
title: How to setup nginx and pm2 for React & node
description: Using PM2 & nginx to get your website out in the world.
date: "2021-07-01"
---

*This tutorial assumes you already have a working Ubuntu VPS with the repositories downloaded to it, specifically in the var/www folder. Also that you have your server setup to serve your React app, have setup your mongodb. I might make another tutorial for getting your repository on your VPS with SSH and GitHub Actions.*

**With this tutorial you will learn how to deploy a React app which is being served by a node.js backend server connected to a mongodb database.**

First we're going to install node, npm, nginx and nano. The second ``npm -v`` command lets you check the npm version (and checking if it installed at all).
```
cd ~
curl -sL https://deb.nodesource.com/setup_15.x -o nodesource_setup.sh
sudo bash nodesource_setup.sh
sudo apt install nodejs
nodejs -v
```
```
sudo apt install npm
npm -v
sudo apt-get install nginx
sudo apt install nano
```

After installing these programs you can now install PM2, a process manager that will keep our server running 24/7.
```
npm install -g pm2
```

Now we setup nginx, which will point your server to the server PM2 runs. Replace domain(.com) with your domain and pay attention to the port that you host your server on. We then simply do a validation check on the file and restart nginx to let the changes take effect.
```
cd /etc/nginx/sites-available
nano default

# copy paste this into default
map $http_upgrade $connection_upgrade {
    default         upgrade;
    ''              close;
}
server {

       server_name domain.com www.domain.com;

       location / {
        # Backend nodejs server
        proxy_pass         http://127.0.0.1:3000;
        proxy_http_version  1.1;
        proxy_set_header    Upgrade     $http_upgrade;
        proxy_set_header    Connection  $connection_upgrade;
    }
}

# test if the configuration is correct
nginx -t

# restart nginx
systemctl restart nginx
```

Then we can install the HTTPS certificates so that your site gets that (green) lock icon (you should do this even if you don't have any user input).
```
sudo apt install snapd
sudo snap install --classic certbot
cd /
certbot --nginx -d domain.com -d www.domain.com
systemctl restart nginx
```

Finally, you can start the PM2 by navigating to your server folder and starting the server which has a app.js file.
```
cd var/www/backend
pm2 start app.js
```
