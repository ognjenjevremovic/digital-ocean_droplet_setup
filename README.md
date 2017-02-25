# Setup a new Digital Ocean droplet

##### Generate SSH key on your local machine

##### Create a Digital Ocean droplet and add ssh key in the process
* In case you need to add a second ssh key:
```sh
cat ~/.ssh/id_rsa.pub | ssh root@[your.ip.address.here] "cat >> ~/.ssh/authorized_keys"
```

##### Add a new user and give it root access:
```ssh
adduser user_name
visudo
```
* Bellow the line:
```
root ALL=(ALL:ALL) ALL
```
* Add a new line
```
user_name ALL=(ALL:ALL) ALL
```
*Of course, you should replace the **user_name**, with the actual username of the user you wish to add.*

##### Switch to the newly created user:
```sh
su user_name
```
##### Add a ssh key to enable login:
```sh
nano .ssh/authorized_keys
```
*paste your local ssh key inside (preferable, the one you entered when you made the droplet)*

##### Permit root user login:
```sh
nano /etc/ssh/sshd_config
```
Inside change the **PermitRootLogin** from *yes* to *no*
```
PermitRootLogin no
```
and add user(s) that are able to login
```
AllowUsers user_name
```
*Once again, replace the **user_name**, with the actual username you previously createad and that you wish to add. If you’re adding more than one user, just separate them with a blank space.*

##### Restrict the permissions of the authorized_keys file:
```sh
chmod 600 .ssh/authorized_keys
```

##### Reload ssh
```sh
service ssh restart
```

##### Exit

##### Login as the new user
```sh
user_name@IP_addr
```
*Replace the **user_name**, with the actual username you previously createad on the server and **IP_addr** with the IP address of your server.*

##### On the machine, install some packages and dependancies first:
```sh
sudo apt-get install build-essential curl git python-setuptools ruby
```

##### Install Node by adding it’s ppa:
```sh
curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
sudo apt-get install nodejs
```

##### Add an ssh key on your server to use git:
```sh
ssh-keygen -t rsa -b 4096 -C ‘your_email@email.com’
```
*This is not the same SSH key that you already added! This time you generate the SSH key on the server to be able to use your *private* Git/Github repositories.*

##### Add the newly generated key to your Git/Github account:

##### Install pm2, for running developed apps:
``` sh
npm install -g pm2
```

##### Pull a project from Github repo
* cd into your application directory (you cloned from Github) and run:
```sh
pm2 start entry_point_of_your_app.js
```

##### Then to run pm2 as a service, making the application run whenever the the machine reboots, run the following (in your application directory):
```sh
pm2 startup systemd
```
*Copy the output of the previous command and run it in bash.*

##### Install Nginx
```ssh
sudo apt-get install nginx
```

##### Open the default server block configuration file:
```sh
sudo nano /etc/nginx/sites-available/default
```

##### Replace the content of the file with:
```
server {
    listen 80;

    server_name your_domain_name;

    location / {
        proxy_pass http://localhost:port_number_on_which_the_app_is_running;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```
*Replace **your_domain_name** with your actual domain name and **port_number_on_which_the_app_is_running** with the actual port, on which your app is listening on.*

##### Check for syntax errors:
```sh
sudo nginx -t
```

##### Restart Nginx:
```sh
sudo systemctl restart nginx
```

##### Permit traffic to Nginx through the firewall, if you have it enabled:
```sh
sudo ufw allow 'Nginx Full'
```

##### Install Letsencrypt package (for HTTPS):
```sh
sudo apt-get install letsencrypt
```

##### Let's Encrypt client needs port 80 in order to verify ownership of the domain, so stop nginx temporarily:
```sh
sudo systemctl stop nginx
```

##### Run letsencrypt:
```sh
sudo letsencrypt certonly –standalone
```
*Use your actual domain name, instead of an IP address of your server as otherwise it won't work! If you don't have a domain name, use a serice such as xip.io .*

##### Open the default server block configuration file:
```sh
sudo nano /etc/nginx/sites-available/default
```

##### Replace the content of the file with:
```
# HTTP - redirect all requests to HTTPS:
server {
        listen 80;
        listen [::]:80 default_server ipv6only=on;
        return 301 https://$host$request_uri;
}

# HTTPS - proxy requests on to local Node.js app:
server {
        listen 443;
        server_name your_domain_name;

        ssl on;
        # Use certificate and key provided by Let's Encrypt:
        ssl_certificate /etc/letsencrypt/live/your_domain_name/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/your_domain_name/privkey.pem;
        ssl_session_timeout 5m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;
        ssl_ciphers	 'EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH';

        # Pass requests for / to localhost:PORT:
        location / {
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-NginX-Proxy true;
                proxy_pass http://localhost:port_number_on_which_the_app_is_running/;
                proxy_ssl_session_reuse off;
                proxy_set_header Host $http_host;
                proxy_cache_bypass $http_upgrade;
                proxy_redirect off;
        }
}
```
*Replace **your_domain_name** with your actual domain name and **port_number_on_which_the_app_is_running** with the actual port, on which your app is listening on.*

##### Check for syntax errors:
```sh
sudo nginx -t
```

##### Start Nginx again:
```sh
sudo systemctl start nginx
```

##### RUN the app :)
