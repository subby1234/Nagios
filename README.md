# Nagios

https://www.virtono.com/community/tutorial-how-to/how-to-install-nagios-on-ubuntu-22-04/

Update and Upgrade
sudo apt update && sudo apt upgrade -y


sudo apt install libldap-2.5-0 libldap-common



Install Prerequisites
sudo apt install autoconf gcc libc6 make wget unzip apache2 apache2-utils php libgd-dev libmcrypt-dev libssl-dev bc gawk dc build-essential snmp libnet-snmp-perl gettext


Create a Nagios User and Group
sudo useradd -m -s /bin/bash nagios
sudo groupadd nagcmd
sudo usermod -a -G nagcmd nagios
sudo usermod -a -G nagcmd www-data


Download and Compile Nagios on Ubuntu
cd /tmp
wget https://assets.nagios.com/downloads/nagioscore/releases/nagios-4.4.6.tar.gz


Extract and Compile Nagios Core
tar -zxvf nagios-4.4.6.tar.gz
cd nagios-4.4.6

./configure --with-nagios-group=nagios --with-command-group=nagcmd
make all


Install Nagios Core Binaries and Web Interface Files
sudo make install
sudo make install-commandmode
sudo make install-init
sudo make install-config
sudo /usr/bin/install -c -m 644 sample-config/httpd.conf /etc/apache2/sites-available/nagios.conf


