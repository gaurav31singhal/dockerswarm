# Dockerswarm
 - Docker swarm a native orchestra-tor and container management solution
 - A swarm consists of multiple Docker hosts runs in swarm mode 
 - Managers to manage membership and delegation 
 - Workers run swarm services
 - Task is a running container a branch of swarm service and managed by a swarm manager
 - On the fly modification of service’s configuration with recycle
 
## NODES 
A node is an instance of the Docker engine participating in the swarm.

### Manager 
- The manager node schedule containers as work called tasks to worker nodes
- Manager nodes also perform the orchestration and cluster management functions
- Maintain desired state of the swarm cluster 
- Manager nodes elect a single leader to conduct orchestration tasks
#### Components

- API server : Accepst command from CLI and create rest objects 
- Orchestrator 
- Allocator : Allocate IP address to tasks 
- Dispatcher : Assign tasks to nodes
- Scheduler : Instruct worker to run a task 



### Workers 
- Executes tasks allocated by manager 
- Worker node notifies manager of the current state of tasks helps maintaining desired state of each worker

#### Components 
- Worker : Connect to dispatcher for assigned tasks
- Executor : Execute assigned tasks 
- Here your containers runs 

### Services
- A service is the definition of the tasks to execute on the manager or worker nodes. 
- It is the central structure of the swarm system and the primary root of user interaction with the swarm
- Service specify which container image to use and which commands to execute inside running containers
- Its a complete defination of your application running inside a container 
- Determines the port where the swarm will make the service available outside the swarm
- an overlay network for the service to connect to other services in the swarm
- CPU and memory limits and reservations
- A rolling update policy
- Replicas of the image to run in the swarm

*Replicated services model, the swarm manager distributes a specific number of replica tasks among the nodes based upon the scale you set in the desired state.

*global services, the swarm runs one task for the service on every available node in the cluster. AV/Monitoring Agents 


### Tasks
- A task carries a Docker container and the commands to run inside the container. 
- It is the atomic scheduling unit of swarm. 
- Manager nodes assign tasks to worker nodes according to the number of replicas set in the service scale. 
- Tasks are execution units that run once to completion. When a task stops, it isn’t executed again, but a new task may take its place.
- Once a task is assigned to a node, it cannot move to another node. It can only run on the assigned node or fail.

### Load balancing
- The swarm manager uses ingress load balancing to expose the services you want to make available externally to the swarm. 
- The swarm manager can automatically assign the service a PublishedPort or you can configure a PublishedPort for the service. - If you do not specify a port, the swarm manager assigns the service a port in the 30000-32767 range
- External components, such as cloud load balancers, can access the service on the PublishedPort of any node in the cluster 
- All nodes in the swarm route ingress connections to a running task instance
- Swarm mode has an internal DNS component that automatically assigns each service in the swarm a DNS entry. 
- The swarm manager uses internal load balancing to distribute requests among services within the cluster via DNS

# Features of Docker swarm
 - Cluster management integrated with Docker Engine
 - kinds of nodes, managers and workers
 - Deploy applications as services and a Declarative service model
 - Elasticity scaling
 - Desired state reconciliation: self healing autu recovery
 - Multi-host networking: Overlay networking 
 - Service discovery/embedded DNS: Swarm manager nodes assign each service in the swarm a unique DNS name and load balances
 - Can introduce external load balancer
 - TLS node to node communication 
 - Rolling updates






## Swarm Networking
A Docker swarm generates two different kinds of traffic:

- Control and management traffic (encrypted): This includes swarm management messages, such as requests to join or leave the swarm. 
- Application/Container traffic: This includes container traffic and traffic to and from external clients.

Following three network concepts are important to swarm services:

- Overlay networks manage communications among the Docker daemons participating in the swarm. Overlay networks can be created for user-     defined networks for standalone containers. You can attach a service to one or more existing overlay networks as well, to  enable         service-to-service communication. 

- Ingress network is a special overlay network that facilitates load balancing among a service’s nodes. When any swarm node receives a       request on a published port, it hands that request off to a module called IPVS. IPVS keeps track of all the IP addresses participating     in that service, selects one of them, and routes the request to it, over the ingress network.

      - The ingress network is created automatically when you initialize or join a swarm.

