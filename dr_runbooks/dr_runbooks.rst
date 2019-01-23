.. _dr_runbooks:

------------------------
Leap: DR Runbooks
------------------------

Overview
++++++++

**Estimated time to complete: 60 MINUTES**

Legacy disaster recovery configurations, which are created with Prism Element, use protection domains and third-party integrations to protect VMs, and they replicate data between on-premises Nutanix clusters.
Protection domains provide limited flexibility in terms of supporting operations such as VM boot order and require you to perform manual tasks to protect new VMs as an application scales up.

Leap uses an entity-centric approach and runbook-like automation to recover applications.
It uses categories to group the entities to be protected and to automate the protection of new entities as the application scales.
Application recovery is more flexible with network mappings, configurable stages to enforce a boot order, and optional inter-stage delays. Application recovery can also be validated and tested without affecting production workloads. All the configuration information that an application requires upon failover are synchronized to the recovery location.

You can use Leap between two physical data centers or between a physical data center and Xi Cloud Services.
Leap works with pairs of physically isolated locations called availability zones.
One availability zone serves as the primary location for an application while a paired availability zone serves as the recovery location.
While the primary availability zone is an on-premises Prism Central instance, the recovery availability zone can be either on-premises or in Xi Cloud Services.

Lab Setup
+++++++++

For this lab you will be using the HPOC you were assigned, as well as the secondary Prism Central you were assigned.

You will also need to deploy two CentOS7 VMs, and deploy Wordpress and MariaDB.

Create Multi-Tier Wordpress App
...............................

.. note::

  When creating the VMs you will use the **Secondary** network, and adding static IPs you have been assigned in the .230-.253 range.
  You will be using the same last octet in the 2nd assigned hosted POC cluster.
  The 2nd HPOC cluster will also have a secondary network that we will use as well. Also check that X.X.X.230+ is also free.

Check Web and DB name, and Secondary Prism Central assignments here - https://docs.google.com/spreadsheets/d/1gN5wEcsQrs2B6NFFom3md5VuPqQizyaafudA5HIghb0/edit?ts=5c475e5d#gid=0

In **Prism Central** > select :fa:`bars` **> Virtual Infrastructure > VMs**, and click **Create VM**.

Fill out the following fields:

- **Name** - *DRWeb1 - DRWeb12 based on assignment*
- **Description** - (Optional) Description for your VM.
- **vCPU(s)** - 2
- **Number of Cores per vCPU** - 1
- **Memory** - 2 GiB

- Select **+ Add New Disk**
    - **Type** - DISK
    - **Operation** - Clone from Image Service
    - **Image** - CentOS7.qcow2
    - Select **Add**

- Select **Add New NIC**
    - **VLAN Name** - Secondary
    - **IP Address**  - *DRWeb1 - DRWeb12 Assigned IP*
    - Select **Add**

Click **Save** to create the VM.

Click **Create VM**, and fill out the following fields:

- **Name** - *DRDB1 - DRDB12 based on assignment*
- **Description** - (Optional) Description for your VM.
- **vCPU(s)** - 2
- **Number of Cores per vCPU** - 1
- **Memory** - 2 GiB

- Select **+ Add New Disk**
    - **Type** - DISK
    - **Operation** - Clone from Image Service
    - **Image** - CentOS7.qcow2
    - Select **Add**

- Select **Add New NIC**
    - **VLAN Name** - Secondary
    - **IP Address**  - *DRDB1 - DRDB12 Assigned IP*
    - Select **Add**

Click **Save** to create the VM.

Power On the VMs.

Create Category
...............

In **Prism Central** > select :fa:`bars` **> Virtual Infrastructure > Categories**, and click **Create Category**.

Fill out the following fields:

- **Name**  - *initials*-DR
- **Purpose** - DR Runbooks
- **Values**  - DB
- **Values**  - web

.. figure:: images/drrunbooks_01.png

Click **Save**.

Assign Category
...............

In **Prism Central** > select :fa:`bars` **> Virtual Infrastructure > VMs**

Select the DRDB VM you created, and click **Manage Categories** from the **Actions** dropdown.

.. figure:: images/drrunbooks_02.png

Search for *initials*-**DR** you just created, and select *initials*-**DR:DB**.

