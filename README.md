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
- To install and run Konga UI(UI for Kong), execute the following commands
	```
	$ git clone https://github.com/pantsel/konga.git
	$ cd konga
	$ npm i
	```
	Konga shall now be running on http://localhost:1337

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
# Kubeless UI Installation

- ## Install latest version of Kubeless UI

	```
	kubectl create -f https://raw.githubusercontent.com/kubeless/kubeless-ui/master/k8s.yaml
	```

- ## Kubeless UI on Minikube

To see the UI come up in minikube

```bash
minikube service ui -n kubeless
```
Alternatively,
```bash
kubectl get svc ui -n kubeless
NAME      CLUSTER-IP   EXTERNAL-IP   PORT(S)          AGE
ui        10.0.0.151   <pending>     3000:31172/TCP   12m
```
and access the UI at
```bash
$(minikube ip):31172
```

# Kong Ingress on Minikube
- Start `minikube`

    ```bash
    minikube start
    ```

    It will take a few minutes to get all resources provisioned.

    ```bash
    kubectl get nodes
    ```

- ## Deploy Kong Ingress Controller

	Deploy Kong Ingress Controller using `kubectl`:

    ```bash
    curl https://raw.githubusercontent.com/Kong/kubernetes-ingress-controller/master/deploy/single/all-in-one-postgres.yaml \
      | kubectl create -f -
    ```

    This command creates:

    ```bash

    namespace "kong" created
    customresourcedefinition "kongplugins.configuration.konghq.com" created
    customresourcedefinition "kongconsumers.configuration.konghq.com" created
    customresourcedefinition "kongcredentials.configuration.konghq.com" created
    service "postgres" created
    statefulset "postgres" created
    serviceaccount "kong-serviceaccount" created
    clusterrole "kong-ingress-clusterrole" created
    role "kong-ingress-role" created
    rolebinding "kong-ingress-role-nisa-binding" created
    clusterrolebinding "kong-ingress-clusterrole-nisa-binding" created
    service "kong-ingress-controller" created
    deployment "kong-ingress-controller" created
    service "kong-proxy" created
    deployment "kong" created
    ```

    *Note:* this process could take up to five minutes the first time

- ## Setup environment variables

	Setup shell variables:

    ```bash
    export KONG_ADMIN_PORT=$(minikube service -n kong kong-ingress-controller --url --format "{{ .Port }}")
    export KONG_ADMIN_IP=$(minikube service   -n kong kong-ingress-controller --url --format "{{ .IP }}")

    export PROXY_IP=$(minikube   service -n kong kong-proxy --url --format "{{ .IP }}" | head -1)
    export HTTP_PORT=$(minikube  service -n kong kong-proxy --url --format "{{ .Port }}" | head -1)
    export HTTPS_PORT=$(minikube service -n kong kong-proxy --url --format "{{ .Port }}" | tail -1)
    ```
	The Kong Ingress Controller will be running on `${PROXY_IP}:${HTTP_PORT}`
	
# Deployement of an example function
The following steps show how to create a sample python function and deploy it on kubeless. It also shows how to add and access it using Kong.

1. Create a sample function you need to be deployed. Let's take an example `test.py`.

	``` 
	def hello(event, context):
  	print event
  	return event['data']
	```
	
	Functions in Kubeless have the same format regardless of the language of the function or the event source. In general, every function:

	- Receives an object event as their first parameter. This parameter includes all the information regarding the event source. In particular, the key 'data' should contain the body of the function request.
	- Receives a second object context with general information about the function.
	- Returns a string/object that will be used as response for the caller.
	
	
2. Create a new function using: 
	```
	$ kubeless function deploy hello --runtime python2.7 \
                                --from-file test.py \
                                --handler test.hello
	INFO[0000] Deploying function...
	INFO[0000] Function hello submitted for deployment
	INFO[0000] Check the deployment status executing 'kubeless function ls hello'
	```
	
	
	You will see the function custom resource created:
	
	```
	$ kubectl get functions
	NAME         AGE
	hello        1h


	$ kubeless function ls
	NAME            NAMESPACE   HANDLER       RUNTIME   DEPENDENCIES    STATUS
	hello           default     helloget.foo  python2.7                 1/1 READY
	```
	
	
	You can then call the function with:
	
	```
	$ kubeless function call hello --data 'Hello world!'
	Hello world!
	```
	
	
3. In order to expose a function, it is necessary to create a HTTP Trigger object. We will create a http trigger to get-python function:
	
	```
	$ kubeless trigger http create get-python --function-name get-python --gateway kong ---hostname "test.com"
	```
	
	This command will create an ingress object. We can see it with kubectl:
	
	```
	$ kubectl get ing
	NAME           HOSTS                              ADDRESS          PORTS     AGE
	get-python    get-python.192.168.99.100.nip.io    192.168.99.100   80        59s
	```
	
	You can test the created HTTP trigger with the following command(The ip might be different):

	```
	$ curl --data '{"Another": "Echo"}' --header "Host: test.com" --header "Content-Type:application/json" 192.168.99.100
	
	{"Another": "Echo"}
	```
