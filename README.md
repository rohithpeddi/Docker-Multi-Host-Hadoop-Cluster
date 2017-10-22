# Docker Multi Host Hadoop Cluster
Deploying a hadoop cluster in docker containers running on multiple hosts. Master and slave containers on different hosts are connected via an overlay network.

## Creating an Overlay Network
Overlay network can be created in multiple ways.


#### [Using swarm](https://www.youtube.com/watch?v=nGSNULpHHZc)

   * Run `docker swarm init` on master node. Use the token generated to join slaves to the swarm
   * Run `docker network create -d overlay --subnet={subnet range} --attachable {network name}`
   * Run a sample `docker service` just to make sure overlay newtork is discoverable across hosts
    
#### [Using KV store](https://luppeng.wordpress.com/2016/05/03/setting-up-an-overlay-network-on-docker-without-swarm/)

   * One Host it used to run Consul and store Key Values
   * Other Hosts docker daemon is run in TCP mode to connect to this Consul
   * Create an attachable overlay network

## Building Hadoop Docker images on all hosts

  * `git clone https://github.com/cpclass/Docker-Multi-Host-Hadoop-Cluster.git`  
  * `cd Docker-Multi-Host-Hadoop-Cluster`
  
#### hadoop-base image

  * `docker build  -f dockerfiles/hadoop-base-Dockerfile -t hadoop-base .`
  
#### hadoop-master image

  * `docker build  -f dockerfiles/hadoop-master-Dockerfile -t hadoop-master .`
  
#### hadoop-slave image

  * `docker build  -f dockerfiles/hadoop-slave-Dockerfile -t hadoop-slave .`
  
 or pull images from docker hub *(To-Do)*
  
  * `docker pull cpclass/hadoop-master`
  * `docker pull cpclass/hadoop-slave`
 
 
  
 ## Running hadoop cluster
 
 Overlay network was created with subnet `10.0.9.0/24` and name `hadoop-overlay` . 
 
 #### master container on Host 1
 
  * `docker run --rm --name hadoop-master --net hadoop-overlay --ip 10.0.9.22 --hostname hadoop-master -it --add-host hadoop-slave-1:10.0.9.23  --add-host hadoop-slave-2:10.0.9.24 -p 50090:50090 -p 8088:8088 -p 50070:50070  hadoop-master`
  
 #### slave conatiner on Host 2, Host 3, Host 4, ....
 
 * `docker run --rm --name hadoop-slave-1  --net hadoop-overlay --ip 10.0.9.23  --hostname hadoop-slave-1 -it --add-host hadoop-master:10.0.9.22 hadoop-slave`
 
  * `docker run --rm --name hadoop-slave-2  --net hadoop-overlay --ip 10.0.9.24  --hostname hadoop-slave-2 -it --add-host hadoop-master:10.0.9.22 hadoop-slave`
 
  ###### Things to note
  
  * Subnet is specified for the network to give static ip's to the conatiners
  * Specify all slaves appropriately `-- add-host` flag of master container and update in `conf\hadoop\master\slaves` file

  ## docker compose *(To-Do)*
  
  *
  *
  
  ### Detailed Documentation on Dockerfiles 

* `RUN ./create-services-group` creates a services group using `create-group` script with GID and group-name taken from `ACCOUNTS_INFO` file inside assest\base directory. 

* `services` group is created with `sudo` privileges
 
* `RUN ./create-hadoop-user` creates a hadoop user using `create-user` base file and adds it to `services` group. 

* Password less ssh is setup. `ssh_config` conf file restricts strictHostKey checking.

* Hadoop-2.7.4 is downloaded to hadoop user home dir. Hadoop Logs and data dirs are mapped to hadoop user home dir.

* *(TO-DO)* Mount container hadoop user home dir to Host hadoop user home dir (Similar group and users can be created on host using `create-all-users-on-host` file)

* Master gets it's configuration from `conf/hadoop/master/` dir. Number of slaves are specified in `slaves` file.

* Master supervisord_conf runs namenode and yarn. Slave supervisord_conf run datanode only.

* `ENTRYPOINT` starts ssh server and hadoop 






