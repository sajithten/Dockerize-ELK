# Dockerize-ELK
In order to see some logs, we’ll need to deploy an application into our cluster. To keep things simple, we’ll run a basic busybox container with a command to push out one log message a second. This will require some YAML,
* busybox.yaml.
```
apiVersion: v1
kind: Pod
metadata:
 name: counter
spec:
 containers:
 - name: count
   image: busybox
   args: [/bin/sh, -c,
           'i=0; while true; do echo "$i: Hello"; i=$((i+1)); sleep 1; done']
```
Then, run the following command to deploy this container into your cluster. If you wish to deploy into a specific namespace, be sure to specify it in your command.

    kubectl apply -f busybox.yaml 
    
This should deploy almost instantly into your cluster. Reading the logs is then simple:

    kubectl logs counter
You should see output that looks something like this:
```
1: Hello
2: Hello
3: Hello
4: Hello
5: Hello
6: Hello
7: Hello
8: Hello
9: Hello
10: Hello
11: Hello
```
To see these logs in real-time, a simple switch can be applied to your previous command:

    kubectl logs counter -f
    
# Install docker
```
sudo apt-get update -y
sudo mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update -y
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
sudo systemctl enable docker.service
sudo systemctl start docker
```
### docker-compose.yaml
```
version: '3'
 
services:
 elasticsearch:
   image: docker.elastic.co/elasticsearch/elasticsearch:7.6.2
   environment:
     - cluster.name=docker-cluster
     - discovery.type=single-node
     - bootstrap.memory_lock=true
     - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
   ulimits:
     memlock:
       soft: -1
       hard: -1
   ports:
     - "9200:9200"
 kibana:
   image: docker.elastic.co/kibana/kibana:7.6.2
   ports:
     - "5601:5601"
 ```
# To run:
    docker-compose up
    
If you’re using Minikube with this setup (which is likely if Elasticsearch is running locally), you’ll need to know the bound host IP that minikube uses. To find this, run the following command:

    minikube ssh "route -n | grep ^0.0.0.0 | awk '{ print \$2 }'"
