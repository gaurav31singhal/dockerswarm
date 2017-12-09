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



### Workers 
- Executes tasks allocated by manager 
- Worker node notifies manager of the current state of tasks helps maintaining desired state of each worker

### Services
- A service is the definition of the tasks to execute on the manager or worker nodes. 
- It is the central structure of the swarm system and the primary root of user interaction with the swarm
- Service specify which container image to use and which commands to execute inside running containers.

*Replicated services model, the swarm manager distributes a specific number of replica tasks among the nodes based upon the scale you set in the desired state.

*global services, the swarm runs one task for the service on every available node in the cluster.

### Tasks
- A task carries a Docker container and the commands to run inside the container. 
- It is the atomic scheduling unit of swarm. 
- Manager nodes assign tasks to worker nodes according to the number of replicas set in the service scale. 
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
 $ docker service inspect --pretty firstservice
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



- 

 
