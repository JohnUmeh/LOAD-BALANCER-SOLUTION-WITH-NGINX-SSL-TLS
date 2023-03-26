# LOAD-BALANCER-SOLUTION-WITH-NGINX-SSL-TLS

In this project, we will configure an Nginx server as a load balancer and also configure secure connection using SSL/TSL certificates

The project architecture

![Screenshot from 2023-03-26 15-53-29](https://user-images.githubusercontent.com/77943759/227784175-eed7cb2f-35da-4a1a-8e0d-aef5168b45e5.png)

Requirements include:
1. An Nginx server(UBUNTU 20.04 was used)- The load balancer
2. A new domain name registration and configuration of  secured connection using SSL/TLS certificates

## **CONFIGURE NGINX AS A LOAD BALANCER**

Create an EC2 VM based on Ubuntu Server 20.04 LTS and name it Nginx LB


Open the following ports: Port 80(for HTTP connections); and TCP port 443(for secured HTTPS connections)

![tcp443](https://user-images.githubusercontent.com/77943759/227786917-0345b571-7d06-40e3-84ba-4a79d7357fdb.png)

Update /etc/hosts file for local DNS with Web Servers’ names with their local server names and IP addresses (WEB1 and WEB2)

![edithosts](https://user-images.githubusercontent.com/77943759/227787234-151db7d3-4dc8-4806-ba31-7daa35422b24.png)

Install nginx

```
sudo apt update
sudo apt install nginx
```
Configure Nginx webserver Load Balancer with server names configured in /etc/hosts

`sudo vi /etc/nginx/nginx.conf`

Populate the file with the following
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

![sudoviedit](https://user-images.githubusercontent.com/77943759/227787687-2fd65b89-fe26-4fc7-8b7c-b90319c1b110.png)

Restart Nginx and check the status to be sure it is up and running

```
sudo systemctl restart nginx
sudo systemctl status nginx
```

## **REGISTER A NEW DOMAIN NAME AND CONFIGURE SECURED CONNECTION USING SSL/TLS CERTIFICATES**

We must register a domain name in order to get a valid SSL certificate. 

Register for domain name in any of these  [Domain name registrar](https://en.wikipedia.org/wiki/Domain_name_registrar): [Godaddy.com](https://www.godaddy.com/en-uk), [Domain.com](https://www.domain.com/), [Bluehost.com](https://www.bluehost.com/) or any other Registrar of your choice

Assign an Elastic IP to the Nginx LB server and associate the domain name with this Elastic IP. This is because a static IP is better when assigning an instance to a domain

**To create elastic IP:**

Open the Amazon EC2 console

In the navigation pane, choose Network & Security, Elastic IPs

Choose Allocate Elastic IP address

For Public IPv4 address pool

Amazon's pool of IPv4 addresses

Choose Allocate

Update A record in your registrar to point to Nginx LB using Elastic IP address

**Associate an Elastic IP address with an instance:**

Open the Amazon EC2 console 

In the navigation pane, choose Elastic IPs

Select the Elastic IP address to associate and choose Actions, Associate Elastic IP address

For Resource type, choose Instance

For instance, choose the instance with which to associate the Elastic IP address. You can also enter text to search for a specific instance

Choose Associate.

To learn more about elastic IPs click [here](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html)

![associateelasticip](https://user-images.githubusercontent.com/77943759/227789915-ff6f0d9f-a244-4288-ab5c-4279babdaaa6.png)

Check that the Web Servers can be reached from the browser using new domain name using HTTP protocol – 

`http://<your-domain-name.com>`

![accesstest](https://user-images.githubusercontent.com/77943759/227790096-1ea7d286-8c0b-4e0b-8495-46bc7ec92e11.png)

Configure Nginx to recognize your new domain name. Update etc/nginx/nginx.conf file. Change server_name www.domain.com to server_name www.<your-domain-name.com>. 

![nginxconf](https://user-images.githubusercontent.com/77943759/227790319-a857a747-f8ae-44ef-8720-38bbb2c723f5.png)

Install certbot and request for an SSL/TLS certificate

Make sure snapd service is active and running

`sudo systemctl status snapd`

![snapd](https://user-images.githubusercontent.com/77943759/227792885-93afd186-bba0-4a09-a521-6e6d3d2face6.png)


Install certbot

`sudo snap install --classic certbot`

![certbot](https://user-images.githubusercontent.com/77943759/227792977-de6eb14b-e891-4b07-b1e4-cae55a34d074.png)


Request your certificate: Ensure to have edited Nginx configuration file above because certbot to ask you to select the domain the certificate will be issued for an this has to have been updated in the configuration file for nginx

```
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --nginx
```

Test secured access to your Web Solution by trying to reach 

`https://<your-domain-name.com>`

Click on the padlock icon and you can see the details of the certificate issued for your website.> Note that TCP port 443 must be open to access HTTPS connection

![certificate](https://user-images.githubusercontent.com/77943759/227791187-d708cb22-14e8-484e-bcd7-1f7d6634810d.png)

Now, Set up periodical renewal of your SSL/TLS certificate

 Test renewal command in dry-run mode

`sudo certbot renew --dry-run`

Let us configure a cronjob to run the command twice a day

`crontab -e`

Add the following line

`* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1`

Change the setting to suit the duration you would prefer

![jobrenewal](https://user-images.githubusercontent.com/77943759/227792793-992f6d5f-3ee8-4e45-8f4c-c74b4c977025.png)

We have just implemented an Nginx Load Balancing Web Solution with secured HTTPS connection with periodically updated SSL/TLS certificates.





