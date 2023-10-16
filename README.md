# Project-3: Rearchitecting_Web_App_AWS_cloud
In this project, I led the migration and rearchitecting of a multi-tier web application. Initially, we shifted from traditional infrastructure to the cloud with a 'lift and shift' strategy ([Project-2](https://github.com/MadhuShetty1814/2.AWS_Cloud_Webapp_Lift-Shift)), setting the stage for an exciting transformation.
Subsequently, I restructured the application using cloud-native services, unlocking several key advantages. We enhanced scalability, optimized costs, bolstered resilience, and achieved greater development speed and security. This journey underscores the potential of cloud-native solutions and the power of DevOps and automation in shaping the future of modern web applications.

  ### About the Project:
  - Multi-Tier web application stack [Vprofile].
  - Re-Architect web app setup on AWS cloud using cloud native services.
  - Architecture to boost agility and improve business continuity.

  ### Scenario:
  - Project services running on Physical/VM/Cloud.
  - Varieties of services that powers your project runtime.
  - Cloud computing team, Virtualization team, DC Ops team, Monitoring team, Sys Admins, etc., are involved.

  ### Problem:
  - Operational Overhead.
  - Struggling with Uptime & Scaling.
  - Upfront Cap Ex and Regular Op Ex.
  - Manual process/Difficult to automate.

  ### Solution:
  - Cloud setup - PAAS/SAAS.
  - Pay As U Go.
  - IAAC.
  - Flexibility.
  - Ease of Infra management.

  ### Vprofile project Architecture:
  ![Architecture](https://github.com/MadhuShetty1814/3.Rearchitecting_Web_App_AWS_cloud/blob/main/Images/Architecture.png)

  ### AWS services used:
  1. Elastic Beanstalk - VM for Tomcat, Nginx LB replacement, Automation for VM scaling.
  2. S3 - Shared storage.
  3. RDS - Database.
  4. Elasticache - Memcached.
  5. AmazonMQ - RabbitMQ.
  6. CloudFront - Content Delivery Network(CDN).
  7. Amazon Certificate Manager[ACM] - For securing website.

  ### Flow of Execution:
  1. Login to AWS.
  2. Create a Key pair.
  3. Create Security Groups (SG) for Elasticache, RDS and AmazonMQ.
  4. Create Elasticache, RDS and AmazonMQ.
  5. Create Elastic Beanstalk.
  6. Update SG of Backend services to allow traffic from Beanstalk.
  7. Update SG of Backend services to allow internal traffic.
  8. Launch EC2 instance for DB initialization.
  9. Login to instance and initialize RDS DB.
  10. Change health check on Beanstalk to /login.
  11. Add HTTPS 443 listener to ELB.
  12. Build Artifact with Backend information.
  13. Deploy Artifact to Beanstalk.
  14. Create CDN with SSL certificate.
  15. Update entry in Domain DNS zones.
  16. Test the URL.

  ### Prerequisites:
  1. Chocolatey for Windows (Package manager)
      * Open PowerShell as admin
        - `$ Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))`
      * Press 'Y' when prompted
      * verify `$ choco --version`

  2. Java11
      `$ choco install adoptopenjdk11 --version=11.0.11.9`
      `$ java -version`

  3. Maven3
      `$ choco install maven --version=3.8.4`
      `$ maven -version`

  4. AWS CLI
      ```
      $ choco install python
      $ pip --version
      $ pip install awscli
      $ aws --version
      ```

  5. Create an ACM certificate
    + Request public certificate => provide domain name as *.<(domain_name)> => DNS validation => RSA key => request & attach it to the Domain service provider & validate it.
    ![ACM](https://github.com/MadhuShetty1814/3.Rearchitecting_Web_App_AWS_cloud/blob/main/Images/ACM.png)
    + Add CNAME and its value to the DNS service provider
    + Note: In CNAME, remove your domain name at last, and in value, remove full-stop. Wait for some time till the certificate is issued.
    ![DNS_entry](https://github.com/MadhuShetty1814/3.Rearchitecting_Web_App_AWS_cloud/blob/main/Images/DNS%20entry.png)
   
  ### Detailed steps:
  - Create key pair (pem):
    ![keypair](https://github.com/MadhuShetty1814/3.Rearchitecting_Web_App_AWS_cloud/blob/main/Images/keypair.png)

  - Create backend security group:
    For backend services (AmazonMQ, RDS, Elasticache) - Add dummy rule (SSH) to create SG and update it to all traffic from its own SG for internal communication.
    ![SG](https://github.com/MadhuShetty1814/3.Rearchitecting_Web_App_AWS_cloud/blob/main/Images/SG.png)

  - Create RDS:
    + Create Subnet group => give name => select default VPC => select all AZ's and subnets => create.
    + Create Parameter group => version mysql 8.0 => give name (rest defaults) => create.
    + Create database => standard create => MySQL => version 8 => select Free tier => Single DB instance => give name => username (admin) => autogenerated password => select burstable classes db.t2.micro (include previous gen) => general purpose storage (gp2) => 20gb storage => uncheck autoscaling (not required) => don't connect to EC2 => default VPC => select subnet group => public access (No) => select backend SG => port 3306 => enable enhanced monitoring => db name (accounts) => select parameter group => enable backup => retention period 7 days => select all log exports => enable auto minor version upgrade => create => view credentials => save password.
    ![RDS](https://github.com/MadhuShetty1814/3.Rearchitecting_Web_App_AWS_cloud/blob/main/Images/RDS.png)

  - Create Elasticache:
    + Create Subnet group => give name => select default VPC => select all AZ's => create.
    + Create Parameter group => give name => select latest version of memcached => create.
    + Create cluster => memcache cluster => standard create => AWS cloud => give name => engine version 1.6 (latest) => port 11211 => select parameter group => cache.t2.micro => nodes 1 => select subnet group => select backend SG => create.
    ![memcache](https://github.com/MadhuShetty1814/3.Rearchitecting_Web_App_AWS_cloud/blob/main/Images/memcache.png)

  - Create AmazonMQ:
    + select RabbitMQ => single instance broker => give name => mq.t3.micro => username (rabbit) => password (rabbit123) => version 3 (latest) => private access => default VPC => select SG => create.
    ![rabbitmq](https://github.com/MadhuShetty1814/3.Rearchitecting_Web_App_AWS_cloud/blob/main/Images/rabbitmq.png)

  - DB Initialization:
    + Launch EC2 => name (mysql client) => ubuntu 22 => select key pair => create SG (Allow SSH 22) => Launch.
    + Update backend SG => add 3306 from mysql client SG.
    ![mysqlSG](https://github.com/MadhuShetty1814/3.Rearchitecting_Web_App_AWS_cloud/blob/main/Images/mysqlSG.png)

    + Login to EC2 - `$ ssh -i <keypair> <user>@<publicIP>` (user name and ssh command can be found by selecting the required instance and clicking on connect => ssh client)
    + Install mysql-client - `$ sudo apt update && sudo apt install mysql-client -y`
    + Login to RDS database - `$ mysql -h <RDS endpoint> -u admin -p<password> accounts` and exit - `mysql > exit`
    + Clone the source code - `$ sudo apt install git -y`
                            - `$ git clone https://github.com/MadhuShetty1814/3.Rearchitecting_Web_App_AWS_cloud.git`
                            - `$ cd 3.Rearchitecting_Web_App_AWS_cloud`
    + Initialize database - `$ mysql -h <RDS endpoint> -u admin -p<password> accounts < src/main/resources/db_backup.sql`
    + Check the database table - `$ mysql -h <RDS endpoint> -u admin -p<password> accounts` => `mysql > show tables;` (It should show the table) => exit - `mysql > exit`

  - Backend information:
    + Copy RabbitMQ endpoint, remove amqps:// at start and :(port) at the end.
    + Copy Elasticache endpoint and remove :(port) at the end.
    + Save it in notepad (These should be updated in application.properties file before building artifact).

  - Create Elastic Beanstalk:
    + IAM => create roles => AWS service => EC2 => name (Beanstalk role) => policies -  1. BeanstalkRoleSNS
                                                                                        2. AdminAccess-Beanstalk
                                                                                        3. BeanstalkWebTier
                                                                                        4. CustomPlatformEC2Role
    ![BeanstalkRole](https://github.com/MadhuShetty1814/3.Rearchitecting_Web_App_AWS_cloud/blob/main/Images/BeanstalkRole.png)

    + Delete existing Beanstalk service role if present. 
    + Create application => web server environment => name of app => name of environment => give unique domain name (Check availability) => select Tomcat platform => version 8.5 with corretto 11 => sample app => custom config => create and use new service role => select key pair => select Beanstalk role => default VPC => select all subnets => PublicIP activated => add tags => Autoscaling (Loadbalanced) => min 2 max 2 => on demand => t2.medium => Application Load Balancer => listeners default => enhanced monitoring => Deployment policy (Rolling) percentage (50%) => create => check URL (Displays default tomcat page).
    ![Beanstalk](https://github.com/MadhuShetty1814/3.Rearchitecting_Web_App_AWS_cloud/blob/main/Images/Beanstalk.png)

    + S3 => open Beanstalk created S3 bucket => permissions => object ownership => ACL enabled
    + Elastic Beanstalk => environment => configuration => instance traffic and scaling (edit) => processes => action => edit => health check path (/login) => sessions => session stickiness enabled => save => add listener => 443 HTTPS => select certificate => add => apply. After sometime the health check status becomes severe, because we haven't deployed the application yet.
    + Open Beanstalk instance and copy its security group id => In backend SG, allow all traffic from beanstalk instance SG.

  - Build and Deploy Artifact:
    + Clone the source code to local - `$ git clone https://github.com/MadhuShetty1814/3.Rearchitecting_Web_App_AWS_cloud.git`
    + Edit application.properties file - (PATH: 3.Rearchitecting_Web_App_AWS_cloud/src/main/resources/application.properties)
      1. Paste the endpoint of RDS by replacing db01.
      2. Change username and password that you have created.
      3. Replace mc01 with memcache endpoint.
      4. Replace rmq01 with rabbitmq endpoint and also update its port number, user and password.
      5. Save it.
    ![application](https://github.com/MadhuShetty1814/3.Rearchitecting_Web_App_AWS_cloud/blob/main/Images/application.png)

    + In VS code => ctrl + shift + p => search default terminal profile => select git bash => view => terminal
    + In VS code terminal, check maven3, java11 and aws cli are installed.
    + Go to directory where pom.xml is present => build the artifact `$ mvn install`
    + Go to Beanstalk environment => Upload and Deploy => choose artifact (vprofile-v2.war) => deploy (Wait for about 10 mins).
    ![BeanstalkENV](https://github.com/MadhuShetty1814/3.Rearchitecting_Web_App_AWS_cloud/blob/main/Images/BeanstalkENV.png)

    + Check the URL.  
    + Copy the endpoint => entry in domain service provider => CNAME => name (vprofile) => target (paste URL) => add
    + verify in the browser => `https://vprofile.<domain>`

  - Create CloudFront:
    + Create distribution => enter the domain name => match viewer => HTTP method all => caching optimized => Alternate domain name (Same as domain name) => select certificate => TLSv1 => Create
    ![cloudfront](https://github.com/MadhuShetty1814/3.Rearchitecting_Web_App_AWS_cloud/blob/main/Images/cloudfront.png)
  
  - Validate:
    + verify in the browser using cloudfront domain name.
    + Login as admin_vp (username and password both) check the services
    ![WebApp](https://github.com/MadhuShetty1814/3.Rearchitecting_Web_App_AWS_cloud/blob/main/Images/web%20app.png)

  ### Credits:
  https://github.com/hkhcoder/vprofile-project