- docker_gwbridge is a bridge network that connects the overlay networks (including the ingress network) to an individual Docker daemon’s   physical network. By default, each container a service is running is connected to its local Docker daemon host’s docker_gwbridge           network.

      - The docker_gwbridge network is created automatically when you initialize or join a swarm.
      
      
      
 ## Build Swarm cluster 1 Managers 2 Workers 
 
 ### Manager
 ```
 $ sudo docker-machine ssh <M1>
 $ docker swarm init --advertise-addr <M1-IP>
 $ docker info
 $ docker node ls 
 ```
 To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-0usvgwrlquvprt3vcs2c4uvrkhub7rfgxymrxy834ebrcnjmk9-6f7uj471ypbm365ygoksxzza1 192.168.99.100:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
 
 ### Add Node 
 #### Worker1
 ```
 $ sudo docker-machine ssh <W1>
 $ docker swarm join --token  SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
  192.168.99.100:2377
 ```
 *some how missed the token you can regenerate from M1 
 ```
 $ docker swarm join-token worker
 ```
 *Invalidate OLD and re-generate new 
 ```
 $ sudo docker swarm join-token  --rotate worker
 ```
 #### Worker2 
 Same as above
 
 ### M1
 ```
 $ sudo docker node ls
 ```
 ### Deploy Service on M1 
 ``` 
 $ sudo docker service create --replicas 1 --name pingservice alpine ping docker.com
 $ sudo docker service ls
 ```
 
 ### Inspect a service
 ```
 $ docker service inspect --pretty pingservice
 ```
 *without pretty you will get a JSON format output 
 ```
 $ sudo docker service ps pingservice
 ```
 *Verify which nodes are running the service
 
 - Manager nodes can also run services if required
 - DESIRED STATE and LAST STATE of the service task to verify tasks are running according to the service definition
 - Goto to Worker/Manager node wherein your service is running , Execute $ docker ps , to see running container
 
 ### Scale your service 
 ### Machine <M1> use
 
 ```
 $ sudo docker service scale pingservice=5
 $ sudo docker service ps 
 ```
 - Goto to Worker/Manager node wherein your service is running , Execute $ docker ps , to see running containers
 
 ### Deleting Service 
 ```
 $ sudo docker service rm pingservice
 $ sudo docker service ps 
 ```
 *Containers may take some time in cleaning up 
 
## Rolling updates 
Now we deploy a service based on the Redis 3.0.6 container image. Then upgrade the service to use the Redis 3.0.7 container image using rolling updates

```
$ sudo docker service create --replicas 3 --name redis --update-delay 10s redis:3.0.6
$ sudo docker service inspect --pretty redis
$ sudo docker service update --image redis:3.0.7 redis redis
$ sudo docker service inspect --pretty redis

 
```
- update-delay flag configures the time delay between updates to a service tasks
- You can describe the time T as a combination of the number of seconds Ts, minutes Tm, or hours Th

The scheduler applies rolling updates as follows by default:

- Stop the first task.
- Schedule update for the stopped task.
- Start the container for the updated task.
- If the update to a task returns RUNNING, wait for the specified delay period then start the next task.
- If, at any time during the update, a task returns FAILED, pause the update.

### Rolling update failed/Paused state
```
$ sudo docker service inspect --pretty redis
$ sudo docker service update redis
$ sudo docker service ps
```
*incase inspect output shows failed/paused state


## Nodes Maintenance 
- Drain a node on the swarm for putting it inside a maintenance time window 
- DRAIN availability prevents a node from receiving new tasks from the swarm manager.
- Manager stops tasks running on the node and launches replica tasks on a node with ACTIVE availability

#### On M1
```
$ sudo docker node ls
$ sudo docker service create --replicas 3 --name redis --update-delay 10s redis:3.0.6
$ sudo docker service ps redis
$ sudo docker node update --availability drain worker1
$ sudo docker node inspect --pretty worker1
```
*output shows Availability:		Drain
```
$ sudo docker service ps redis
```
#### Reactivatation Post Maintenance window 