Install Nagios on Ubuntu Plugins
sudo apt update
sudo apt install nagios-plugins
sudo cp /usr/lib/nagios/plugins/* /usr/local/nagios/libexec/

sudo /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
sudo systemctl restart nagios
sudo systemctl restart apache2

sudo ln -s /etc/apache2/sites-available/nagios.conf /etc/apache2/sites-enabled/

sudo a2enmod cgi rewrite

Set Nagios Admin Password
sudo htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin


Start Nagios and Apache Services
sudo systemctl enable nagios
sudo systemctl enable apache2
sudo systemctl restart nagios
sudo systemctl restart apache2


sudo systemctl status nagios
sudo systemctl status apache2

.....................................................................

Install check_nrpe on Master:
cd

sudo apt-get update
sudo apt install nagios-nrpe-plugin
/usr/lib/nagios/plugins/check_nrpe --version


sudo mkdir -p /usr/local/nagios/etc/servers

sudo nano /usr/local/nagios/etc/servers/Host1.cfg


define host {
    use             linux-server
    host_name       Host1
    alias           My Test Host
    address         192.168.1.10 
    max_check_attempts 5
    check_period    24x7
    notification_interval 30
    notification_period 24x7
}

define service {
    use             generic-service
    host_name       Host1
    service_description PING
    check_command   check_ping!100.0,20%!500.0,60%
}

define service {
    use             generic-service
    host_name       Host1
    service_description HTTP
    check_command   check_http
}


sudo systemctl restart nagios

......................................

Create a Configuration File for Slave2:


sudo nano /usr/local/nagios/etc/servers/Host2.cfg


2. **Define Host2 Configuration:**

```plaintext
define host {
    use             linux-server
    host_name       Host2
    alias           My Second Host
    address         192.168.1.20  # Update with the IP address of Host2
    max_check_attempts 5
    check_period    24x7
    notification_interval 30
    notification_period 24x7
}

define service {
    use             generic-service
    host_name       Host2
    service_description PING
    check_command   check_ping!100.0,20%!500.0,60%
}

define service {
    use             generic-service
    host_name       Host2
    service_description HTTP
    check_command   check_http
}

sudo systemctl restart nagios


............................................................................
ON SLAVES

Install Nagios NRPE Client


sudo apt update && sudo apt upgrade -y  

sudo apt install nagios-nrpe-server nagios-plugins  

Configuring Nagios Client
sudo nano /etc/nagios/nrpe.cfg  

allowed_hosts=127.0.0.1, 192.168.1.100


restart the NRPE service 
sudo systemctl restart nagios-nrpe-server  




Step 4: Verify Connection from Nagios

To verify the connection between the Nagios server and the NRPE client, log in to your Nagios server and check if it can communicate with the NRPE service properly.

/usr/lib/nagios/plugins/check_nrpe --version
/usr/lib/nagios/plugins/check_nrpe -H 192.168.1.11  

If successful, the output should be “NRPE v4.0.0”, indicating that the Nagios server has successfully communicated with NRPE.




.............................
Use /etc/nagios/nrpe.cfg for NRPE client configuration on the server where the service is running.
Use /usr/local/nagios/etc/services.cfg for defining service checks and configurations in Nagios for monitoring services.

To create a monitoring service in Nagios to check whether the Apache2 service is up or not, you can follow these steps:

1. Define a new service in your Nagios configuration file (usually located at `/usr/local/nagios/etc/services.cfg` or similar):

appropriate `check_command` defined in your Nagios configuration. 

The `check_http` command is commonly used to check the status of an HTTP server like Apache. Ensure that it is properly defined in commands configuration file 

sudo nano /etc/nagios/nrpe.cfg  

Check for or add:

command[check_http]=/usr/lib/nagios/plugins/check_http -H $ARG1$ -p $ARG2$



sudo nano /etc/nagios/nrpe.cfg


1. Define a new service in your Nagios configuration file (usually located at `/usr/local/nagios/etc/services.cfg` or similar):

define service {
    use                             generic-service
    host_name                       localhost
    service_description             Check Apache2 Service
    check_command                   check_nrpe!check_http!-H localhost
}
 
sudo systemctl restart nagios



 
..........................................................
1. Define Host (ftp-server.cfg):

cfg

define host {
    use         linux-server
    host_name   ftp-server
    alias       FTP Server
    address     192.168.1.200
}

2. Define Service Check (ftp.cfg):

cfg

define service {
    use                 generic-service
    host_name           ftp-server
    service_description FTP Service
    check_command       check_ftp
}

3. Configure NRPE and Check Command:
Ensure that the NRPE client is installed on the FTP server and that a command check_ftp is defined in the NRPE configuration file to check the status of the FTP service.

4. Notifications:
Configure notifications to be sent via email or other means when the FTP service is down or experiencing issues. You can define contact groups, email addresses, and notification commands in Nagios configuration files.
.................................................

1. On both the slaves run a CPU checking service
2. Reduce the check interval on slave2 to 1 minute since it’s a critical server
ChatGPT

To accomplish the tasks you've outlined, we'll need to make modifications to the Nagios configuration files on both slave servers. Below are the steps to achieve this:

    Configure CPU checking service on both slaves:
        Define a service check to monitor CPU usage on both slave servers.
        Use the appropriate plugin to check CPU usage (e.g., check_cpu).
        Set thresholds for warning and critical levels based on your requirements.

    Reduce check interval on slave2:
        Modify the check interval for the CPU service check on slave2 to 1 minute.
        This will ensure that Nagios checks the CPU status of slave2 more frequently.

Here's how you can implement these changes:

1. Define CPU checking service on both slaves:

cfg

# For Slave 1
define service {
    use                 generic-service
    host_name           slave1
    service_description CPU Usage
    check_command       check_nrpe!check_cpu
}

# For Slave 2
define service {
    use                 generic-service
    host_name           slave2
    service_description CPU Usage
    check_command       check_nrpe!check_cpu
}

2. Reduce check interval on slave2:

cfg

# For Slave 1 (Normal check interval)
define host {
    use         linux-server
    host_name   slave1
    ...
}

# For Slave 2 (Reduced check interval)
define host {
    use         linux-server
    host_name   slave2
    check_interval 1 ; Check the host every minute (60 seconds)
    ...
}

Make sure to adjust the configurations according to your actual server names, IP addresses, and paths to the check commands. Also, ensure that the appropriate NRPE plugin for checking CPU usage (check_cpu) is available on the slave servers.

After making these changes, restart the Nagios service to apply the new configurations. Once done, Nagios will start monitoring the CPU usage on both slave servers, with slave2 being checked every minute due to the reduced check interval.
