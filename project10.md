## LOAD BALANCER SOLUTION WITH NGINX AND SSL/TLS

This project consists of two parts:

1. Configure Nginx as a Load Balancer
2. Register a new domain name and configure secured connection using SSL/TLS certificates

Your target architecture will look like this:

![](./Project10/images/nginx_lb.png)

You can either uninstall Apache from the existing Load Balancer server, or create a fresh installation of Linux for Nginx.

1. Create an EC2 VM based on Ubuntu Server 20.04 LTS and name it `Nginx LB` (do not forget to open TCP port 80 for HTTP connections, also open TCP port 443 – this port is used for secured HTTPS connections)

![](./Project10/images/nginx%20LB.PNG)

2. Update `/etc/hosts` file for local DNS with Web Servers’ names (e.g. Web1 and Web2) and their local IP addresses

3. Install and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers

Update the instance and Install Nginx

```
sudo apt update
sudo apt install nginx
```

![](./Project10/images/update%20%26%26%20install%20Nginx.PNG)

Configure Nginx LB using Web Servers’ names defined in `/etc/hosts`

Open the default nginx configuration file.

`sudo vi /etc/nginx/nginx.conf`

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

![](./Project10/images/edit%20nginx%20conf0.PNG)

![](./Project10/images/edit%20nginx%20conf.PNG)

Restart Nginx and make sure the service is up and running

```
sudo systemctl restart nginx
sudo systemctl status nginx
```

![](./Project10/images/enable%20%26%26%20start%20nginx.PNG)

![](./Project10/images/nginx%20running.PNG)