```
$ sudo docker node update --availability active worker1
$ sudo docker node inspect --pretty worker1
$ sudo docker service ps
```

### Routing 
- The routing mesh enables each node in the swarm to accept connections on published ports for any service running in the        swarm, even if there’s no task running on the node. 
- The routing mesh routes all incoming requests to published ports on available nodes to an active container
- In order to use the ingress network in the swarm, you need to have the following ports open between the swarm nodes before    you enable swarm mode:
 
   - Port 7946 TCP/UDP for container network discovery
   - Port 4789 UDP for the container ingress network
   
### Publish a port for a service for external access 
```
docker service create \
  --name my-web \
  --publish target=8080,port=80 \
  --replicas 2 \
  nginx
```

### On fly 
```
$ docker service update \
  --publish-add target=<PUBLISHED-PORT>,port=<CONTAINER-PORT> \
  <SERVICE>
```
### Inspect service 
```
$ docker service inspect --format="{{json .Endpoint.Spec.Ports}}" my-web
```

### Publish a port for TCP only or UDP only
```
$ sudo docker service create --name dns-cache -p 53:53 dns-cache
  
```
### UDP
```
$ sudo docker service create --name dns-cache -p 53:53/udp dns-cache
```

### TCP/UDP Both 

```
$ sudo docker service create --name dns-cache -p 53:53 -p 53:53/udp dns-cache
```
### BYPASS DEFAULT SWARM ROUTING MESH
- Host Mode when you access the bound port on a given node, always accessing the instance of the service running on that node
- 5 nodes but run 10 replicas container you cannot specify a static target port. Either allow Docker to assign a random high-numbered port (by leaving off the target), or ensure that only a single instance of the service runs on a given node, by using a global service rather than a replicated one, or by using placement constraints.
- To bypass the routing mesh, you must use the long --publish service and set mode to host. If you omit the mode key or set it to ingress, the routing mesh is used.

```
$ sudo docker service create --name dns-cache \
  --publish target=53,port=53,protocol=udp,mode=host \
  --mode global dns-cache
```
## NODES WORKING
### M- Nodes
- Maintaining cluster state (Raft) 
- Scheduling services
- Assiging tasks 

### Fault tolerance
- A three-manager swarm tolerates a maximum loss of one manager.
- A five-manager swarm tolerates a maximum simultaneous loss of two manager nodes.
- An N manager cluster will tolerate the loss of at most (N-1)/2 managers.
- Docker recommends a maximum of seven manager nodes for a swarm.

### W - nodes are also 

- sole purpose is to execute containers
- Worker nodes don’t participate in the Raft distributed state, make scheduling decisions

### Change Roles 
- Drain : By default tasks cn be schdule on Manager , To restrict it make it as a drain node.
- Promote : Promoting worker as Manager , In case of Manager maintenance.
```
$ sudo docker node promote <node name>
```

### Pending services

- State of service 
- All nodes paused or drained 
- Resources not available 
- Constraints of placement 

### Replicated and global services
#### There are two types of service deployments, replicated and global.

- For a replicated service, you specify the number of identical tasks you want to run. For example, you decide to deploy an   HTTP service with three replicas, each serving the same content.

- A global service is a service that runs one task on every node. There is no pre-specified number of tasks. Each time you add a node to the swarm, the orchestrator creates a task and the scheduler assigns the task to the new node. Good candidates for global services are monitoring agents, an anti-virus scanners or other types of containers that you want to run on every node in the swarm.

 ### Task workflow & states
- Create a service by using docker service create or the UCP web UI or CLI.
- The request goes to a Docker manager node.
- The Docker manager node schedules the service to run on particular nodes.
- Each service can start multiple tasks.
- Each task has a life cycle, with states like NEW, PENDING, and COMPLETE.

####  States

