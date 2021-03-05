# GCG_Wordpress

**HOW TO HOST A FREE WEBSITE ON GOOGLE CLOUD PLATFORM**
(without/with domain)  
---------------------------------------------------------------------------    
Based on:   
- https://www.youtube.com/watch?v=f56PG7QxjFI  
- https://tonyteaches.tech/google-cloud-free-website-hosting-tutorial/  
(But with few others steps)  

**The final project:**
http://35.222.237.117/
.    
    
**1. CREATE A GOOGLE CLOUD PLATFORM ACCOUNT**  
First things first. Create yourself a Google Cloud Platform (GCP) account. This video will walk you through the process of setting up your GCP account if you don’t already have one  
  
**Info**: 
- https://youtu.be/XcjeGDeSEew   
  
    
**2. SPIN UP A COMPUTE ENGINE VM ON THE FREE TIER**  
From the GCP dashboard, click on Compute Engine. Create a VM instance.  
  
In order to create your VM instance on the free tier, you must configure your VM with the following restrictions:  
  
Non-preemptible f1-micro VM instance  
US regions: Oregon (us-west1), Iowa (us-central1), or South Carolina (us-east1)  
Up to 30 GB-months HDD  
  
![image](https://user-images.githubusercontent.com/72107370/109911977-234bf680-7c79-11eb-8d07-3b2a17ce30c5.png)
  
Notice how it says “Your first 744 hours of f1-micro instance usage are free this month”. This number will vary depending on how many days are in the current month. For example, this screenshot was from October which has 31 days.  

31 days x 24 hours = 744 hours (for month)  
Feel free to choose any operating systems for the boot disk. I chose Ubuntu 20.04 LTS.  

**Warning**:  
- It is necessary billing que VM engines, so you must add your bill acount, but with this configuration there will no cost for this website  
- Dont forget to check, Allow HTTP trafic and Allow HTTPs trafic on Firewall
  
  
**3. CONNECT YOUR DOMAIN NAME** (optional)  

You can optionally associate a domain name with your IP address. If you don’t have a domain name, feel free to skip ahead to the next step.  

Otherwise, you can use create a DNS A record at your domain registrar with a value of the IP address of your Google Cloud Platform VM instance.  

(*I use this) In Google Domains, for example, you can add the DNS A records for your domain name. The screenshot assumes the IP address of your VM instance is 35.222.110.###.  

**Info**:   
It is possible purchase a domain for less than 3 o 12 $, I put a few web sites for buy a domain, if you want, but if you are watching this you don't need this :)    
- http://domain.google/  
- https://www.dreamhost.com/  
  
  
**4. LOGIN TO YOUR SERVER**  
  
Go to VM instance, go to connect in your intance they are differents option we go with.  
- Open in browser windows (it is a terminal of your computer in GCP)  
   
![image](https://user-images.githubusercontent.com/72107370/109912014-3f4f9800-7c79-11eb-8fdd-2a7df305ba36.png)
     
         
**5. UPDATE/UPGRADE YOUR VM**  

Once you’re logged in to your server, the first thing you want to do is update your system.  
  
- sudo apt update   
- sudo apt upgrade  
- sudo apt-get install nano   

**Info**:  
(Why? i like nano but you can install any tex editor, it will be necessary for configurated text files)  
[Fast commands: Ctrl o (save), Enter, Ctrl x] Also in this editor you can copy an paste text, cool no!  
   
      
**6. INSTALL THE WEB SERVER, DATABASE, AND PHP**  
  
Use the apt package manager to install the Nginx web server, Mariadb database, and PHP.  
  
- sudo apt-get install nginx mariadb-server php-fpm php-mysql  
  
**Info**:
- [For see some sentences for MariaDB: https://www.ochobitshacenunbyte.com/2018/11/15/listar-bases-de-datos-y-usuarios-en-mysql-y-mariadb/]  
  
Run on the console:
- php --version
- mysql --version
For be sure.
   
      
**7. SETUP THE WORDPRESS DATABASE**  
  
First, secure your database installation. After executing the following command, answer Y for each security configuration option.
- sudo mysql_secure_installation  
  
**Info**:
Type your password o set it, and then yes, yes, yes ...  
    
**Info**:
- sudo mysql -u root -p
u for user, p for passw, and you get in into MariaDB to run sql consults.
  
Create a database and user with appropriate privileges for WordPress. Access the MySQL command prompt by simply typing mysql.  
- create database example_db default character set utf8 collate utf8_unicode_ci;  
- create user 'example_username'@'localhost' identified by 'example_password';  
- grant all privileges on example_db.* TO 'example_username'@'localhost';  
- flush privileges;  
- exit  
  
**Warning**:  
It is important that you write on paper your DB, user and pass, we going to use in the wordpress setup.  
After all it is conf and worpress is runing, it is possible add new user in the setup interface.
  
  
**8. INSTALL WORDPRESS**  
  
Next let’s download and install the latest version of WordPress from the official website.  
- cd /var/www  
- sudo wget https://wordpress.org/latest.tar.gz  
- sudo tar -zxvf latest.tar.gz  
- sudo rm latest.tar.gz  

Also, change the owner and group of the WordPress root directory to www-data.  
- sudo chown www-data:www-data -R wordpress/  
  
  
**9. CONFIGURE NGINX TO SERVER YOUR WORDPRES WEBSITE**   

Make a configuration file for your WordPress website at /etc/nginx/sites-available/example.conf with the following content adjusted accordingly for your website. Of course, feel free to name your configuration as you see fit.  

**Warning**:  
This step is the more difficult, So.. baby steps  
It is necessary use nano (see step 5)  

	upstream example-php-handler {
        server unix:/var/run/php/php7.4-fpm.sock; #Use this beacause it is the php version, can run php --version to see your version
	}
	server {
        listen 80; 
        server_name example.com www.example.com; # In my case i use an external ip so i change to 35.222.237.117; or put server_name _;
        root /var/www/wordpress;
        index index.php;
        location / { 
                try_files $uri $uri/ /index.php?$args;
        }   
        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass example-php-handler; # See that this it the ref to upstream on the top of the doc.
        }   
	}

You will need to change the server_name option to your domain name, or if you don’t have a domain name, simply change this line to server_name _;.   
Also, depending on what version of PHP was installed, you may need to update line 2 to the actual version of PHP that’s installed on your server.
Finally, publish your website by making a symlink from your sites-available/example.conf file to the sites-enabled directory.	  
  
- sudo ln -s /etc/nginx/sites-available/example.conf /etc/nginx/sites-enabled/  
  
Test your Nginx configuration changes and restart the web server.  
- nginx -t  
- systemctl restart nginx  
*Warning:  
(Remenber use sudo for the commands, if it is necessary)  
Clean your browser cache 
  
  
**10. SETUP WORDPRESS**  
  
Navigate to your IP address or domain name (in this case example.com) and you’ll see the famous five-minute WordPress installation process. In reality, it take about a minute to fill out this form.  
Give your website a title, username, and secure password.  
  
![image](https://user-images.githubusercontent.com/72107370/109912193-8c336e80-7c79-11eb-8812-53d8e1ae1e25.png)
  
![image](https://user-images.githubusercontent.com/72107370/109912207-935a7c80-7c79-11eb-9f1f-a7321cf5b666.png)
  
  
**FIN**  
  
(They lived happily ever after ...)  

Example:  
http://35.222.237.117/  
  
