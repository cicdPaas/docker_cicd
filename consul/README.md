---------------------------
Consul Cluster of 3 docker containers
---------------------------

1) Launch 3 ubuntu docker containers

2) Install consul on each of them:

   wget https://releases.hashicorp.com/consul/0.6.3/consul_0.6.3_linux_amd64.zip
   unzip consul_0.6.3_linux_amd64.zip
   mv consul /usr/local/bin/

3) Create consul config directory where we'll keep configuration files:
   mkdir /etc/consul.d

4) Get IP of your docker containers and note it: docker inspect <container-name> | grep IP

5) Create 1st config file config.json at /etc/consul.d and paste the following content :
 
{
"datacenter" : "dc1",
"data_dir" : "/tmp/consul",
"log_level" : "INFO",
"node_name" : "<node name>",
"server" : true,
"bootstrap_expect" : 3,
"bind_addr" : "<ip of docker container>"
}

Note: You can also copy the content from config.json file.

6) Then in other containers, create this config file /etc/consul.d/config.json and paste the following content:

{
"datacenter" : "dc1",
"data_dir" : "/tmp/consul",
"log_level" : "INFO",
"node_name" : "<name of node>",
"server" : true,
"bootstrap_expect" : 3,
"bind_addr" : "<ip of container>",
"start_join" : ["<ip of first server>"]
}

Note: You can also copy the content from config1.json file.

7) Now on the 1st server, lets start consual agent:
   consul agent -config-dir=/etc/consul.d &

8) After that, issue above command on rest of containers

Note: Now you can see that all servers are in sync and they will elect a leader.

9) Issue below command to see cluster members:
   consul members

Result:::: Now we have a HA consul cluster.

------------------------------------
Clients with service
------------------------------------

Now we will launch another ubuntu docker container in which we'll launch a consul agent (as we did above) and run it in client mode

1) Create config.json in /etc/consul.d:

{
"datacenter" : "dc1",
"data_dir" : "/tmp/consul",
"log_level" : "INFO",
"node_name" : "<node-name>",
"server" : false,
"bind_addr" : "<container ip>",
"start_join" : ["<first server ip>"]
}

Note: You can also copy the content from agent-config.json file.

2) Now install any of your favorite service, for this example, I am using mysql-server

:::::: Define a service ::::::

3) We'll create a config file for defining a service, name it mysql.json and create it in /etc/consul.d:

{"service": {"name": "mysql", "tags": ["mysql"], "port": 3306}}

4) Now start consul agent:

   consul agent -config-dir=/etc/consul.d &

Note: You will see a message saying mysql service synced

5) Also it will join servers, and if you issue this command:
   consul members
 Note: you must see 4 members

Info: By default consul services will be resolved by this type of url: <service name>.consul.service. So here the service name will be: mysql.service.consul

7) Here, Consul runs its own dns server to resolve services at port 8600. So, we'll try to resolve this service by using dig:

   dig @127.0.0.1 -p 8600 mysql2.service.consul

8) If you do not have dig, install it using:
   apt-get install dnsutils


----------
Dnsmasq
----------

Now we'll setup a dns forwarder on client so that all requests ending with consul could be forwarded to consul's dns server for resolution

1) We'll use dnsmasq forwarder, Install it using:
   apt-get install dnsmasq


2) Configure dnsmasq to forward all request from default dns port 53 to 8600 if service name ends with consul.

3) For this first open dnsmasq configuration file /etc/dnsmasq.conf and at last place this line:
   user=root

4) Now we'll create a config file which will do the required forwarding:

5) Create a file: /etc/dnsmasq.d/10-consul and insert below line inside it:

   server=/consul/127.0.0.1#8600

6) start dnsmasq:
   service dnsmasq start

Execute the command: dig @127.0.0.1 mysql.service.consul


7) Make an additional entry in /etc/resolv.conf

nameserver 127.0.0.1

8) Save and exit

9) Now, dig again without using port..

dig mysql.service.consul