- NEW	The task was initialized.
- PENDING	Resources for the task were allocated.
- ASSIGNED	Docker assigned the task to nodes.
- ACCEPTED	The task was accepted by a worker node. If a worker node rejects the task, the state changes to REJECTED.
- PREPARING	Docker is preparing the task.
- STARTING	Docker is starting the task.
- RUNNING	The task is executing.
- COMPLETE	The task exited without an error code.
- FAILED	The task exited with an error code.
- SHUTDOWN	Docker requested the task to shut down.
- REJECTED	The worker node rejected the task.
- ORPHANED	The node was down for too long.

#### View state
Run docker service ps <service-name> to get the state of a task. The CURRENT STATE field shows the task’s state and how long it’s been there.
```
$ sudo docker service ps
```

### Manage Nodes in Swarm
#### List nodes
```
$ sudo docker node ls
```

##### The AVAILABILITY column shows whether or not the scheduler can assign tasks to the node
- Active means that the scheduler can assign tasks to the node.
- Pause means the scheduler doesn’t assign new tasks to the node, but existing tasks remain running.
- Drain means the scheduler doesn’t assign new tasks to the node. The scheduler shuts down any existing tasks and schedules them on an available node.

##### The MANAGER STATUS column shows node participation in the Raft consensus:

- No value indicates a worker node that does not participate in swarm management.
- Leader means the node is the primary manager node that makes all swarm management and orchestration decisions for the swarm.
- Reachable means the node is a manager node participating in the Raft consensus quorum. If the leader node becomes - unavailable, the node is eligible for election as the new leader.
- Unavailable means the node is a manager that is not able to communicate with other managers. If a manager node becomes unavailable, you should either join a new manager node to the swarm or promote a worker node to be a manager.

### Inspect an individual node

```
$ docker node inspect self --pretty
```
### Apply labels

```
$ docker node update --label-add foo --label-add bar=baz node-1 node-1
```

### Promote & Demote mode
```
$ sudo docker promote w2
$ sudo docker demote w2
$ docker node update --role manager w2
$ docker node update --role worker w2 
```

### Leave & Remove

#### On Worker 
```
$ sudo docker swarm leave
```
#### On Manager
```
$ sudo docker node rm w1
```
### Services 
##### Preference
```
docker service create \
  --replicas 9 \
  --name redis_2 \
  --placement-pref 'spread=node.labels.datacenter' \
  redis:3.0.6
```

#### Constraints 
```
docker service create \
  --name my-nginx \
  --global \
  --constraint region==east \
  --constraint type!=devel \
  nginx
```

#### update behaviour 
```
$ docker service create \
  --replicas 10 \
  --name my_web \
  --update-delay 10s \
  --update-parallelism 2 \
  --update-failure-action continue \
  alpine

```

#### Roll back previous version
```
docker service update \
  --rollback \
  --update-delay 0s
  my_web
```
#### Volumes and bind access

```
docker service create \
  --mount src=<VOLUME-NAME>,dst=<CONTAINER-PATH> \
  --name myservice \
  <IMAGE>
```

```
$ docker service create \
  --mount type=bind,src=<HOST-PATH>,dst=<CONTAINER-PATH> \
  --name myservice \
  <IMAGE>
```
## Docker Configs 

- service configs,  allow you to store non-sensitive information, such as configuration files, outside a service’s image or running containers. 
- This allows you to keep your images as generic as possible, without the need to bind-mount configuration files into the containers or use environment variables.

- Configs operate in a similar way to secrets, except that they are not encrypted at rest and are mounted directly into the container’s filesystem without the use of RAM disks. 
- Configs can be added or removed from a service at any time, and services can share a config


```
$ docker config create homepage index.html
$ docker config ls 
$ docker service create

    --name web
    --publish target=8000,port=8080
    --config src=homepace,target="/var/www/html/index.html"
    webserver:v1  
 $ docker config rm homepage
```

## Secrets 

a secret is a blob of data, such as a password, SSH private key, SSL certificate, or another piece of data that should not be transmitted over a network or stored unencrypted in a Dockerfile or in your application’s source code. In Docker 1.13 and higher, you can use Docker secrets to centrally manage this data and securely transmit it to only those containers that need access to it. Secrets are encrypted during transit and at rest in a Docker swarm. A given secret is only accessible to those services which have been granted explicit access to it, and only while those service tasks are running.

