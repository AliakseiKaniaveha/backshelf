# Docker Swarm on localhost through dind (Docker in Docker)
## Brief
This is about how to get [Docker Swarm](https://docs.docker.com/engine/swarm/) configured on your local machine in ~10 minutes.<br>
The idea is to use docker containers as swarm nodes.

Might be useful as a playground, [stress testing](https://github.com/AliakseiKaniaveha/eater) sandbox, experinmenting or whatever else.

## Subject

### Localhost
Spin up a manger node. dind is for [Docker in Docker](https://shisho.dev/blog/posts/docker-in-docker/)
> docker run --privileged --interactive --tty --detach --name swarm-lab-manager docker:dind

Spin up two worker nodes.
> docker run --privileged --interactive --tty --detach --memory 4g --cpus 2 --name swarm-lab-worker1 docker:dind<br>
> docker run --privileged --interactive --tty --detach --memory 4g --cpus 2 --name swarm-lab-worker2 docker:dind

Check ip addressess: m: 172.17.0.2, w1: 172.17.0.3, w2: 172.17.0.3
> docker network inspect bridge

Jump into manager
> docker exec -it swarm-lab-manager /bin/sh

### Manager host
Initialize swarm, notice the token
> docker swarm init --advertise-addr 172.17.0.2

### Worker host
Jump into worker (from localhost terminal)
> docker exec -it swarm-lab-worker1 /bin/sh

Add worker node to swarm
> docker swarm join --token <token form output above > 172.17.0.2:2377

Do the same for worker 2.

### Manager host  
Check swarm nodes
>docker node ls

Give it some load (image from https://github.com/AliakseiKaniaveha/eater used in this example). `tail -f /dev/null` is a command which never complete, a trick to keep container up
> docker service create --detach --constraint node.role!=manager --replicas 1 --name eater1 YOUR_DOCKERHUB_NAME/eater tail -f /dev/null<br>
> docker service create --detach --constraint node.role!=manager --replicas 1 --name eater2 YOUR_DOCKERHUB_NAME/eater tail -f /dev/null 

### Worker host
Overview eater1 and eater2 containers
> docker container ls

Jump into an eater container
> docker exec -it <eater service container id> /bin/sh
  
[Have fun](https://github.com/AliakseiKaniaveha/eater/blob/main/README.md#run)

## Couple of useful commands

>docker service rm eater1 eater2 <service name><br>
>docker rm --force swarm-lab-worker1 <node name><br>
  
>docker container ls -a    // list all containers<br>
>docker exec -it <container name> /bin/sh   // sh into container<br>
>docker start <container name>   // start container if stopped<br>
>docker logs <container name> // container logs<br>
>docker info<br>

>docker service ls   // list services in the swarm<br>
>docker service rm <service name>   // remove service<br>
>docker node ls // list nodes in the swarm<br>
>docker node ps <node>// list tasks running on the node<br>
>docker node ps $(docker node ls -q) // list tasks running on all nodes<br>
>docker node inspect <node><br>

>docker stats    // monitor CPU and RAM<br>

>// top -pid - show data of process by pid<br>
>// ps aux - lilst processes<br>
>// grep eater  - get only processes with 'eater' token<br>
>// awk '{print $2}' - print only the second column of grep output (pid)<br>
>// tail -1 - get last line only, as grep command itself listed in the processes list<br>
>top -pid $(ps aux | grep eater | awk '{print $2}' | tail -1)<br>
