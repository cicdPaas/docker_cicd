# Installation

* wget https://releases.hashicorp.com/consul/0.6.3/consul_0.6.3_linux_amd64.zip
* unzip consul_0.6.3_linux_amd64.zip
* mv consul /usr/local/bin/
* Sandbox for consul setup
  * Will do on docker with ubuntu as base image
  * Dockerfile.consul
  * Run consul command on launched container to verify if consul is running fines

# Playing with consul Binary

## Basic parameters of consul agent when running in client mode

* data_dir : This is a mandatory parameter consul agent can't get started with it
* node : default value is hostname
* dc : default value is dc1
* run consul agent --help to know about other important parameters

## Basic parameters of consul agent when running in server mode

* server : this parameter is provided to run consul binary in server mode
* bootstrap : this parameter is prvoded to make a server leader forcefully
* bootstrap_expect : to enforce leader election will happen only when there are minimum n number of nodes
* join : to provide the ip of one of the server of existing Cluster

## Example of running a Cluster

* Launch server1
  * docker run -it --rm --name server1 opstree/consul
* Run consul in server mode with a bootstrap expect value of 3
  * consul agent -data-dir=/tmp/consul -node=server1 -server --bootstrap-expect=3
* Launch server2
  * docker run -it --rm --name server2 opstree/consul
* Run consul in server mode with a bootstrap expect value of 3
  * consul agent -data-dir=/tmp/consul -node=server2 -server --bootstrap-expect=3
* Launch server3
    * docker run -it --rm --name server3 opstree/consul
* Run consul in server mode with a bootstrap expect value of 3 and join it with already running consul server(172.17.0.8)
  * consul agent -data-dir=/tmp/consul -node=server3 -server --bootstrap-expect=3  -join=172.17.0.8
* Join the consul server running in server2 docker container
  * docker exec server2 consul join 172.17.0.8
