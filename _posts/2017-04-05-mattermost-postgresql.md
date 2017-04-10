---
title:  "Mattermost install -- PostgreSQL"
date:   2017-04-05 09:38:26
categories: MATTERMOST
---

# CentOS 7.2 install PostgreSQL

按照官方的安装文档安装mattermost时，踩了一些坑，根据官方文档并结合自己的安装过程，重新整理了下mattermost的安装配置。
踩坑的原因：1.使用的CentOS版本为7.2,而官网的版本为7.1；2.CentOS 7之后有些命令跟7之前的不太一样。 3.官方文档关于pg的配置不够详细。（由于自己时pg小白，所以没能顺利搞下来，还是查资料配置pg）

**Log in to the server that will host the database, and open a terminal window.**

**Download the PostgreSQL 9.4 Yum repository.**

{% highlight c %}

wget https://download.postgresql.org/pub/repos/yum/9.4/redhat/rhel-7-x86_64/pgdg-redhat94-9.4-3.noarch.rpm

{% endhighlight %}

**Install the Yum repository from the file that you downloaded.**

{% highlight c %}

sudo yum localinstall pgdg-redhat94-9.4-3.noarch.rpm

{% endhighlight %}

**Install PostgreSQL.**

{% highlight c %}

sudo yum install postgresql94-server postgresql94-contrib

{% endhighlight %}

**Initialize the database.**

{% highlight c %}

sudo /usr/pgsql-9.4/bin/postgresql94-setup initdb

{% endhighlight %}

**Set PostgreSQL to start on boot.**

{% highlight c %}

sudo systemctl postgresql-9.4.service enable

{% endhighlight %}

**Start the PostgreSQL server.**

{% highlight c %}

sudo systemctl start postgresql-9.4.service

{% endhighlight %}

**Disable firewall**

{% highlight c %}

sudo systemctl disable  firewalld.service

{% endhighlight %}

**Stop firewall**

{% highlight c %}

sudo systemctl stop firewalld.service

{% endhighlight %}

**Switch to the postgres Linux user account that was created during the installation.**

{% highlight c %}

sudo --login --user postgres

{% endhighlight %}

**Start the PostgreSQL interactive terminal.**

{% highlight c %}

psql

{% endhighlight %}

**Create the Mattermost user ‘mmuser’.**

{% highlight c %}

postgres=# CREATE USER mmuser WITH PASSWORD 'mattermost';

{% endhighlight %}

**Grant the user access to the Mattermost database.**

{% highlight c %}

postgres=# GRANT ALL PRIVILEGES ON DATABASE mattermost to mmuser;

{% endhighlight %}

**Exit the PostgreSQL interactive terminal.**

{% highlight c %}

postgre=# \q

{% endhighlight %}

**Log out of the postgres account.**

{% highlight c %}

exit

{% endhighlight %}

**Allow Postgres to listen on all assigned IP Addresses.**

{% highlight c %}

vim /var/lib/pgsql/9.4/data/postgresql.conf
Find the following line:
\#listen_addresses = 'localhost'
Uncomment the line and change localhost to *:
listen_addresses = '*'

{% endhighlight %}

**If the Mattermost server is on a separate machine, modify the file pg_hbe.conf to allow the Mattermost server to communicate with the database.**

- vim /var/lib/pgsql/9.4/data/pg_hba.conf
- update pg_hba.conf config

**restart pgsql**

{% highlight c %}

**systemctl restart  postgresql-9.4.service**

{% endhighlight %}

**Verify that you can connect with the user mmuser**

{% highlight c %}

psql --host=localhost --dbname=mattermost --username=mmuser --password
The PostgreSQL interactive terminal starts. To exit the PostgreSQL interactive 	terminal, type \q and press Enter.

{% endhighlight %}