.. figure:: images/drrunbooks_03.png

Click **Save**.

Select the DRWeb VM you created, and click **Manage Categories** from the **Actions** dropdown.

Search for *initials*-**DR** you just created, and select *initials*-**DR:Web**.

Click **Save**.

Configure DRDB VM
.................

Login to *DRDB1 - DRDB12 based on assignment* via ssh or Console session.

- **Username** - root
- **password** - nutanix/4u

First lets update all installed packages.

.. code-block:: bash

  yum -y update

Now set the hostname:

.. code-block:: bash

  nmtui

- **Hostname**  - drdbXX.ntnxlab.local (drdb1-drdb12 based on assignment)

Now disable the Firewall:

.. code-block:: bash

  systemctl disable firewalld

  systemctl stop firewalld

Turn off SELinux:

.. code-block:: bash

  setenforce 0

  sed -i 's/enforcing/disabled/g' /etc/selinux/config /etc/selinux/config

Install MariaDB:

.. code-block:: bash

  yum install -y mariadb mariadb-server

Start MariaDB, and set it to start on reboot:

.. code-block:: bash

  systemctl start mariadb

  systemctl enable mariadb


Create database for Wordpress (Use root user account):

.. code-block:: bash

  mysql -u root

  MariaDB [(none)]> CREATE DATABASE wpdb;

Create new MariaDB user for wordpress:

.. code-block:: bash

  CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'techsummit';

  CREATE USER 'wpuser'@'drwebXX IP local PC' IDENTIFIED BY 'techsummit';

  CREATE USER 'wpuser'@'drwebXX IP remote PC' IDENTIFIED BY 'techsummit';

  GRANT ALL PRIVILEGES ON wpdb.* TO 'wpuser'@'localhost';

  GRANT ALL PRIVILEGES ON wpdb.* TO 'wpuser'@'drwebXX IP local PC';

  GRANT ALL PRIVILEGES ON wpdb.* TO 'wpuser'@'drwebXX IP remote PC';

  MariaDB [(none)]> FLUSH PRIVILEGES;

  MariaDB [(none)]> quit

Configure the MariaDB server on database to listen on public IP (or all interfaces).

Edit the MariaDB configuration file (/etc/my.cnf.d/server.cnf).

.. code-block:: bash

  vi /etc/my.cnf.d/server.cnf

Add the following line:

.. code-block:: bash

  bind-address = 0.0.0.0

Restart MariaDB for the changes to take effect:

.. code-block:: bash

  systemctl restart mariadb

Configure DRWeb VM
..................

Login to *DRWeb1 - DRWeb12 based on assignment* via ssh or Console session.

- **Username** - root
- **password** - nutanix/4u

First lets update all installed packages.

.. code-block:: bash

  yum -y update

  yum install -y unzip

Now set the hostname:

.. code-block:: bash

  nmtui

- **Hostname**  - drwebXX.ntnxlab.local (drweb1-drweb12 based on assignment)

Now disable the Firewall:

.. code-block:: bash

  systemctl disable firewalld

  systemctl stop firewalld

Turn off SELinux:

.. code-block:: bash

  setenforce 0

  sed -i 's/enforcing/disabled/g' /etc/selinux/config /etc/selinux/config

Install the Apache web server:

.. code-block:: bash

  yum install -y httpd

Start the web server, and enable it to start upon server boot:

.. code-block:: bash

  systemctl start httpd

  systemctl enable httpd

In order to install and use PHP 7.2, we need to install REMI repositories:

.. code-block:: bash

  rpm -Uvh http://rpms.remirepo.net/enterprise/remi-release-7.rpm

  yum install -y yum-utils

  yum-config-manager --enable remi-php72

Next, install PHP 7.2 along with the required PHP extensions:

.. code-block:: bash

  yum install -y php php-cli php-mbstring php-gd php-mysqlnd php-xmlrpc php-xml php-zip php-curl

Finally, complete the LAMP installation by installing MariaDB client package:

.. code-block:: bash

  yum install -y mariadb mariadb-server

Start MariaDB, and set it to start on reboot:

.. code-block:: bash

  systemctl start mariadb

  systemctl enable mariadb

