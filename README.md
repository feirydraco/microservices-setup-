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

# Kubeless and Minikube installation

- ## Install minikube 
	Execute the following commands
	```
	$ curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl && chmod +x kubectl && sudo mv kubectl /usr/local/bin
	$ curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube && sudo mv 	minikube /usr/local/bin/	
	$ minikube start
	```

- ## Installing and deploying Kubeless
	Execute the following commands
	```
	$ export RELEASE=$(curl -s https://api.github.com/repos/kubeless/kubeless/releases/latest | grep tag_name | cut -d '"' -f 4)
	$ kubectl create ns kubeless
	$ kubectl create -f https://github.com/kubeless/kubeless/releases/download/$RELEASE/kubeless-$RELEASE.yaml
	```
	Now, ensure kubeless is running by executing
	```
	$ kubectl get pods -n kubeless
	NAME                                           READY     STATUS    RESTARTS   AGE
	kubeless-controller-manager-567dcb6c48-ssx8x   1/1       Running   0          1h

	$ kubectl get deployment -n kubeless
	NAME                          DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
	kubeless-controller-manager   1         1         1            1           1h

	$ kubectl get customresourcedefinition
	NAME                          AGE
	cronjobtriggers.kubeless.io   1h
	functions.kubeless.io         1h
	httptriggers.kubeless.io      1h
	```
	
