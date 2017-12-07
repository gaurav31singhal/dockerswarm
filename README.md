# dockerswarm
Docker swarm a native orchestra-tor and container management solution
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