Configure the MariaDB server on database to listen on public IP (or all interfaces).

Edit the MariaDB configuration file (/etc/my.cnf.d/server.cnf).

.. code-block:: bash

  vi /etc/my.cnf.d/server.cnf

Add the following line:

.. code-block:: bash

  bind-address = 0.0.0.0

Restart MariaDB for the changes to take effect:

.. code-block:: bash

  systemctl restart mariadb

Download the latest WordPress version:

.. code-block:: bash

  curl https://wordpress.org/latest.zip -o wordpress.zip

Extract it to the /var/www//html directory on your server:

.. code-block:: bash

  unzip -d /var/www/html/ wordpress.zip

Set proper permissions on WordPress files and directories:

.. code-block:: bash

  chown apache:apache -R /var/www/html/wordpress/

Rename wp-config-sample.php WordPress configuration file to wp-config.php:

.. code-block:: bash

  mv /var/www/html/wordpress/wp-config-sample.php /var/www/html/wordpress/wp-config.php

Edit the wp-config.php file and modify the following lines

.. code-block:: bash

  vi /var/www/html/wordpress/wp-config.php

  /** The name of the database for WordPress */
  define('DB_NAME', 'wpdb');

  /** MySQL database username */
  define('DB_USER', 'wpuser');

  /** MySQL database password */
  define('DB_PASSWORD', 'techsummit');

  /** MySQL hostname */
  define('DB_HOST', ‘drdbXX.ntnxlab.local');

You will have to add these ones

.. code-block:: bash

  define( 'WP_HOME', 'http://drwebXX.ntnxlab.local' );
  define( 'WP_SITEURL', ‘http://drwebXX.ntnxlab.local' );

Now we will have to setup the Apache configuration so it can serve the WordPress directory.

Add the contents below in the /etc/httpd/conf.d/wordpress.conf file using vi or your favorite editor:

.. code-block:: bash

  vi /etc/httpd/conf.d/wordpress.conf

  Add the following lines (Update ServerName & ServerAlias):

  <VirtualHost *:80>
  ServerAdmin admin@your-domain.com
  DocumentRoot /var/www/html/wordpress
  ServerName drwebXX.ntnxlab.local
  ServerAlias drwebXX.ntnxlab.local

  Alias /matomo “/var/www/html/wordpress/”
  <Directory /var/www/html/wordpress/>
  Options +FollowSymlinks
  AllowOverride All

  </Directory>

  ErrorLog /var/log/httpd/wordpress-error_log
  CustomLog /var/log/httpd/wordpress-access_log common
  </VirtualHost>

Save the changes and restart Apache for the changes to take effect:

.. code-block:: bash

  systemctl restart httpd

Open http://drwebXX.ntnxlab.local in the web browser on your *initials*-**Windows-ToolsVM**, and finish the WordPress installation.

Create Protection Policy
++++++++++++++++++++++++

Leap is built into Prism Central and requires no additional appliances or consoles to manage. Before you can begin managing DR-Orchestration with Leap, the service must be enabled.

.. note::

  Leap can only be enabled once per Prism Central instance. If **Leap** displays a green check mark next to it, that means Leap has already been enabled for the Prism Central instance being used.

Enable Leap and Connect Availability Zone (Local)
.................................................

In **Prism Central**, click the **?** drop down menu, expand **New in Prism Central** and select **Leap**.

In **Prism Central** > select :fa:`bars` **> Administration > Availability Zones**, and click **Connect to Availability Zone**.

.. note::

  You can only setup the **Connect to Availability Zone** once to a given Prism Central.

Fill out the following fields:

- **Availability Zone Type**  - Physical location
- **IP Address for Remote PC**  - *Assigned DR PC IP*
- **Username**  - admin
- **Password**  - techX2019!

.. figure:: images/drrunbooks_04.png

Click **Connect**.

Enable Leap and Connect Availability Zone (Remote)
.................................................

In **DR Prism Central**, click the **?** drop down menu, expand **New in Prism Central** and select **Leap**.

In **DR Prism Central** > select :fa:`bars` **> Administration > Availability Zones**, and click **Connect to Availability Zone**.

.. note::

  You can only setup the **Connect to Availability Zone** once to a given Prism Central.

Fill out the following fields:

- **Availability Zone Type**  - Physical location
- **IP Address for Remote PC**  - *Assigned PC IP*
- **Username**  - admin
- **Password**  - techX2019!

.. figure:: images/drrunbooks_05.png

Click **Connect**.

.. note::

  If Leap has been enabled on both PC's and the PC’s have been paired, proceed.

Create Protection Policy
++++++++++++++++++++++++

In **Prism Central** > select :fa:`bars` **> Policies > Protection Policies**, and click **Create Protection Policy**.

Fill out the following fields:

- **Name**  - *initials*-Protection
- **Primary Location**  - Local AZ
- **Remote Location** - Assigned DR PC
- **Target Cluster**  - Assigned DR HPOC
- **Recovery Point Objective**  - Hours
- **Start immediately** - 1
- **Remote Retention**  - 2
- **Local Retention**  - 2

- Select **+ Add Categories**
    - **Select Categories - *initials*-**DR:Web**
    - **Select Categories - *initials*-**DR:DB**
    Select **Save**

.. figure:: images/drrunbooks_06.png

Click **Save**

Create Recovery Plan
++++++++++++++++++++++++

In **Prism Central** > select :fa:`bars` **> Policies > Recovery Plans**, and click **Create Recovery Plan**.

Fill out the following fields:

- **Primary Location**  - Local AZ
- **Remote Location** - Assigned DR PC

Click **Proceed**

Fill out the following fields:

- **Name**  - *initials*-Recover
- **Recovery Plan Description** - optional

Click **Next**

Select **+ Add Entities**

- **Search Entities by**  - VM Name
    - Add *DRDB1 - DRDB12 based on assignment*
    Select **Add**

.. figure:: images/drrunbooks_07.png

Click **+ Add New Stage**

.. figure:: images/drrunbooks_08.png

Select **+ Add Entities**

- **Search Entities by**  - VM Name
    - Add *DRWeb1 - DRWeb12 based on assignment*
    Select **Add**

.. note::

  Sometimes it can take up to 5 minutes for the individual VMs to be added to the protection policy.
  Since we added the policy at the start you should be good to go.

  If you don’t want to wait you can manually protect the VM by using “Protect” on the VM menu in PC.

Add in a delay between stages 1 and 2 or 60 seconds to make sure the database is up first before the web front end loads.

Click **+ Add Delay**

- **Seconds** - 60

Click **Add**

.. figure:: images/drrunbooks_09.png

Click **Next**

Virtual networks in on-premises Nutanix clusters are virtual subnets that are bound to a single VLAN.

At physical locations, including the recovery location, administrators must create these virtual subnets manually, with separate virtual subnets created for production and test purposes.

.. note::

  You must create these virtual subnets before configuring recovery plans.

When configuring a recovery plan, map the virtual subnets at the source location to the virtual subnets at the recovery location.

Fill out the following fields:

- Local AZ
    - **Virtual Network or Port Group** - Secondary

- Remote AZ
    - **Virtual Network or Port Group** - Secondary

.. figure:: images/drrunbooks_10.png

.. note::

  You can leave out the Test Failback Network as we don’t have enough networks setup. Typically, the Test Network will be a non-routable network.

  If you are not using Nutanix AHV IPAM and need to retain your IP addresses, you would need to install NGT. ESXi will always need NGT to reserve IP address.

Click **Done**, and click **Continue** on the "incomplete Network Mapping" warning.

Failover to the Remote AZ (PC)
++++++++++++++++++++++++++++++







Getting Engaged with the Product Team
+++++++++++++++++++++++++++++++++++++

+---------------------------------------------------------------------------------+
|  DR Runbooks Product Contacts                                                   |
+================================+================================================+
|  Slack Channel                 |  #Prism-Pro                                    |
+--------------------------------+------------------------------------------------+
|  Product Manager               |  Mark Nijmeijer, hmark.nijmeijer@nutanix.com   |
+--------------------------------+------------------------------------------------+
|  Product Marketing Manager     |                                                |
+--------------------------------+------------------------------------------------+
|  Technical Marketing Engineer  |  Dwayne Lessner, dwayne@nutanix.com            |
+--------------------------------+------------------------------------------------+


Takeaways
+++++++++
