# PrestaShop Ecommerce Deployment Task
A step-by-step guide to setup PrestaShop using AWS

Prestashop (https://www.prestashop.com/en) is an Open Source Software.

![Finished Shop](./awsimgs/prestafront.gif)

Create a New Server Instance and Install PrestaShop on the Instance. 
Your new Prestashop Installation should have a publicly accessible URL (use AWS default). 
- Submit the URL to the working Prestashop Installation hosted on your new Instance. 
- Submit a step by step documentation of your implementation process. Include Screenshots of your AWS Console and relevant configuration decisions taken.

**Important**: Your Database should not be hosted on the same Server hosting your Application. 

**Important**: Use only the Free-Tier of Amazon Web Services to answer this Section: 

### Implementation: 
- PrestaShop Installation: http://ec2-3-92-78-204.compute-1.amazonaws.com/
- Documentation: https://github.com/mrdankuta/presta-setup/blob/main/doc.md


## Services Used
- AWS EC2
- AWS RDS

## Steps
- Networking Configuration
- Provision RDS Database
- EC2 Instance Provisioning
  - Install Apache Webserver
  - Install PHP and necessary Extensions
  - Install MySQL Client
- Connect to remote database server from app server
- Create database for app
- Download PrestaShop installation pack
- Run installation in browser
- Login to ecommerce shop