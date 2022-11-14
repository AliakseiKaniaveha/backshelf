# Docker Swarm on localhost through dind (Docker in Docker)
## Brief
Docker Swarm locally for expererimenting etc.
## Subject
host> docker run --privileged --interactive --tty --detach --name swarm-lab-manager docker:dind
host> docker run --privileged --interactive --tty --detach --memory 4g --cpus 2 --name swarm-lab-worker1 docker:dind
host> docker run --privileged --interactive --tty --detach --memory 4g --cpus 2 --name swarm-lab-worker2 docker:dind
host> docker network inspect bridge   // check ips: m: 172.17.0.2, w1: 172.17.0.3, w2: 172.17.0.3
host (new terminal)> docker stats
host (new terminal)> docker exec -it swarm-lab-manager /bin/sh
manager> docker swarm init --advertise-addr 172.17.0.2
host (new terminal)> docker exec -it swarm-lab-worker1 /bin/sh
worker1> docker swarm join --token <token form output> 172.17.0.2:2377
host (new terminal)> docker exec -it swarm-lab-worker2 /bin/sh
worker2> docker swarm join --token <token form output> 172.17.0.2:2377
manager> docker node ls
/// --limit-cpu decimal --limit-memory bytes --reserve-cpu decimal --reserve-memory bytes (1mb == 1048576 512mb == 536870912 1Gb == 1073741824 2 Gb == 2147483648)
manager> docker service create --detach --constraint node.role!=manager --replicas 1 --name eater1 YOUR_DOCKERHUB_NAME/eater tail -f /dev/null
manager> docker service create --detach --constraint node.role!=manager --replicas 1 --name eater1 epamaliakseikaniaveha/eater tail -f /dev/null
manager> docker service create --detach --constraint node.role!=manager --replicas 1 --name eater2 epamaliakseikaniaveha/eater tail -f /dev/null
manager> docker service create --detach --constraint node.role!=manager --replicas 1 --name eater3 epamaliakseikaniaveha/eater tail -f /dev/null
manager> docker service create --detach --constraint node.role!=manager --replicas 1 --name eater4 epamaliakseikaniaveha/eater tail -f /dev/null
///do some experiments
docker service create --reserve-memory 1073741824 --detach --constraint node.role!=manager --replicas 1 --name eater1 epamaliakseikaniaveha/eater tail -f /dev/null
manager> docker service create --reserve-memory 2147483648 --detach --constraint node.role!=manager --replicas 1 --name eater1 epamaliakseikaniaveha/eater tail -f /dev/null
worker> docker container ls
worker> docker exec -it <eater service container id> /bin/sh
worker> docker exec -it f9365c0f8055 /bin/sh
worker> docker exec -it ca533118ce2b /bin/sh
eater> ./eater/eater.sh
///
manager> docker service rm eater1 eater2 eater3 eater4
host> docker rm --force swarm-lab-worker1




docker container ls -a    // list all containers
docker exec -it <container name> /bin/sh   // sh into container
docker start <container name>   // start container if stopped
docker logs <container name> // container logs
docker info

docker service ls   // list services in the swarm
docker service rm <service name>   // remove service
docker node ls // list nodes in the swarm
docker node ps <node>// list tasks running on the node
docker node ps $(docker node ls -q) // list tasks running on all nodes
docker node inspect <node>

docker stats    // monitor CPU and RAM
x free -m // monitor RAM
x top // monitor RAM and CPU

docker exec -it swarm-lab-manager /bin/sh
docker exec -it swarm-lab-worker1 /bin/sh


// top -pid - show data of process by pid
// ps aux - lilst processes
// grep eater  - get only processes with 'eater' token
// awk '{print $2}' - print only the second column of grep output (pid)
// tail -1 - get last line only, as grep command itself listed in the processes list
top -pid $(ps aux | grep eater | awk '{print $2}' | tail -1)


docker build -t eater -f ./src/main/java/eater/Dockerfile .
docker run --interactive --tty --detach --name eater eater
docker exec -it eater /bin/sh
