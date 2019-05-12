## You have to set a web server who should BE available on the VM’s IP or an host (init.login.com for exemple). About the packages of your web server, you can choose between Nginx and Apache. You have to set a self-signed SSL on all of your services.

You have to set a self-signed SSL on all of your services

https://www.digitalocean.com/community/tutorials/how-to-create-a-ssl-certificate-on-apache-for-debian-8
  
First, enable the SSL module of apache2 then reload the service
```
sudo a2enmod ssl
sudo service apache2 reload
```
  
We now need to create a directory where we'll put our certificate and private key.
```
sudo mkdir /etc/apache2/ssl
```

Then generate the certificate and private key  
-days *number of days the certificate will be valid*
-keyout *path to our generated key, here /etc/apache2/ssl/apache.key*
-out *path to our generated certificate, here /etc/apache2/ssl/apache.crt*
  
```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/apache2/ssl/apache.key -out /etc/apache2/ssl/apache.crt
```
  
Protect our private key and certificate
```
sudo chmod 600 /etc/apache2/ssl/*
```

We are now going to create our own website configuration file
```
cd /etc/apache2/sites-available/
sudo touch epham.conf
```

And the folder in which our html pages will be
```
cd /var/www/html/     # Go to folder where all available websites html folders are
sudo mkdir epham      # Create a folder for our website
cd epham
mkdir html            # Create html folder of our website
cd html
sudo touch index.html # Create homepage file
```

Now that all our website's folders are created, we are going disable the default websites and enable ours to make sure it is the only *enabled* website (there should be 3 available websites, *000-default, default-ssl and epham*.
```
sudo a2ensite epham.conf  # Enable our website configuration file
sudo a2dissite            # Disable first default website
  000-default
sudo a2dissite            # Disable second default website
  default-ssl
sudo systemctl reload apache2 # Reload apache service
```
  
In our configuration file */etc/apache2/sites-available/epham.conf*
```
<VirtualHost *:80>
	ServerName 10.12.1.140
	DocumentRoot /var/www/html/epham/html
	Redirect permanent /secure https://10.12.1.140
</VirtualHost>

<VirtualHost *:443>
    	SSLEngine On
    	SSLCertificateFile /etc/apache2/ssl/apache.crt
    	SSLCertificateKeyFile /etc/apache2/ssl/apache.key

    	ServerAdmin epham@student.42.fr
    	ServerName 10.12.1.140
      	DocumentRoot /var/www/html/epham/html/
    	ErrorLog /var/www/html/epham/log/error.log
    	CustomLog /var/www/html/epham/log/access.log combined
</VirtualHost>
```

Reload the service
```
sudo service apache2 restart
```

Now, go to your browser, and type your address (https) : *https://10.12.1.140*  
  
Click on *advanced* and go to your page
  
If you click on *Not secure*, your should find the certificate you have created.
