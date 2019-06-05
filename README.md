The setup involves following steps:- 

# Install Kong

- ## Install the files
 ```
  $ sudo yum update -y
  $ sudo yum install -y wget
  $ wget https://bintray.com/kong/kong-rpm/rpm -O bintray-kong-kong-rpm.repo
  $ export major_version=`grep -oE '[0-9]+\.[0-9]+' /etc/redhat-release | cut -d "." -f1`
  $ sed -i -e 's/baseurl.*/&\/centos\/'$major_version''/ bintray-kong-kong-rpm.repo
  $ sudo mv bintray-kong-kong-rpm.repo /etc/yum.repos.d/
  $ sudo yum update -y
  $ sudo yum install -y kong
 ```
 
 - ## Setup the database 
 
    When using a database, you will use the kong.conf configuration file for setting Kong’s configuration properties at start-up and the database as storage of all configured entities, such as the Routes and Services to which Kong proxies. ( When not using a database, you will use kong.conf its configuration properties and a kong.yml file for specifying the entities as a declarative configuration.)
	
	To setup the DB using PostgreSQL run the following commands in the postgres
	
	```
	CREATE USER kong; CREATE DATABASE kong OWNER kong;
	```
	
	Then run the Kong migrations
	
	```
	 $ kong migrations bootstrap [-c /path/to/kong.conf]
	 ```
	 
- ## Start Kong

	```
	$ kong start [-c /path/to/kong.conf]
	```
- ## Use Kong
	Kong should now be running at `http://localhost:8000`
	
# Konga UI Installation
To install and run Konga UI(UI for Kong), execute the following commands
```
$ git clone https://github.com/pantsel/konga.git
$ cd konga
$ npm i
```
	
