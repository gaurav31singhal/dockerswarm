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
 $ docker service create --replicas 1 --name helloworld alpine ping docker.com
 $ docker service ls
 ```
 
