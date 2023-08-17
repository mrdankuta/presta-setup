# Step-by-step PrestaShop Installation Documentation

## Step 1: Prepare remote database server
- Navigate to `Azure DB for MySQL` in Azure Portal
![Alt text](./images/1.png)
- Create new MySQL server. Select `Flexible server` 
![Alt text](./images/image.png)
![Alt text](./images/image-1.png)
- When deployment is complete navigate to the resource and go to `Connect` under the `Settings` pane. Take note of the database credentials. This will be used to connect PrestaShop later.
![Alt text](./images/image-2.png)
- Create a database on the database server specifically for the PrestaShop installation.
  ![Alt text](./images/image-15.png)

## Step 2: Provision and Configure a Virtual Machine for the Application
- Navigate to `Virtual machines` and Create an instance. In this case I am using Ubuntu 22.04 image
  ![Alt text](./images/image-3.png)
  ![Alt text](./images/image-4.png)
- Download private key to be able to connect via SSH
  ![Alt text](./images/image-5.png)
  ![Alt text](./images/image-6.png)
- When the resource has been deployed, open a terminal on the local machine and connect via SSH with the private key downloaded earlier.
  ![Alt text](./images/image-7.png)
- The next step is to configure the server and install all that is required to run PrestaShop. In my case I have created a Bash script to complete all the necessary installations.
  ```
  #!/bin/bash

  # Update package lists
  sudo apt update

  # Install Apache
  sudo apt install apache2 -y

  # Install PHP and necessary extensions
  sudo apt install php php-curl php-dom php-fileinfo php-gd php-intl php-mbstring php-zip php-json php-iconv -y

  # Enable necessary Apache modules
  sudo a2enmod rewrite
  sudo a2enmod auth_basic

  # Configure php.ini settings
  sudo sed -i 's/allow_url_fopen = Off/allow_url_fopen = On/' /etc/php/7.4/apache2/php.ini
  sudo sed -i 's/register_globals = On/register_globals = Off/' /etc/php/7.4/apache2/php.ini
  sudo sed -i 's/upload_max_filesize = 2M/upload_max_filesize = 16M/' /etc/php/7.4/apache2/php.ini

  # Restart Apache
  sudo systemctl restart apache2

  # Install MySQL client
  sudo apt install mysql-client -y

  ```
- Check that Apache webserver is installed and running:
  ```
  sudo systemctl status apache2
  ```
  ![Alt text](./images/image-8.png)
- Return to the Azure Portal and navigate to `Public IP Addresses` -> `Configure` and enter a DNS name label to be able to use Azure's default DNS names instead of using public IPs. In my case I have entered `bincompresta`. Hence my DNS label will be `bincompresta.eastus.cloudapp.azure.com`.
  ![Alt text](./images/imagedns.png)

## Step 3: Install PrestaShop
- The PrestaShop official documentation can be found here: https://docs.prestashop-project.org/v.8-documentation/getting-started/installing-prestashop
- I implemented it by first downloading PrestaShop from their GitHub releases:
  ```
  wget https://github.com/PrestaShop/PrestaShop/releases/download/8.1.1/prestashop_8.1.1.zip
  ```
- Next, copy the downloaded file to `/var/www/html` which is the root folder for the Apache webserver:
  ```
  sudo cp prestashop_8.1.1.zip /var/www/html
  ```
- Navigate to the webserver's root folder:
  ```
  cd /var/www/html
  ```
- Delete the default `index.html` file that comes with the Apache webserver and unzip the prestashop zipped folder:
  ```
  sudo unzip prestashop_8.1.1.zip
  ```
  - Install `unzip` if not available:
    ```
    sudo apt-get install unzip
    ```
- Now, visit the dns label (`bincompresta.eastus.cloudapp.azure.com`) previously created in the browser to complete the installation with the PrestaShop GUI
  ![Alt text](./images/image-10.png)
- Follow the installation assistant and clicking `next` to proceed
  ![Alt text](./images/image-11.png)
- In my case I encountered an error when PrestaShop ran a System Compatibility check. In the bash script above, I had omitted a critical extension needed by PrestaShop, `PHP-Mysql`. Responsible for enabling PHP to connect to the MySQL database.
  ![Alt text](./images/image-12.png)
- To fix this, I executed the following command then clicked on the provided refresh button:
  ```
  sudo apt install php-mysql
  ```
  ![Alt text](./images/image-13.png)
- Enter information for the PrestaShop Ecommerce Store and credentials for the store's Admin account
  ![Alt text](./images/image-14.png)
- Next step is to connect the remote database. Enter the connection details from the `Azure DB for MySQL` server that was previously configured.
  
  ![Alt text](./images/image-16.png)
  - I encountered a blocker here. I used the same credentials to connect from `TablePlus` software from my computer. It was successful:
    ![Alt text](./imagetp.png)
  - I SSH'd into the Application Virtual Machine and tried to connect to the remote MySQL server as a client using the `mysql -h presta-db-server.mysql.database.azure.com -u bincomdb -p` command. It was successful.