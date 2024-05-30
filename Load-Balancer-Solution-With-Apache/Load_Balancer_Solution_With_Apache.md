# Load Balancer Solution With Apache

A Load Balancer (LB) distributes clients' requests among underlying Web Servers and makes sure that the load is distributed in an optimal way.

The diagrame below shows the architecture of the solution
![Architecture](./images/architecture-diag.png)

## Task
Deploy and configure an Apache Load Balancer for Tooling Website solution on a separate Ubuntu EC2 instance. Make sure that users can be served by Web servers through the Load Balancer.

## Prerequisites

Ensure that the following servers are installedd and configure already.

- Two RHEL9 Web Servers
- One MySQL DB Server (based on Ubuntu 24.04)
- One RHEL9 NFS Server

## Prerequisites Configurations

- Apache (httpd) is up and running on both Web Servers.
- ```/var/www``` directories of both Web Servers are mounted to ```/mnt/apps``` of the NFS Server.
- All neccessary TCP/UDP ports are opened on Web, DB and NFS Servers.
- Client browsers can access both Web Servers by their Public IP addresses or Public DNS names and can open the ```Tooling Website``` (e.g, ```http://<Public-IP-Address-or-Public-DNS-Name>/index.php```)

# Step 1 - Configure Apache As A Load Balancer

## 1. Create an Ubuntu Server 24.04 EC2 instance and name it Project-8-apache-lb

![ec2 lb](./images/ec2-lb-detail.png)

## 2. Open TCP port 80 on Project-8-apache-lb by creating an Inbounb Rule in Security Group

![Port 80](./images/lb-security-rule.png)

## 3. Instal Apache Load Balancer on Project-8-apache-lb and configure it to point traffic coming to LB to both Web Servers.

### i. Install Apache2

- Access the instance

```bash
ssh -i "my-devec2key.pem" ec2-user@18.219.148.178
```
![ssh](./images/ssh-lb.png)

- Update and upgrade Ubuntu

```bash
sudo apt update && sudo apt upgrade
```
![update ubuntu](./images/update-upgrade-lb.png)

- Install Apache

```bash
sudo apt install apache2 -y
```
![Apache](./images/install-apache.png)

```bash
sudo apt-get install libxml2-dev
```
![Apache dependencies](./images/install-dependencies.png)

### ii. Enable the following modules

```bash
sudo a2enmod rewrite

sudo a2enmod  proxy

sudo a2enmod  proxy_balancer

sudo a2enmod  proxy_http

sudo a2enmod  headers

sudo a2enmod  lbmethod_bytraffic
```
![modules](./images/enable-modules.png)

### iii. Restart Apache2 Service

```bash
sudo systemctl restart apache2
sudo systemctl status apache2
```
![Restart apache](./images/restart-apache.png)

## Configure Load Balancing

### i. Open the file 000-default.conf in sites-available

```bash
sudo vi /etc/apache2/sites-available/000-default.conf
```
### ii. Add this configuration into the section ```<VirtualHost *:80>  </VirtualHost>```

```bash
<Proxy “balancer://mycluster”>
            BalancerMember http://172.31.46.91:80 loadfactor=5 timeout=1
           BalancerMember http://172.31.43.221:80 loadfactor=5 timeout=1
           ProxySet lbmethod=bytraffic
           # ProxySet lbmethod=byrequests
</Proxy>


ProxyPreserveHost on
ProxyPass / balancer://mycluster/
ProxyPassReverse / balancer://mycluster/
```
![Server config](./images/apache-server-config.png)

### iii. Restart Apache

```bash
sudo systemctl restart apache2
```
![Restart apache](./images/restart-apache-status.png)

```bytraffic``` balancing method with distribute incoming load between the Web Servers according to currentraffic load. The proportion in which traffic must be distributed can be controlled bt ```loadfactor``` parameter.

Other methods such as ```bybusyness```, ```byrequests```, ```heartbeat``` can also be adopted.


## 4. Verify that the configuration works

### i. Access the website using the LB's Public IP address or the Public DNS name from a browser

![lb public ip](./images/lb-public-ip.png)
![lb-website](./images/lb-wesite.png)

__Note__: If in the previous project, ```/var/log/httpd``` was mounted from the Web Server to the NFS Server, unmount them and ensure that each Web Servers has its own log directory.

### ii. Unmount the NFS directory

- Check if the Web Server's log directory is mounted to NSF

```bash
df -h
sudo umount -f /var/log/httpd
```
If the directory is busy, the services using it needs to be stopped first.

- Check that the directory is unmounted
```bash
df -h
```
![unmount](./images/unmount.png)

### iii. Open two ssh consoles for both Web Server and run the command:

```bash
sudo tail -f /var/log/httpd/access_log
```
Wbe Server 1 ```access_log```
![web1 accesslog](./images/access-log-web1-1.png)

Wbe Server 2 ```access_log```
![web1 accesslog](./images/access-log-web2-1.png)

### iv. Refresh the browser page several times and ensure both Web Servers receive HTTP and GET requests. New records must apear in each web server log files. The number of request to each servers will be approximately the same since ```loadfactor``` is set to the same value for both servers. This means that traffic will be evenly distributed between them.

Web Server 1 ```access_log```
![logs](./images/access-log-web1-2.png)

Web Server 2 ```access_log```
![logs](./images/access-log-web2-2.png)


# Optional Step - Configure Local DNS Names Resolution

Sometimes it is tedious to remember and switch between IP addresses, especially if there are lots of servers to manage. It is best to configure local domain name resolution. The easiest way is use ```/etc/hosts``` file, although this approach is not very scalable, but it is very easy to configure and shows the concept well.

## Configure the IP address to domain name mapping for our Load Balancer.

### Open the hosts file

```bash
sudo vi /etc/hosts
```

### Add two records into file with Local IP address and arbitrary name for the Web Servers

![dns host](./images/dns-hosts.png)

### Update the LB config file with those arbitrary names instead of IP addresses

```bash
sudo vi /etc/apache2/sites-available/000-default.conf
```
```bash
BalancerMember http://Web1:80 loadfactor=5 timeout=1
BalancerMember http://Web2:80 loadfactor=5 timeout=1
```
![dns name](./images/dns-apache-config.png)


### Try to curl the Web Servers from LB locally

```bash
curl http://Web1
```
![curl web1](./images/curl-web1.png)

```bash
curl http://Web2
```
![curl web2](./images/curl-web2.png)


Remember, This is only internal configuration and also local to the LB server, these names will neither be 'resolvable' from other servers internally nor from the Internet.


### Conclusion

The mod_proxy_balancer module in Apache HTTP Server offers robust features for load balancing, including support for sticky sessions, health checks, and various load balancing algorithms. Properly configuring these options ensures high availability, scalability, and reliability for web applications.

