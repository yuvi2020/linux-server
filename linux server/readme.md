inux_server_configuration
This project about create a linux server and using it to host our web application

IP Address
IP Address: 13.127.214.54

Host Name
Host Name: ec2-13-127-214-54.ap-south-1.compute.amazonaws.com

Amazon Lightsail server setup
Open Amazon Lightsail website and create a account in it.

After creating an account go to log in page.

After log in, click on Create Instance.

Select Platform (Linux/Unix).

Select Blueprint (Select OS Only and then select Ubuntu 16.04 LTS).

Scroll down and click on Create, it will take some time to setup.

After it is set up, you will see running in the left corner of the status card. Write down the public IP address on a paper as you will use it a lot in the following steps.

Click on Manage, in next page it shows connect, now scroll down and you'll see an link like Account Page.

Click on Account Page and click the download button to download your Private Key. It is a .pem file, not the rsa file in other step-by-step guides, but we can still use it to log into our server.

Click the Networking tab and find the Add another at the bottom. Add port 123 and 2200. Amazon Lightsail allows only port 22 and 80 by default, no matter how you set it up in ubuntu's ufw.

Application      Protocol      Port range
SSH              TCP           22
HTTP             TCP           80
Custom           UDP           123
Custom           TCP           2200
Server Configuration
Save the .pem file where is your vagrant file is located.

Now we have to move/copy our .pem file into .ssh folder. I've saved my file as ak.pem. To move/copy our .pem file follow below command.

 1. sudo cp -R /vagrant/YOURFILENAME.pem ~/.ssh/YOURFILENAME.pem
We need to make our public key usable and secure. Going back to your terminal and write the command

 2. sudo chmod 600 ~/.ssh/YOURFILENAME.pem
Now we use this key to log into our Amazon Lightsail server

 3. sudo ssh -i ~/.ssh/YOURFILENAME.pem ubuntu@13.127.214.54
Amazon Lightsail does not allow you to log in as a root user, but we can switch to become a root user. By executing the following command we can become root user

 4. sudo su -
After we became root user, we've have to create a user called grader.

    5. sudo adduser grader
Now create a new file under the sudoers directory

 6. sudo nano /etc/sudoers.d/grader
and fill that with

    grader ALL = (ALL:ALL) ALL
then save it.(Control X, then type Y, then hit enter key on your keyboard) 7. Run the following commands to update all packages and install package:

    7. sudo apt-get update
    8. sudo apt-get upgrade
    9. sudo apt-get install finger
Open a new terminal (Windows + r) and give following command

 10. ssh-keygen -f ~/.ssh/udacity_key.rsa
Stay on the same terminal window and give the following command to read the public key. Copy that public key

 11. cat ~/.ssh/udacity_key.rsa.pub
Going back to the first terminal window where you are logged into Amazon Lightsail as the root user, move to grader's folder by

12. cd /home/grader
Create a .ssh directory

13. mkdir .ssh
Create a file to store the public key

14. touch .ssh/authorized_keys
Edit authorized_keys file

15. sudo nano .ssh/authorized_keys
Change the permissions

16. sudo chmod 700 /home/grader/.ssh
17. sudo chmod 644 /home/grader/.ssh/authorized_keys
Change the owner from root to grader

18. sudo chown -R grader:grader /home/grader/.ssh
Restart the ssh service

19. sudo service ssh restart
Disconnect from Amazon Lightsail Server

20. ~.
Log into the server as grader

21. ssh -i ~/.ssh/udacity_key.rsa grader@13.127.214.54
We now need to enforce the key-based authentication:

22. sudo nano /etc/ssh/sshd_config
Find the PasswordAuthentication line and change text after to no. After this, restart ssh again:

    23. sudo service ssh restart
We now need to change the ssh port from 22 to 2200, as required by Udacity:

24. sudo nano /etc/ssh/sshd_config
Find the Port line and change 22 to 2200. Restart ssh:

    25. sudo service ssh restart
Disconnect the server

26. ~.
and then log back through port 2200:

    27. ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@13.127.214.54
Disable ssh login for root user, as required by Udacity:

28. sudo nano /etc/ssh/sshd_config
Find the PermitRootLogin line and edit to no. Restart ssh:

29. sudo service ssh restart
Now we need to configure UFW to fulfill the requirement:

30. sudo ufw allow 2200/tcp
31. sudo ufw allow 80/tcp
32. sudo ufw allow 123/udp
33. sudo ufw enable
And check the status of ufw by giving following command:

    34. sudo ufw status
you'll get an output like this

    To                         Action      From
    --                         ------      ----
    2200/tcp                   ALLOW       Anywhere
    80/tcp                     ALLOW       Anywhere
    123/udp                    ALLOW       Anywhere
    2200/tcp (v6)              ALLOW       Anywhere (v6)
    80/tcp (v6)                ALLOW       Anywhere (v6)
    123/udp (v6)               ALLOW       Anywhere (v6)
