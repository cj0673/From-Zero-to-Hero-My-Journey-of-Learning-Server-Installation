# Installing Multiple Services on Debian 11

Welcome to the first installment of From Zero to Hero: My Journey of Learning Server Installation. In this article, I will take you through the process of setting up a web server from scratch on a Debian 11 system. We will cover the installation of Nginx, MariaDB, PHP, Nextcloud, PhpMyAdmin, Fail2ban, Nftables, and also show you how to use Certbot to create a SSL certificate for securing your server. This guide is meant to be a comprehensive guide for those who are new to server administration and want to learn how to set up and maintain a web server. Whether you're setting up a personal website or a small business website, this guide will provide you with the knowledge you need to get started. Let's begin!

***

## First

Let me first introduce my server setup and the requirements needed to install the packages. Please note that my methods may only serve as reference and suggestions if your setup or environment differs from mine.

1.I created a 4-core, 2GB memory VM using TrueNAS SCALE. You can also consider this VM as a 4-core, 2GB memory computer, and the fact that it's a VM should not affect the following operations.

2.I have access to a user with sudo privileges on a Debian system, and I am able to log in via SSH.

***

## Install Debian 11

First, let's go to the [Debian](https://www.debian.org/download) to download the ISO file. The current date is January 27th, 2023 and the latest version is Debian 11 (bullseye). Use this ISO file to boot your computer and install Debian 11. If you're planning on using a USB drive for booting, you may need to use a tool like [Rufus](https://rufus.ie).

Next, follow the Debian installation guide and install it according to your needs. In the installation guide, for the system to pre-install packages, select "System Basic Tools" and "SSH Tools", unselect other options. A * denotes that it is selected, and if it does not have a * it means it is not selected, please use the space bar to switch between selected and unselected.

After the Debian installation is complete, you will be prompted to reboot and remove the ISO file. Next, you can use SSH to connect to the host to complete further operations. When using SSH connection, you will need to use [Putty](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html), and in order to be able to automatically log in to SSH, I also used PuttyGen to generate a private key.

***

## Setup Putty Auto-login

First, open your PuttyGen, set the Parameters to RSA-2048 and select "show fingerprint as SHA265". Then, click on "Generate" and move your mouse randomly within the program window, the program will automatically generate the key. Once completed, click on "Save private key" and save it in the desired location. When prompted to save a key without a passphrase, please press "YES". For this example, we will "assume" that the saved location is /user/debian_private.ppk. This step is now complete, but do not close the PuttyGen program yet.

Next, open your Putty application. In the "Host Name" field, enter the IP address of your SSH host. In the Putty left sidebar, navigate to "Connection > Data" and enter the username you want to use for automatic login in the "Auto-login username" field. In this example, we will assume the username is "root." Then, navigate to "Connection > SSH > Auth" in the left sidebar, and enter the path of the private key file for authentication that you generated earlier. In our example, this path would be "/user/debian_private.ppk". Lastly, navigate to "Session" in the left sidebar, enter a name for the session in the "Saved Sessions" field, and click "Save". This will allow you to quickly connect to the SSH host in the future by double-clicking on this session name.

Next, we need to upload the public key to the host. Log in to the user you entered in the Auto-login username field. In our example, this user is root. Please create a .ssh directory under the root home directory. You need to use the command <pre><code>mkdir ~/.ssh </code></pre> and create a file named authorized_keys in the .ssh directory. You need to use the command <pre><code>nano authorized_keys</code></pre> copy all data from the Public Key for pasting into OpenSSH authorized_keys file in PuttyGen and paste it into the authorized_keys file. Then press Ctrl+S, Ctrl+X, save the file and exit.

Now, you have completed the setup for automatic login with Putty to SSH, and you will no longer need to enter a password for future connections as it will be done automatically through the use of the public key. Remember to keep your private key safe as it is important.

***

## Enable and configure firewall

Nftables is a packet filtering and classification framework that replaces the iptables, ip6tables, arptables, and ebtables utilities. It is designed to provide a new level of control over packet filtering, making it more flexible and efficient. nftables is built on the Netfilter project and uses the same hook and target infrastructure as iptables. It also provides a userspace command line utility, nft, for easy configuration and management.

In Debian 11, the nftables firewall service is already installed but not activated. You can start it and set it to automatically run at boot by using the commands <pre><code>systemctl start nftables
systemctl enable nftables</code></pre> You can view the current firewall rules by using the command <pre><code>nft list ruleset</code></pre>

***

## Install fail2ban

Fail2ban is an intrusion prevention software that monitors log files for failed login attempts and temporarily bans the IP address that made the attempts. It is commonly used to protect servers from brute-force attacks. It works by applying rules to the log files, and when a certain number of failed login attempts are detected from a specific IP address within a certain amount of time, that IP address is banned for a specified amount of time. The rules and ban settings are configurable, allowing administrators to fine-tune the protection for their specific needs. Fail2ban can be used to monitor and protect various services, such as SSH, HTTP, and SMTP.

You can install fail2ban via the apt repository using the command <pre><code>apt update
apt install fail2ban</code></pre> and then configure it by editing the configuration file located at <pre><code>/etc/fail2ban/jail.conf</code></pre> There are several common parameters that can be modified within the config file such as <pre><code>bantime, findtime, maxretry</code></pre>

Bantime is used to set the duration of time that an IP address is blocked for each offense, findtime is used to set the time period in which failed login attempts are counted, and maxretry is used to set the number of failed login attempts before an IP address is blocked. For example, if bantime is set to 1d, findtime is set to 10m, and maxretry is set to 3, then an IP address that makes 3 failed login attempts within 10 minutes will be blocked for 1 day. Additionally, in order for nftables to work in conjunction with fail2ban, you will need to modify the <pre><code>banaction, banaction_allports</code></pre> parameters in the conf file, and change them to nftables.

After modifying the configuration of fail2ban, remember to use the command <pre><code>systemctl restart fail2ban</code></pre> to restart the service. You can also use the command <pre><code>fail2ban-client status</code></pre> to check the current rules in operation.

***

## Install Nginx

Nginx is a web server and reverse proxy server. It is open-source and known for its high performance, stability, and low resource usage. Nginx can handle high levels of concurrent connections and is commonly used as a load balancer or for serving static content. It also supports various web protocols such as HTTP, HTTPS, and HTTP/2.

You can be installed using the command <pre><code>apt install Nginx</code></pre> from the apt repository.

***

## Install Certbot and configure SSL certificate

[Certbot](https://certbot.eff.org/) is a tool used to obtain and renew SSL/TLS certificates from Let's Encrypt, a free, automated, and open certificate authority (CA). It is designed to be run on a server and automate the process of obtaining and renewing SSL/TLS certificates, making it easy to secure web servers and other services. 

First, install snapd using the command <pre><code>apt install snapd</code></pre>then install the latest version of snapd by installing the core package <pre><code>snap install core</code></pre>Next, install Certbot using snap <pre><code>snap install --classic certbot</code></pre> Once Certbot is installed, you can use the command <pre><code>certbot --nginx</code></pre> to apply for an SSL certificate. During the certbot guide, enter the domain name for which you want to apply for the SSL certificate, for example <pre><code>www.example.com, example.com</code></pre> Certbot will then automatically complete the certificate application and incorporate the certificate into Nginx.

***

## Install MariaDB

[MariaDB](https://mariadb.org/) is an open-source relational database management system that is a fork of the popular MySQL database. It is known for its compatibility with MySQL and its focus on speed, security, and reliability. MariaDB includes features such as support for high-availability and scalability, as well as a wide range of storage engines and plugins.

It can be installed on Linux systems via the apt package repository by running the command <pre><code>apt install mariadb-server</code></pre> and further configured using the command <pre><code>mysql_secure_installation</code></pre> for basic database setup.

***

## Install phpMyAdmin

[phpMyAdmin](https://www.phpmyadmin.net/) is a web-based tool for managing MySQL and MariaDB databases. It allows users to interact with the databases through a graphical user interface, making it easy to perform tasks such as creating and modifying tables, managing users, and running SQL queries. It can be installed through package managers such as apt and yum, and is commonly used in conjunction with a web server like Apache or Nginx.

It can be installed via apt repository by running the command <pre><code>apt install phpmyadmin</code></pre> and following the installation prompts. It's important to note that the password set during the installation process is separate and different from your root password for added security.

The default installation location of phpMyadmin is <pre><code>/usr/share/phpmyadmin</code></pre> where you should see files related to phpMyadmin. After installation, we also need to modify some settings of Nginx in order for you to access phpMyadmin through the URL.

You can find a file named "default" in the directory <pre><code>/etc/nginx/sites-available</code></pre> open the default file using an editor and add the following content in the server{...} block<pre><code>location /phpmyadmin {
	root /usr/share/;
	index index.php index.html index.htm;
	location ~ ^/phpmyadmin/(.+.php)$ {
		include /etc/nginx/fastcgi_params;
	    try_files $uri =404;
	    root /usr/share/;
	    fastcgi_pass unix:/run/php/php7.4-fpm.sock;
	    fastcgi_index index.php;
	    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
	}
	location ~* ^/phpmyadmin/(.+.(jpg|jpeg|gif|css|png|js|ico|html|xml|txt))$ {
	    root /usr/share/;
	}
}</code></pre>

After saving and exiting the file, use the command "systemctl restart nginx" to restart the Nginx service, so that the recently set configuration takes effect.

***

# Install PHP

We also need to install PHP packages to allow Nginx to properly interpret PHP files. You can use <pre><code>apt install php-fpm</code></pre> in the apt repository to install PHP packages. After installation, some files need to be modified to make it more suitable for running Nextcloud. The directory <pre><code>/etc/php/7.4/fpm/php.ini</code></pre> should be modified, please note that the directory 7.4 is your php version. If your version is different from mine, please change it. Modify the following items <pre><code>cgi.fix_pathinfo=0
memory_limit = 512M
listen.mode = 0660
opcache.interned_strings_buffer=8</code></pre> and in the file <pre><code>/etc/php/7.4/fpm/pool.d/www.conf</code></pre> modify the following items <code><pre>listen.mode = 0660
pm.max_children = 20
pm.start_servers = 4
pm.min_spare_servers = 2
pm.max_spare_servers = 8
clear_env = no
env[HOSTNAME] = $HOSTNAME
env[PATH] = /usr/local/bin:/usr/bin:/bin
env[TMP] = /tmp
env[TMPDIR] = /tmp
env[TEMP] = /tmp</code></pre> Then restart php-fpm to make the new configuration file effective, use <pre><code>systemctl restart php7.4-fpm</code></pre>

***

## Install Nextcloud

[Nextcloud](https://nextcloud.com/) is an open-source, self-hosted file sharing and collaboration platform. It allows users to store and share files, as well as collaborate on documents and projects. Nextcloud also offers features such as calendar and contact management, video and audio calls, and task management. It can be installed on a server or local machine, and can be accessed through a web interface or mobile apps. Nextcloud is a popular alternative to proprietary file sharing services like Google Drive and Dropbox.

You can download the Nextcloud files by going to the [website](https://nextcloud.com/install/#instructions-server) and finding "DOWNLOAD SERVER," then "COMMUNITY PROJECTS" and clicking "Get ZIP file" under "Archive." This will give you a ZIP file. You can then upload the file to the <pre><code>/var/www/html</code></pre> directory on your host using a method such as FTP, or have the host use wget to download the ZIP.

After the ZIP file is downloaded, we need to extract it. You can install the unzip package through apt repository by running the command <pre><code>apt install unzip</code></pre> Once the installation is completed, use the following command to extract the file <pre><code>unzip latest.zip</code></pre> Note that you should switch to the directory where the file is located before running the command <pre><code>cd /var/www/html</code></pre>

Up to this point, the Nextcloud directory has been correctly placed in the web directory, but before we proceed with the installation, we need to set up the database account and database that Nextcloud will use. You can access your database through the web by visiting "www.example.com/phpmyadmin", using the root account and the root password that you set earlier. Once logged in to the database, navigate to "User Accounts" and click on "Add User Account." Set the "User name" to "nextcloud" using the "Use text field" option and the "Host name" to "Local". Set the "Password" using the "Use text field" option and create a password. Then, under "Database for user account", check the option for "Create database with same name and grant all privileges." Finally, click "Go" at the bottom of the form to complete the database setup.

Now, you can access your nextclod through the web by visiting "www.example.com/nextcloud" to enter your Nextcloud. After entering, please input some basic information in the installation guide. The administrator account at the top is used for the account and password you will use to log in to Nextcloud. The fields below are about the data storage location and database account. Please enter the information that we just registered in the database according to the information, the user is "nextcloud" and the database name is also "nextcloud", the database host is "localhost:3306". After all the data is entered, you can press the install button.

After the installation of nextcloud is complete, you will need to make some changes to the Nginx configuration to make it work properly. Since the .htaccess file in nextcloud is meant for Apache, but in this example we are using Nginx, you will need to modify the Nginx configuration file.

Please go to "/etc/nginx/sites-available" and edit the default file. Inside the server{...} block, add the following <pre><code>location = /robots.txt {
    allow all;
    log_not_found off;
    access_log off;
}

location ^~ /.well-known {
	# The rules in this block are an adaptation of the rules
	# in the Nextcloud `.htaccess` that concern `/.well-known`.
    location = /.well-known/carddav { return 301 /nextcloud/remote.php/dav/; }
	location = /.well-known/caldav  { return 301 /nextcloud/remote.php/dav/; }
    location /.well-known/acme-challenge    { try_files $uri $uri/ =404; }
	location /.well-known/pki-validation    { try_files $uri $uri/ =404; }
    # Let Nextcloud's API for `/.well-known` URIs handle all other
	# requests by passing them to the front-end controller.
	return 301 /nextcloud/index.php$request_uri;
}

location ^~ /nextcloud {
	index index.php index.html /nextcloud/index.php$request_uri;
    # set max upload size and increase upload timeout:
	client_max_body_size 512M;
	client_body_timeout 300s;
	fastcgi_buffers 64 4K;
    # Enable gzip but do not remove ETag headers
	gzip on;
	gzip_vary on;
	gzip_comp_level 4;
	gzip_min_length 256;
	gzip_proxied expired no-cache no-store private no_last_modified no_etag auth;
	gzip_types application/atom+xml application/javascript application/json application/ld+json application/manifest+json application/rss+xml application/vnd.geo+json application/vnd.ms-fontobject application/wasm application/x-font-ttf application/x-web-app-manifest+json application/xhtml+xml application/xml font/opentype image/bmp image/svg+xml image/x-icon text/cache-manifest text/css text/plain text/vcard text/vnd.rim.location.xloc text/vtt text/x-component text/x-cross-domain-policy;
	  add_header Referrer-Policy                      "no-referrer"   always;
	add_header X-Content-Type-Options               "nosniff"       always;
	add_header X-Download-Options                   "noopen"        always;
	add_header X-Frame-Options                      "SAMEORIGIN"    always;
	add_header X-Permitted-Cross-Domain-Policies    "none"          always;
	add_header X-Robots-Tag                         "none"          always;
	add_header X-XSS-Protection                     "1; mode=block" always;
	add_header Strict-Transport-Security		"15552000"	always;
    # The settings allows you to optimize the HTTP2 bandwitdth.
	# See https://blog.cloudflare.com/delivering-http-2-upload-speed-improvements/
	# for tunning hints
	client_body_buffer_size 512k;
    # Remove X-Powered-By, which is an information leak
	fastcgi_hide_header X-Powered-By;
    # Rule borrowed from `.htaccess` to handle Microsoft DAV clients
	location = /nextcloud {
		if ( $http_user_agent ~ ^DavClnt ) {
			return 302 /nextcloud/remote.php/webdav/$is_args$args;
		}
	}
    # Rules borrowed from `.htaccess` to hide certain paths from clients
	location ~ ^/nextcloud/(?:build|tests|config|lib|3rdparty|templates|data)(?:$|/)    { return 404; }
	location ~ ^/nextcloud/(?:\.|autotest|occ|issue|indie|db_|console)                  { return 404; }
	# Ensure this block, which passes PHP files to the PHP process, is above the blocks
	# which handle static assets (as seen below). If this block is not declared first,
	# then Nginx will encounter an infinite rewriting loop when it prepends
	# `/nextcloud/index.php` to the URI, resulting in a HTTP 500 error response.
	location ~ \.php(?:$|/) {
		# Required for legacy support
		rewrite ^/nextcloud/(?!index|remote|public|cron|core\/ajax\/update|status|ocs\/v[12]|updater\/.+|oc[ms]-provider\/.+|.+\/richdocumentscode\/proxy) /nextcloud/index.php$request_uri;
        fastcgi_split_path_info ^(.+?\.php)(/.*)$;
		set $path_info $fastcgi_path_info;
        try_files $fastcgi_script_name =404;
        fastcgi_pass unix:/run/php/php7.4-fpm.sock;
		include fastcgi_params;
		fastcgi_split_path_info ^(.+\.php)(/.+)$;
		fastcgi_param PATH_INFO $path_info;
		fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
		fastcgi_param modHeadersAvailable true;         # Avoid sending the security headers twice
		fastcgi_param front_controller_active true;     # Enable pretty urls
		fastcgi_intercept_errors on;
		fastcgi_request_buffering off;
		fastcgi_max_temp_file_size 0;
	}
	location ~ \.(?:css|js|svg|gif|png|jpg|ico|wasm|tflite|map)$ {
		try_files $uri /nextcloud/index.php$request_uri;
		add_header Cache-Control "public, max-age=15778463, $asset_immutable";
		access_log off;     # Optional: Don't log access to assets
		location ~ \.wasm$ {
			default_type application/wasm;
		}
	}
	location ~ \.woff2?$ {
		try_files $uri /nextcloud/index.php$request_uri;
		expires 7d;         # Cache-Control policy borrowed from `.htaccess`
		access_log off;     # Optional: Don't log access to assets
	}
	# Rule borrowed from `.htaccess`
	location /nextcloud/remote {
		return 301 /nextcloud/remote.php$request_uri;
	}
	location /nextcloud {
		try_files $uri $uri/ /nextcloud/index.php$request_uri;
	}
}</code></pre> and then restart Nginx using the command <pre><code>systemctl restart nginx</code></pre>

Next, you will need to modify the Nextcloud settings by adding the following content <pre><code>'default_phone_region' => 'TW',</code></pre> to the path "/var/www/html/nextcloud/config/config.php)", where TW should be replaced with your country code. The code can be referred to [ISO 3166-1 alpha-2](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2#Officially_assigned_code_elements).

With this, everything you have installed should be working properly. I am glad you have reached this point. This is my first article and also my first tutorial. I hope my teaching method will allow you to learn how to operate, rather than being too complex. The internet is a constantly evolving place, and I do not mind receiving any feedback from anyone. Perhaps you have a better method that is more convenient or secure, etc. I welcome your rational discussion and guidance. Humans progress through constant discussions and exchanges. I also welcome any corrections to my grammar or description errors. My English is not very good, and some parts of this article were completed using translation. If you have misunderstood something in the article, I apologize. Thank you again for reading this article. Let's improve together and have a great day!
