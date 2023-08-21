## Load-Balancing-with-Apache

I created an ubuntu server which will server as loadbalancer to the webservers

![LBServer](https://github.com/Oolabanji/test_/assets/136812420/9ee2b41c-70c6-4de2-9dc2-7bb98752b008)


I opened up TCP port 80 in the load balancers inbound rule as requests are made through it.

![openport80](https://github.com/Oolabanji/test_/assets/136812420/cfc664bf-4c7e-4b37-aa68-16d34a018bd1)

### Installing Packages
I installed apache2, libxml and then configured apache for loadbalancing via enabling proxy and proxy_balancer

### Installing apache2

sudo apt update

sudo apt install apache2 -y

sudo apt-get install libxml2-dev

#Enable following modules:

sudo a2enmod rewrite

sudo a2enmod proxy

sudo a2enmod proxy_balancer

sudo a2enmod proxy_http

sudo a2enmod headers

sudo a2enmod lbmethod_bytraffic

#Restart apache2 service

sudo systemctl restart apache2

sudo systemctl status apache2

![apacheserverrunning](https://github.com/Oolabanji/test_/assets/136812420/d0df8a4a-f799-4e6a-9a6b-354b9def884d)


### Configuring Load Balancer

I edited the default.conf file to add the backend web servers into the loadbalancers proxy for routing.

'sudo vi /etc/apache2/sites-available/000-default.conf'

I added this configuration into this section <VirtualHost *:80>  </VirtualHost>

<Proxy "balancer://mycluster">
               BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
               BalancerMember http://<WebServer2-Private-IP-Address>:80 loadfactor=5 timeout=1
               ProxySet lbmethod=bytraffic
               # ProxySet lbmethod=byrequests
        </Proxy>

        ProxyPreserveHost On
        ProxyPass / balancer://mycluster/
        ProxyPassReverse / balancer://mycluster/


![virtualhost80](https://github.com/Oolabanji/test_/assets/136812420/46850040-efa2-49ee-abc1-8397cf70508c)

I restart the apache2 server 

sudo systemctl restart apache2

On the web browser, I test the load balancing connection using the public Ip address of the load balancer server.


![testloadbalancerserver](https://github.com/Oolabanji/test_/assets/136812420/0fe14bcd-d983-47e2-bca7-aa7c8d95076d)


To confirm that traffic is routed evenly to both web servers as the load balancer server is receiving traffic (which in our case is by refreshing the webpage) we can check the logs both servers receive sudo tail -f /var/log/httpd/access_log

Server1

![server1](https://github.com/Oolabanji/test_/assets/136812420/2868a898-d6db-4490-b548-296fc0cdf324)

Server2

![server2](https://github.com/Oolabanji/test_/assets/136812420/9b09355b-3a3a-415f-b3ce-a9bf4cc67514)


### Configuring DNS Names (Locally)

In order not to always provide webserver private ip address whenever a new web server needs to be added on the list of loadbalancer proxy, I specified them on the hosts file and provide a domain name for each.

sudo vi /etc/hosts

![hosts](https://github.com/Oolabanji/test_/assets/136812420/239b730a-a65c-44fb-bdfc-b54352992cc1)

Then I  curl the dns name on the loadbalancer server. Since the DNS names are local DNS configuration I can only access them locally hence the loadbalancer uses them locally to target the backend web servers

Server1
![web1](https://github.com/Oolabanji/test_/assets/136812420/50894d90-5d8b-4c7b-abe9-49dea675fb22)

Server2


![web2](https://github.com/Oolabanji/test_/assets/136812420/9ec994e0-c7d7-4865-b476-ccbfb273c4cf)