Deploy Catalog Application
We will use a virtual machine, apache2, and postgre to host our application. Before we use any thing, we need to log into the Amazon Terminal through your Terminal with grader.

Install required packages

 35. sudo apt-get install apache2
 36. sudo apt-get install libapache2-mod-wsgi python-dev
 37. sudo apt-get install git
Enable mod_wsgi by

 38. sudo a2enmod wsgi
and start the web server by

    39. sudo service apache2 start
                  or
    40. sudo service apache2 restart
You should input the public IP address and you should see a page which shown as Apache2 Ubuntu Default Page. If you do not see the page, you have to check the error message and google a solution. 3. Set up the folder structure

    41. cd /var/www
    42. sudo mkdir catalog
    43. sudo chown -R grader:grader catalog
    44. cd catalog
Now we clone the project from Github

 45. git clone [your project link] catalog
Copy your link from your Github Profile, it'll be easy. 5. Create a .wsgi file

    46. sudo nano catalog.wsgi
and add the following thing into this file

    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0, "/var/www/catalog/")

    from catalog import app as application
    application.secret_key = 'supersecretkey'
Rename the Project.py to init.py

 47. sudo mv project.py __init__.py
Now we need to install and start the virtual machine

 48. sudo pip install virtualenv
 49. sudo virtualenv venv
 50. source venv/bin/activate
 51. sudo chmod -R 777 venv
You should see a (venv) appears before your username in the command line. 8. Now we need to install the Flask and other packages needed for this application

    52. sudo apt-get install python-pip (If needed)
    53. sudo pip install flask
    54. sudo pip install httplib2
    55. sudo pip install oauth2client
    56. sudo pip install sqlalchemy
    57. sudo pip install psycopg2 (sometimes it asks to install psycopg2-binary. You'll find this while executing your program)
    58. sudo pip install requests
    59. sudo pip install redirect
    60. sudo pip install psslib
Use the below command to change the client_secrets.json line to /var/www/catalog/catalog/client_secrets.json

 61. sudo nano __init__.py
and change the host to your Amazon Lightsail public IP address and port to 80 and change the last line of your program to this

    before app.run(host='0.0.0.0', port=5000)
    after  app.run()
Now we need to configure and enable the virtual host

62. sudo nano /etc/apache2/sites-available/catalog.conf
paste the following code and save

    <VirtualHost *:80>
        ServerName [YOUR PUBLIC IP ADDRESS]
        ServerAlias [YOUR AMAZON LIGHTSAIL HOST NAME]
        ServerAdmin admin@35.167.27.204
        WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
        WSGIProcessGroup catalog
        WSGIScriptAlias / /var/www/catalog/catalog.wsgi
        <Directory /var/www/catalog/catalog/>
            Order allow,deny
            Allow from all
        </Directory>
        Alias /static /var/www/catalog/catalog/static
        <Directory /var/www/catalog/catalog/static/>
            Order allow,deny
            Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
You can find the host name in this link: http://www.hcidata.info/host2ip.cgi

Now we need to setup the database

63. sudo apt-get install libpq-dev python-dev
64. sudo apt-get install postgresql postgresql-contrib
65. sudo su - postgres
You should see the username changed again in command line, and type

    66. psql
to get into postgres command line.

Now we create a user to create and set up the database. I name my database catalog with user catalog

67. CREATE USER catalog WITH PASSWORD [your password];
68. ALTER USER catalog CREATEDB;
69. CREATE DATABASE catalog WITH OWNER catalog;
70. (Connect to database) \c catalog
71. REVOKE ALL ON SCHEMA public FROM public;
72. GRANT ALL ON SCHEMA public TO catalog;
73. Quit the postgrel command line: (\q) and then (exit)
use

74. sudo nano __init__.py
command to change all engine to engine = create_engine('postgresql://catalog:[your password]@localhost/catalog change you engine in database_setup.py also.

Run the database_setup.py and init.py by using

75. python database_setup.py
76. python __init__.py
Restart the Apache Server by using below command

77. sudo service apache2 restart
and enter your public IP address or host name into the browser. Your application should be online now!

To Enable and Disable default Apache2
After executing your programs it may give default apache page in website. To solve that issues follow below instructions

    78. (Disable) sudo a2dissite 000-default.conf
    79. (Enable) sudo a2ensite catalog
After performing both enable and disable operations you have to restart apache2

    80. sudo service ssh reload (Only after disabling the apache2)
    81. sudo service apache2 reload
    82. sudo service apache2 restart
After executing above command re-execute your database_setup.py and init.py files and enter your IP address in your website.

Reference
My sincere thanks to the people who poster their step-by-step on their Github: https://github.com/chuanqin3/udacity-linux-configuration