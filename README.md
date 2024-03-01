# docker-nginx-phpfpm-mysql-modsecurity-phpmyadmin

# How to Setup Nginx Phpfpm Mysql Modsecurity Phpmyadmin on docker

## Step1: Install the necessary Ubuntu packages with the below command:
```
 apt update && apt install unzip docker.io docker-compose zip git
```
## Step2: Clone the default code
```
Git clone https://github.com/jassi-devops/docker-nginx-phpfpm-mysql-modsecurity-phpmyadmin.git
```
## Step3: Go to the nginx-conf directory and change the domain according to your env 
```
cd docker-nginx-phpfpm-mysql-modsecurity-phpmyadmin/webserver/nginx-conf
sed -i 's/wordpress.com/your_wordpress_domain/g' *
sed -i 's/joomla.com/your_joomla_domain/g' *
sed -i 's/drupal.com/your_drupal_domain/g' *
sed -i 's/phpmyadmin.com/your_phpmyadmin_domain/g' *
cd  ../../
```
Buld the image and start the container with docker-compose
```
docker-compose build && docker-compose up –d
```
## Step4: create database, user and assign the necessary permission

For creating the DB detail either we can take the ssh and run the mysql commands or login to the docker db container and execute the command
```
ssh -p 23 root@server_ip 
```
or
```
docker exec –it db bash
```
Enter into the MySQL with below command, it will ask for the password so you have to enter the password which is mentioned in the docker-nginx-phpfpm-mysql-modsecurity-phpmyadmin/docker-compose.yml file where you have to check the Mysql service block
```
mysql -u root -p
```
once you entered into MySQL, run the below command to create the DB, User, and necessary permissions
### For Joomla:
```
CREATE DATABASE joomla;
CREATE USER 'jm_user'@'%' IDENTIFIED WITH mysql_native_password BY 'YLUPeVNtpdPGEII0';
GRANT ALL PRIVILEGES ON joomla.* TO 'jm_user'@'%';
FLUSH PRIVILEGES;
```
### For wordpress

Database detail is mentioned in the mysql service that you can find in the docker-nginx-phpfpm-mysql-modsecurity-phpmyadmin/docker-compose.yml file
	
### For Drupal:
```
CREATE DATABASE joomla;
CREATE USER 'jm_user'@'%' IDENTIFIED WITH mysql_native_password BY 'YLUPeVNtpdPGEII0';
GRANT ALL PRIVILEGES ON joomla.* TO 'jm_user'@'%';
FLUSH PRIVILEGES;
```
Now run exit, to out from the Mysql
```
Exit
```
Now again run exit to out from the DB container
```
exit
```

## Step5: Add the WordPress DB detail in the wp-config.php
Goto WordPress wordpress/wordpress/wp-config.php directory and edits the file with vim or nano and change the WordPress DB detail as you created in the step 4.

define( 'DB_NAME', 'your_db_name' );
/** Database username */
define( 'DB_USER', 'your_db_user' );
/** Database password */
define( 'DB_PASSWORD', 'your_db_password' );
	
## Step6: Obtaining SSL Certificates and Credentials

Open the docker-compose file docker-compose.yml and change the command in the certbot service as shown below. Be sure to replace the webroot path, email address, and domain names listed here with your information:
```yaml
  certbot:
    depends_on:
      - webserver
    build: './certbot'
    container_name: certbot
    volumes:
      - certbot-etc:/etc/letsencrypt
      - ./wordpress/wordpress:/var/www/wordpress
      - ./drupal/drupal:/var/www/drupal
      - ./joomla/joomla:/var/www/joomla
      - ./phpmyadmin/phpmyadmin:/var/www/phpmyadmin
    command: certonly --webroot --webroot-path=/var/www/wordpress --email sammy@your_domain --agree-tos --no-eff-email --staging -d your_domain 
```
Save and exit the file.

Run the below command to create a staging SSL for the WordPress domain
```
docker-compose up --force-recreate --no-deps certbot
```	
You can now check that your certificates have been mounted to the webserver container with docker-compose exec:
```
docker-compose exec webserver ls -la /etc/letsencrypt/live
```	
Once your certificate requests succeed, the following is the output:
```
Output
total 16
drwx------    3 root     root          4096 May 10 15:45 .
drwxr-xr-x    9 root     root          4096 May 10 15:45 ..
-rw-r--r--    1 root     root           740 May 10 15:45 README
drwxr-xr-x    2 root     root          4096 May 10 15:45 your_domain
```
Now that you know your request will be successful, you can edit the certbot service definition to remove the --staging flag. Open docker-compose.yml:
```
vim docker-compose.yml
```
Find the section of the file with the certbot service definition, and replace the --staging flag in the command option with the --force-renewal flag, which will tell Certbot that you want to request a new certificate with the same domains as an existing certificate. The following is the certbot service definition with the updated flag:
```yaml
  certbot:
    depends_on:
      - webserver
    build: './certbot'
    container_name: certbot
    volumes:
      - certbot-etc:/etc/letsencrypt
      - ./wordpress/wordpress:/var/www/wordpress
      - ./drupal/drupal:/var/www/drupal
      - ./joomla/joomla:/var/www/joomla
      - ./phpmyadmin/phpmyadmin:/var/www/phpmyadmin
    command: certonly --webroot --webroot-path=/var/www/wordpress --email sammy@**your_domain** --agree-tos --no-eff-email **--force-renewal** -d **your_domain**
```
```
docker-compose up --force-recreate --no-deps certbot
```
## Step 7: In the same (step 4), you have to obtain the certificate for the rest of the domains.
### For drupal:
Change the command in docker-compose for the certbot service with –staging flag.
```
command: certonly --webroot --webroot-path=/var/www/drupal --email sammy@your_domain --agree-tos --no-eff-email --staging -d your_domain
```
Run the below commands to create the certificates.
```
docker-compose up --force-recreate --no-deps certbot
```
```
docker-compose exec webserver ls -la /etc/letsencrypt/live
```
Change the command in docker-compose for the certbot service with – renewal flag.
```
command: certonly --webroot --webroot-path=/var/www/drupal --email sammy@your_domain --agree-tos --no-eff-email --force-renewal -d your_domain
```
Run the below commands to create the certificates.
```
docker-compose up --force-recreate --no-deps certbot
docker-compose exec webserver ls -la /etc/letsencrypt/live
```

