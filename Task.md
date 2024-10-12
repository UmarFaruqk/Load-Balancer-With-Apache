## Load Balancer Solution With Apache
After completing DevOps tooling website solution Project, you might wonder how a user will be accessing each of the webservers using 3 different IP addreses or 3 different DNS names. You might also wonder what is the point of having 3 different servers doing exactly the same thing.
When we access a website in the Internet we use an URL and we do not really know how many servers are out there serving our requests, this complexity is hidden from a regular user, but in case of websites that are being visited by millions of users per day (like Google or Reddit) it is impossible to serve all the users from a single Web Server (it is also applicable to databases, but for now we will not focus on distributed DBs).
When you have just one Web server and load increases - you want to serve more and more customers, you can add more CPU and RAM or completely replace the server with a more powerful one - this is called "vertical scaling". This approach has limitations - at some point you reach the maximum capacity of CPU and RAM that can be installed into your server.
Another approach used to cater for increased traffic is "horizontal scaling" - distributing load across multiple Web servers. This approach is much more common and can be applied almost seamlessly and almost infinitely (you can imagine how many server Google has to serve billions of search requests).
Horizontal scaling allows to adapt to current load by adding (scale out) or removing (scale in) Web servers. Adjustment of number of servers can be done manually or automatically (for example, based on some monitored metrics like CPU and Memory load).
**SCALABILITY**: Scalability is the measure of a system's ability to increase or decrease in performance and cost in response to changes in application and system processing demands.
In our set up in the previous project  we had 3 Web Servers and each of them had its own public IP address and public DNS name. A client has to access them by using different URLs, which is not a nice user experience to remember addresses/names of even 3 server, let alone millions of Google servers.
In order to hide all this complexity and to have a single point of access with a single public IP address/name, a Load Balancer can be used. A Load Balancer (LB) distributes clients' requests among underlying Web Servers and makes sure that the load is distributed in an optimal way.
Let us take a look at the updated solution architecture with an LB added on top of Web Servers (for simplicity let us assume it is a software L7 Application LB, for example - Apache, NGINX or HAProxy)
![reference image](/Pictures/pic1.PNG)
In this project we will enhance our Tooling Website solution by adding a Load Balancer to disctribute traffic between Web Servers and allow users to access our website using a single URL.
**Prerequisites**
Make sure that you have the following servers installed and configured within the previous project:
1. Two RHEL8 Web Servers
2. One MySQL DB Server (based on Ubuntu 20.04)
3. One RHEL8 NFS server

![reference image](/Pictures/pic2.PNG)

# Step-By-Step Procedure 
1. Create an Ubuntu Server 20.04 EC2 instance and name it *Project-8-apache-lb*
2. Open TCP port 80 on *Project-8-apache-lb* by creating an Inbound Rule in Security Group. ![reference image](/Pictures/pic3.PNG)
3. Install Apache Load Balancer on Project-8-apache-lb server and configure it to point traffic coming to LB to both Web Servers: 
1. Update the terminal using *sudo apt update*
2. Install apache2 using *sudo apt install apache2 -y* and *sudo apt-get install libxml2-dev* ![reference image](/Pictures/pic4.PNG)
3. Enable the following modules ![reference image](/Pictures/pic5.PNG)
4. Restart apache2 service *sudo systemctl restart apache2*
5. check the status *sudo systemctl status apache2* ![reference image](/Pictures/pic6.PNG)

# Configuring Load Balancer
1. run *sudo vi /etc/apache2/sites-available/000-default.conf* and add the private IP of both webservers ![reference image](/Pictures/pic7.PNG) save and exit
2. Then restart apache server *sudo systemctl restart apache2*
The **bytraffic** above is a method of balacind which distribute incoming load between your Web Servers according to current traffic load. and also with **loadfactor** We can control in which proportion the traffic must be distributed
3. Verify that our configuration works - try to access your LB's public IP address or Public DNS name from your browser using *http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php* and you should see this ![reference image](/Pictures/pic11.PNG) ![reference image](/Pictures/pic12.PNG)
**Note** If in the previous project, you mounted */var/log/httpd/* from your Web Servers to the NFS server - unmount them and make sure that each Web Server has its own log directory.
4. Open two ssh/Putty consoles for both Web Servers and run following command *sudo tail -f /var/log/httpd/access_log* 
5. Try to refresh your browser page *http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php* several times and make sure that both servers receive HTTP GET requests from your LB - new records must appear in each server's log file. The number of requests to each server will be approximately the same since we set *loadfactor* to the same value for both servers - it means that traffic will be disctributed evenly between them and you should see this ![reference image](/Pictures/pic8.PNG) and ![reference image](/Pictures/pic9.PNG)

# Configure Local DNS Names Resolution (Optional)
Sometimes it is tedious to remember and switch between IP addresses, especially if you have a lot of servers under your management. What we can do, is to configure local domain name resolution. The easiest way is to use /etc/hosts file, although this approach is not very scalable, but it is very easy to configure and shows the concept well. So let us configure IP address to domain name mapping for our LB.
1. Open the host file on the LB using *sudo vi /etc/hosts* and Add 2 records into this file, replace the Local IP address with an arbitrary name for both of your Web Servers ![reference image](/Pictures/pic13.PNG)
2. Now you can update your LB config file with those names instead of IP addresses ![reference image](/Pictures/pic14.PNG)
3. You can try to curl your Web Servers from LB locally using *curl http://Web1* or *curl http://Web2*  and you should see this ![reference image](/Pictures/pic15.PNG) and ![reference image](/Pictures/pic16.PNG)
   
Remember, this is only internal configuration and it is also local to your LB server, these names will neither be 'resolvable' from other servers internally nor from the Internet.

**Congratulations!**
You have just implemented a Load balancing Web Solution for your DevOps team. I hope you enjoyed the RIDE!