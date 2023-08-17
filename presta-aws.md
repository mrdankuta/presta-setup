# Step-by-step PrestaShop Installation Documentation

## Step 0: Prepare Networking Infrastructure 
- Itemize all that'll be needed for the entire infrastructure to work:
  - Create VPC
    ![Alt text](./awsimgs/image.png)
  - Create Subnet
    ![Alt text](./awsimgs/image-2.png)
  - Create Internet Gateway & associate with VPC
    ![Alt text](./awsimgs/image22233.png)
    ![Alt text](./awsimgs/image-1111.png)
    ![Alt text](./awsimgs/image-2222.png)
  - Navigate to VPC's default Route table and add route for connecting to the internet via the previously created Internet Gateway.
    ![Alt text](./awsimgs/rtb1image.png)
  - Associate Route Table to the Previously created Subnet. 
    ![Alt text](./awsimgs/rtb-sbntimage.png)
  - Create Security Group that will be attached to the RDS server and open port `3306` for connecting to `MySQL`. Inbound connection can be limited to the Subnet CIDR for higher security.
    ![Alt text](./awsimgs/rds-sg-image.png)
  - Create another Security Group for the webserver where the PrestaShop will be installed. Open port `80` for accessing the App via `HTTP`. Also open port `22` for `SSH` access.
    ![Alt text](./awsimgs/app-sg-image.png)


## Step 1: Prepare remote database server 
- Navigate to `RDS` in AWS Portal and create a database.
- Select MySQL engine type.
  ![Alt text](./awsimgs/rds1image.png)
- Choose the VPC the infrastructure in being built in. Choose the Security Group previously created for database connectivity.
  ![Alt text](./awsimgs/rds2image-1.png)
- Create database.
  ![Alt text](./awsimgs/rds3image-2.png)
- At the point of creating the database with the configuration above, an error will occur. Return to the subnets and create another subnet in another Availability Zone. Note the CIDR block.
  ![Alt text](./awsimgs/add-subnet-image.png)
- Another error about VPC DNS hostname not enabled. Go to the VPC -> Edit VPC Settings and enable DNS Hostnames
  ![Alt text](./awsimgs/dns-hostnames-image-1.png)

- Take note of the database credentials. This will be used to connect PrestaShop later.
  ![Alt text](./awsimgs/dbcredimage.png)

- Create a database on the database server specifically for the PrestaShop installation.


## Step 2: Provision and Configure a Virtual Machine for the Application
- Navigate to `EC2` and Launch an instance. In this case I am using Ubuntu 22.04 image
  ![Alt text](./awsimgs/ec21image.png)

- Download new private key or select a previously created one to be able to connect via SSH. Edit network setting to select the right VPC, Subnet, enable `auto-assign publi ip`, selct the security group previously created for the App server.
  ![Alt text](./awsimgs/ec22image.png)

- When the resource has been deployed, open a terminal on the local machine and connect via SSH with the private key downloaded earlier.

- The next step is to configure the server and install all that is required to run PrestaShop. In my case I have created a Bash script to complete all the necessary installations.
  ```
  #!/bin/bash

  # Update package lists
  sudo apt update

  # Install Apache
  sudo apt install apache2 -y

  # Install PHP and necessary extensions
  sudo apt install php php-mysql php-curl php-dom php-fileinfo php-gd php-intl php-mbstring php-zip php-json php-iconv -y

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
  ![Alt text](./awsimgs/scriptrunimage.png)
  ![Alt text](./awsimgs/setup-run.gif)
- You can visit the Public IP address or the Public DNS provided by AWS to check that the webserver was properly installed. You should see this image:
  ![Alt text](./awsimgs/a2worksimage.png)
- Next, we will connect to the remote database to prepare the database with the right privileges.
  ```
  # Connect to the Database
  mysql -h bincom-presta-rds.ckoz8ewfenvj.us-east-1.rds.amazonaws.com -u bincomdb -p

  # Create the database PrestaShop should use
  CREATE DATABASE bincomecom;

  # Create all privileges on the database to the admin user
  GRANT ALL PRIVILEGES ON bincomecom.* TO 'bincomdb';
  FLUSH PRIVILEGES;

  exit
  ```



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
  sudo rm -f index.html
  sudo unzip prestashop_8.1.1.zip
  ```
  - Install `unzip` if not available:
    ```
    sudo apt-get install unzip
    ```
- Now, visit the public dns in the browser to complete the installation with the PrestaShop GUI
  ![Alt text](./awsimgs/presta1image.png)
  ![Alt text](./awsimgs/presta2image.png)
  ![Alt text](./awsimgs/presta3image.png)
  ![Alt text](./awsimgs/presta4image.png)
- Enter RDS database credentials saved earlier
  ![Alt text](./awsimgs/presta5image.png)
- Wait for installation to complete
  ![Alt text](./awsimgs/presta6image.png)
- Installation complete. Note the message asking to delete the `install` folder.
  ![Alt text](./awsimgs/presta7image.png)
- In terminal navigate to `/var/www/html` and view the files in the directory
  ```
  cd /var/www/html
  ls
  ```
  ![Alt text](./awsimgs/prestafilesimage.png)
- Delete the `install` folder as required. The downloaded zip file can also be downloaded.
  ```
  sudo rm -rf install
  sudo rm -f prestashop_8.1.1.zip
  ```
  ![Alt text](./awsimgs/prestafiles2image.png)
- Click `Manage your store` to login to the PrestaShop admin dashboard.
  ![Alt text](./awsimgs/prestaloginimage.png)
  ![Alt text](./awsimgs/prestadashboardimage.png)
- Click `View my store` or visit the public dns from your EC2 instance to view the store front.
  ![Alt text](./awsimgs/prestafront.gif)