You can use secrets to manage any sensitive data which a container needs at runtime but you don’t want to store in the image or in source control, such as:

- Usernames and passwords
- TLS certificates and keys
- SSH keys
- Other important data such as the name of a database or internal server
- Generic strings or binary content (up to 500 kb in size)

## Commands

- docker secret create
- docker secret inspect
- docker secret ls
- docker secret rm
- --secret flag for docker service create
- --secret-add and --secret-rm flags for docker service update

```
$ docker secret create homepage index.html
$ $ docker service create

    --name web
    --publish target=8000,port=8080
    --secret src=homepace,target="/var/www/html/index.html"
    webserver:v1  
 $ docker secret rm homepage
```
 
 
## LOCK Swarm 
In Docker 1.13 and higher, the Raft logs used by swarm managers are encrypted on disk by default. This at-rest encryption protects your service’s configuration and data from attackers who gain access to the encrypted Raft logs. One of the reasons this feature was introduced was in support of the new Docker secrets feature.

When Docker restarts, both the TLS key used to encrypt communication among swarm nodes, and the key used to encrypt and decrypt Raft logs on disk, are loaded into each manager node’s memory. Docker 1.13 introduces the ability to protect the mutual TLS encryption key and the key used to encrypt and decrypt Raft logs at rest, by allowing you to take ownership of these keys and to require manual unlocking of your managers. This feature is called autolock.

When Docker restarts, you must unlock the swarm first, using a key encryption key generated by Docker when the swarm was locked.

```
$ sudo docker swarm update --autolock=true
$ sudo service docker restart
$ sudo docker service ls
```
##### Unlock 
```
$ sudo docker swarm unlock
```
#### View and autorotate 
```
$ sudo docker swarm unlock-key
$ sudo docker swarm unlock-key --rotate
```

### Monitor Swarm cluster 

$ sudo docker node inspect manager1 --format "{{ .ManagerStatus.Reachability }}"

reachable

To query the status of the node as a worker that accept tasks:


$ docker node inspect manager1 --format "{{ .Status.State }}"

ready

#### force removal node
$ docker node rm --force node9

### Back up the swarm

You can back up the swarm using any manager. Use the following procedure.

- If the swarm has auto-lock enabled, you will need the unlock key in order to restore the swarm from backup. Retrieve the unlock key if necessary and store it in a safe location. If you are unsure, read Lock your swarm to protect its encryption key.

- Stop Docker on the manager before backing up the data, so that no data is being changed during the backup. It is possible to take a backup while the manager is running (a “hot” backup), but this is not recommended and your results will be less predictable when restoring. While the manager is down, other nodes will continue generating swarm data that will not be part of this backup.

- Back up the entire /var/lib/docker/swarm directory.


- Restart the manager.

## Restore 


- Recover from disaster
- Restore from a backup
After backing up the swarm as described in Back up the swarm, use the following procedure to restore the data to a new swarm.

Shut down Docker on the target host machine where the swarm will be restored.

Remove the contents of the /var/lib/docker/swarm directory on the new swarm.

Restore the /var/lib/docker/swarm directory with the contents of the backup.

Note: The new node will use the same encryption key for on-disk storage as the old one. It is not possible to change the on-disk storage encryption keys at this time.

In the case of a swarm with auto-lock enabled, the unlock key is also the same as on the old swarm, and the unlock key will be needed to restore.
Start Docker on the new node. Unlock the swarm if necessary. Re-initialize the swarm using the following command, so that this node does not attempt to connect to nodes that were part of the old swarm, and presumably no longer exist.

$ docker swarm init --force-new-cluster

Verify that the state of the swarm is as expected. This may include application-specific tests or simply checking the output of docker service ls to be sure that all expected services are present.

If you use auto-lock, rotate the unlock key.

Add manager and worker nodes to bring your new swarm up to operating capacity.

####  From the node to recover
docker swarm init --force-new-cluster --advertise-addr node01:2377
