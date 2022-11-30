## LOAD BALANCER SOLUTION WITH NGINX AND SSL/TLS

This project consists of two parts:

1. Configure Nginx as a Load Balancer
2. Register a new domain name and configure secured connection using SSL/TLS certificates

Your target architecture will look like this:

![](./images/nginx_lb.png)

You can either uninstall Apache from the existing Load Balancer server, or create a fresh installation of Linux for Nginx.

1. Create an EC2 VM based on Ubuntu Server 20.04 LTS and name it `Nginx LB` (do not forget to open TCP port 80 for HTTP connections, also open TCP port 443 – this port is used for secured HTTPS connections)

![](./images/nginx%20LB.PNG)

2. Update `/etc/hosts` file for local DNS with Web Servers’ names (e.g. Web1 and Web2) and their local IP addresses

3. Install and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers

Update the instance and Install Nginx

```
sudo apt update
sudo apt install nginx
```

![](./images/update%20%26%26%20install%20Nginx.PNG)

Configure Nginx LB using Web Servers’ names defined in `/etc/hosts`

Open the default nginx configuration file.

`sudo vi /etc/nginx/nginx.conf`

![](./images/edit%20nginx%20conf0.PNG)

```
#insert following configuration into http section

 upstream myproject {
    server Web1 weight=5;
    server Web2 weight=5;
  }

server {
    listen 80;
    server_name www.domain.com;
    location / {
      proxy_pass http://myproject;
    }
  }

#comment out this line
#       include /etc/nginx/sites-enabled/*;
```

![](./images/edit%20nginx%20conf.PNG)

Restart Nginx and make sure the service is up and running

```
sudo systemctl restart nginx
sudo systemctl status nginx
```

![](./images/enable%20%26%26%20start%20nginx.PNG)

![](./images/nginx%20running.PNG)

## REGISTER A NEW DOMAIN NAME AND CONFIGURE SECURED CONNECTION USING SSL/TLS CERTIFICATES

Let us make necessary configurations to make connections to our Tooling Web Solution secured!

In order to get a valid SSL certificate – you need to register a new domain name, you can do it using any `Domain name registrar` – a company that manages reservation of domain names. The most popular ones are: `Godaddy.com, Domain.com, Bluehost.com`.

1. Register a new domain name with any registrar of your choice in any domain zone (e.g. `.com, .net, .org, .edu, .info, .xyz` or any other)

![](./images/free%20domain.PNG)

![](./images/create%20hosted%20zone.PNG)

![](./images/create%20hosted%20zone2.PNG)

![](./images/connect%20to%20hosting%20site.PNG)

![](./images/manage%20custom%20name%20server.PNG)

![](./images/change%20successful.PNG)

2. Assign an Elastic IP to your Nginx LB server and associate your domain name with this Elastic IP

You might have noticed, that every time you restart or stop/start your EC2 instance – you get a new public IP address. When you want to associate your domain name – it is better to have a static IP address that does not change after reboot. Elastic IP is the solution for this problem, learn how to allocate an Elastic IP and associate it with an EC2 server on this page.

3.  A record in your registrar to point to Nginx LB using Elastic IP address

![](./images/create%20record.PNG)

![](./images/www%20record.PNG)

![](./images/summary.PNG)

Check that your Web Servers can be reached from your browser using new domain name using HTTP protocol – `http://<your-domain-name.com>`

![](./images/browse%20my%20domain%20name.PNG)

4. Configure Nginx to recognize your new domain name

Update your `nginx.conf` with `server_name www.<your-domain-name.com>` instead of `server_name www.domain.com`

![](./images/update%20%26%26%20install%20Nginx.PNG)

![](./images/enable%20%26%26%20start%20nginx.PNG)

![](./images/nginx%20running.PNG)

![](./images/test%20nginx%20successful.PNG)

5. Install `certbot` and request for an SSL/TLS certificate

Make sure `snapd` service is active and running

`sudo systemctl status snapd`

Install certbot

`sudo snap install --classic certbot`

Request your certificate (just follow the certbot instructions – you will need to choose which domain you want your certificate to be issued for, domain name will be looked up from `nginx.conf` file so make sure you have updated it on step 4).

```
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --nginx
```

Test secured access to your Web Solution by trying to reach `https://<your-domain-name.com>`

You shall be able to access your website by using HTTPS protocol (that uses TCP port 443) and see a padlock pictogram in your browser’s search string.
Click on the padlock icon and you can see the details of the certificate issued for your website.

6. Set up periodical renewal of your SSL/TLS certificate

By default, LetsEncrypt certificate is valid for 90 days, so it is recommended to renew it at least every 60 days or more frequently.

You can test renewal command in `dry-run` mode

`sudo certbot renew --dry-run`

Best pracice is to have a scheduled job that to run renew command periodically. Let us configure a cronjob to run the command twice a day.

To do so, lets edit the `crontab` file with the following command

`crontab -e`

Add following line:

`* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1`

You can always change the interval of this cronjob if twice a day is too often by adjusting schedule expression.

# Congratulations!
You have just implemented an Nginx Load Balancing Web Solution with secured HTTPS connection with periodically updated SSL/TLS certificates