### For Joomla:
Change the command in docker-compose for the certbot service with –staging flag.
```
command: certonly --webroot --webroot-path=/var/www/joomla --email sammy@your_domain --agree-tos --no-eff-email --staging -d your_domain 
```
Run the below commands to create the certificates.
```
docker-compose up --force-recreate --no-deps certbot
docker-compose exec webserver ls -la /etc/letsencrypt/live
```
Change the command in docker-compose for the certbot service with – renewal flag.
```
command: certonly --webroot --webroot-path=/var/www/joomla --email sammy@your_domain --agree-tos --no-eff-email --force-renewal -d your_domain 
```
Run the below commands to create the certificates.
```
docker-compose up --force-recreate --no-deps certbot
docker-compose exec webserver ls -la /etc/letsencrypt/live
```	

### For phpmyadmin:
Change the command in docker-compose for the certbot service with –staging flag.
```
command: certonly --webroot --webroot-path=/var/www/phpmyadmin --email sammy@your_domain --agree-tos --no-eff-email --staging -d your_domain
```
Run the below commands to create the certificates.
```
docker-compose up --force-recreate --no-deps certbot
docker-compose exec webserver ls -la /etc/letsencrypt/live
```
Change the command in docker-compose for the certbot service with – renewal flag.
```
command: certonly --webroot --webroot-path=/var/www/phpmyadmin --email sammy@your_domain --agree-tos --no-eff-email --force-renewal -d your_domain
```
Run the below commands to create the certificates.
```
docker-compose up --force-recreate --no-deps certbot
docker-compose exec webserver ls -la /etc/letsencrypt/live
```

## Step 8: Enabling Nginx Conf with SSL and Mod_security 
Since you are going to recreate the webserver service to include these additions, you can stop it now:
```
docker-compose stop webserver
```
Goto nginx-conf directory and delete the domain that is used for http:// and rename the domain.conf-443 to domain.conf 
``` 
cd docker-nginx-phpfpm-mysql-modsecurity-phpmyadmin/webserver/nginx-conf
rm -rf wordpress.conf joomla.conf drupal.conf phpmyadmin.conf
mv wordpress.conf-443 wordpress.conf
mv joomla.conf-443 joomla.conf
mv drupal.conf-443 drupal.conf
mv phpmyadmin.conf-443 phpmyadmin.conf
```
Start the webserver service 
```
docker-compose up -d --force-recreate --no-deps webserver
```
Check your services with docker-compose ps:
```
docker-compose ps
```
The output should indicate that your db, phpmyadmin, joomla, drupal, wordpress and webserver services are running:

```
certbot      certbot certonly --webroot ...   Exit 0
db           docker-entrypoint.sh /usr/ ...   Up       0.0.0.0:23->22/tcp,:::23->22/tcp, 3306/tcp, 33060/tcp
drupal       docker-php-entrypoint php-fpm    Up       9000/tcp
joomla       /entrypoint.sh php-fpm           Up       9000/tcp
phpmyadmin   /docker-entrypoint.sh /usr ...   Up       0.0.0.0:24->22/tcp,:::24->22/tcp, 443/tcp, 80/tcp
webserver    /docker-entrypoint.sh /usr ...   Up       0.0.0.0:26->22/tcp,:::26->22/tcp,
                                                       0.0.0.0:443->443/tcp,:::443->443/tcp,
                                                       0.0.0.0:80->80/tcp,:::80->80/tcp
wordpress    docker-entrypoint.sh php-fpm     Up       9000/tcp
```

## Step9: Completing the Installation through the Web Interface
In your web browser, navigate to your server’s domain. Remember to substitute your_domain with your domain name:
```
https://your_domain
```

Important Things to Know
All applications and MySQL data are persistent

Wordpress Files directory: docker-nginx-phpfpm-mysql-modsecurity-phpmyadmin/wordpress/wordpress
Joomla Files directory: docker-nginx-phpfpm-mysql-modsecurity-phpmyadmin/joomla/joomla
Drupal Files directory: docker-nginx-phpfpm-mysql-modsecurity-phpmyadmin/drupal/drupal
Drupal Files directory: docker-nginx-phpfpm-mysql-modsecurity-phpmyadmin/drupal/drupal
Phpmyadmin Files directory: docker-nginx-phpfpm-mysql-modsecurity-phpmyadmin/phpmyadmin/phpmyadmin
Mysql Data directory: /var/lib/docker/volumes/app_db_data/_data

How to change SSH root user password
To change the SSH root user password for a specific application, you have to open the Docker file from the application-specific folder and change the password
For example, if we have to change the ssh root password for a webserver application then follow the below steps:
cd docker-nginx-phpfpm-mysql-modsecurity-phpmyadmin/webserver
edit the Dockerfile and change the password from Mysql@123 to your_password
FROM
RUN echo -n 'root:Mysql@123' | chpasswd
To
RUN echo -n 'root:your_password’| chpasswd

To make changes affective, you have to recreate the webservice service
docker-compose up -d --force-recreate --no-deps webserver

Same is for Mysql and phpmyadmin

