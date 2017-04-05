---
title:  "Mattermost install -- PostgreSQL"
date:   2017-04-05 09:28:
categories: MATTERMOST
---

# CentOS 7.2 install PostgreSQL

1. Log in to the server that will host the database, and open a terminal window.
2. Download the PostgreSQL 9.4 Yum repository.

{% highlight c %}

	wget https://download.postgresql.org/pub/repos/yum/9.4/redhat/rhel-7-x86_64/pgdg-redhat94-9.4-3.noarch.rpm

{% endhighlight %}

3. Install the Yum repository from the file that you downloaded.

{% highlight c %}

  	sudo yum localinstall pgdg-redhat94-9.4-3.noarch.rpm

{% endhighlight %}

4. Install PostgreSQL.

{% highlight c %}

  	sudo yum install postgresql94-server postgresql94-contrib

{% endhighlight %}

5. Initialize the database.

{% highlight c %}

  	sudo /usr/pgsql-9.4/bin/postgresql94-setup initdb

{% endhighlight %}

6. Set PostgreSQL to start on boot.

{% highlight c %}

  	sudo systemctl postgresql-9.4.service enable

{% endhighlight %}

7. Start the PostgreSQL server.

{% highlight c %}

  	sudo systemctl start postgresql-9.4.service

{% endhighlight %}

8. Disable firewall

{% highlight c %}

  	sudo systemctl disable  firewalld.service

{% endhighlight %}

9. Stop firewall

{% highlight c %}

  	sudo systemctl stop firewalld.service

{% endhighlight %}

10. Switch to the postgres Linux user account that was created during the installation.

{% highlight c %}

  	sudo --login --user postgres

{% endhighlight %}

11.	Start the PostgreSQL interactive terminal.

{% highlight c %}

  	psql

{% endhighlight %}

12.	Create the Mattermost user ‘mmuser’.

{% highlight c %}

  	postgres=# CREATE USER mmuser WITH PASSWORD 'mattermost';

{% endhighlight %}

13.	Grant the user access to the Mattermost database.

{% highlight c %}

	postgres=# GRANT ALL PRIVILEGES ON DATABASE mattermost to mmuser;

{% endhighlight %}

14.	Exit the PostgreSQL interactive terminal.

{% highlight c %}

	postgre=# \q

{% endhighlight %}

15.	Log out of the postgres account.

{% highlight c %}

	exit

{% endhighlight %}

16. Allow Postgres to listen on all assigned IP Addresses.

{% highlight c %}

	vim /var/lib/pgsql/9.4/data/postgresql.conf
    Find the following line:
	\#listen_addresses = 'localhost'
	Uncomment the line and change localhost to *:
	listen_addresses = '*'

{% endhighlight %}

17. If the Mattermost server is on a separate machine, modify the file pg_hbe.conf to allow the Mattermost server to communicate with the database.
	- vim /var/lib/pgsql/9.4/data/pg_hba.conf
	- update pg_hba.conf config
18. restart pgsql

{% highlight c %}

	systemctl restart  postgresql-9.4.service

{% endhighlight %}

19. Verify that you can connect with the user mmuser
	
{% highlight c %}

	psql --host=localhost --dbname=mattermost --username=mmuser --password
	The PostgreSQL interactive terminal starts. To exit the PostgreSQL interactive 	terminal, type \q and press Enter.

{% endhighlight %}