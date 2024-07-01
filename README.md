# Techplement
___
# Techplement - Week 1 Tasks

## Overview

This document provides a detailed description of the steps I followed to deploy WordPress and MySQL on AWS using EC2 instances in both monolithic and microservices architectures. Additionally, it includes instructions on setting up a welcome page in WordPress.

## Prerequisites

Before starting, I ensured the following prerequisites were met:

1. Created a free-tier AWS account.
2. Installed AWS CLI on my local machine and configured it with my AWS credentials.
3. Generated an SSH key pair for secure access to the EC2 instances.

## Task Breakdown

### Monolithic Architecture

#### Step 1: Launch an EC2 Instance

1. **Open the AWS Management Console** and navigated to the EC2 dashboard.
2. **Launch Instance**:
   - **AMI**: Selected an Ubuntu AMI (`ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64`).
   - **Instance Type**: Chose `t2.micro`.
   - **Key Pair**: Selected my previously created key pair.
   - **Security Group**: Created a new security group with the following rules:
     - Allowed SSH (port 22) from my IP.
     - Allowed HTTP (port 80) from anywhere.
3. Launched the instance and waited for it to be in the running state.

#### Step 2: Install WordPress and MySQL

1. Connected to the EC2 instance via SSH:
   ```bash
   ssh -i /key-pair.pem ubuntu@3.209.28.142
   ```
2. Updated the package list:
   ```bash
   sudo apt update
   ```
3. Installed Apache, MySQL, PHP, and related packages:
   ```bash
   sudo apt install apache2 mysql-server php libapache2-mod-php php-mysql -y
   ```
4. Secured MySQL installation:
   ```bash
   sudo mysql_secure_installation
   ```
5. Created a MySQL database and user for WordPress:
   ```bash
   sudo mysql -u root -p
   CREATE DATABASE wordpress;
   GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpressuser'@'localhost' IDENTIFIED BY 'password';
   FLUSH PRIVILEGES;
   EXIT;
   ```
6. Installed WordPress:
   ```bash
   cd /tmp
   wget https://wordpress.org/latest.tar.gz
   tar -xzvf latest.tar.gz
   sudo mv wordpress /var/www/html/
   sudo chown -R www-data:www-data /var/www/html/wordpress
   sudo chmod -R 755 /var/www/html/wordpress
   ```
7. Configured Apache:
   ```bash
   sudo nano /etc/apache2/sites-available/wordpress.conf
   ```
   Added the following content:
   ```apache
   <VirtualHost *:80>
       DocumentRoot /var/www/html/wordpress
   </VirtualHost>
   ```
   Enabled the configuration and rewrite module:
   ```bash
   sudo a2ensite wordpress.conf
   sudo a2enmod rewrite
   sudo systemctl restart apache2
   ```
8. Completed the WordPress installation by navigating to `http://3.209.28.142` in my web browser and following the setup instructions.

### Microservices Architecture

#### Step 1: Launch EC2 Instances

Repeated the instance launch process for two instances, one for WordPress and one for MySQL, with the same configurations but different names to identify them.

#### Step 2: Configure Security Groups

1. **MySQL Instance Security Group**:

   - Allowed SSH (port 22) from my IP.
   - Allowed MySQL (port 3306) from the WordPress instance's private IP address.

2. **WordPress Instance Security Group**:
   - Allowed SSH (port 22) from my IP.
   - Allowed HTTP (port 80) from anywhere.

#### Step 3: Install MySQL on MySQL Instance

1. Connected to the MySQL EC2 instance via SSH:
   ```bash
   ssh -i /path/to/your-key-pair.pem ubuntu@54.197.61.78
   ```
2. Updated the package list and installed MySQL:
   ```bash
   sudo apt update
   sudo apt install mysql-server -y
   ```
3. Secured MySQL installation:
   ```bash
   sudo mysql_secure_installation
   ```
4. Created a MySQL database and user for WordPress:
   ```bash
   sudo mysql -u root -p
   CREATE DATABASE wordpress;
   GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpressuser'@'wordpress-ec2-private-ip' IDENTIFIED BY 'password';
   FLUSH PRIVILEGES;
   EXIT;
   ```
5. Updated MySQL configuration to allow remote access:
   ```bash
   sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
   ```
   Changed `bind-address` to `0.0.0.0`:
   ```ini
   bind-address = 0.0.0.0
   ```
   Restarted MySQL:
   ```bash
   sudo systemctl restart mysql
   ```

#### Step 4: Install WordPress on WordPress Instance

1. Connected to the WordPress EC2 instance via SSH:
   ```bash
   ssh -i /path/to/your-key-pair.pem ubuntu@54.156.198.64
   ```
2. Updated the package list:
   ```bash
   sudo apt update
   ```
3. Installed Apache, PHP, and related packages:
   ```bash
   sudo apt install apache2 php libapache2-mod-php php-mysql -y
   ```
4. Installed WordPress:
   ```bash
   cd /tmp
   wget https://wordpress.org/latest.tar.gz
   tar -xzvf latest.tar.gz
   sudo mv wordpress /var/www/html/
   sudo chown -R www-data:www-data /var/www/html/wordpress
   sudo chmod -R 755 /var/www/html/wordpress
   ```
5. Configured Apache:
   ```bash
   sudo nano /etc/apache2/sites-available/wordpress.conf
   ```
   Added the following content:
   ```apache
   <VirtualHost *:80>
       DocumentRoot /var/www/html/wordpress
   </VirtualHost>
   ```
   Enabled the configuration and rewrite module:
   ```bash
   sudo a2ensite wordpress.conf
   sudo a2enmod rewrite
   sudo systemctl restart apache2
   ```
   This makes the homepage wordpress page.
6. Configured WordPress to connect to MySQL:
   ```bash
   sudo nano /var/www/html/wordpress/wp-config.php
   ```
   Updated the database settings:
   ```php
   define('DB_NAME', 'wordpress');
   define('DB_USER', 'wordpressuser');
   define('DB_PASSWORD', 'password');
   define('DB_HOST', 'mysql-ec2-private-ip');
   ```
7. Completed the WordPress installation by navigating to `http://54.156.198.64` in my web browser and following the setup instructions.

### Create a Welcome Page in WordPress

1. Logged into the WordPress admin dashboard.
2. Created a new page named "Welcome".
3. Set the "Welcome" page as the homepage:
   - Went to `Settings` > `Reading`.
   - Selected `A static page` and chose "Welcome" from the dropdown menu.
