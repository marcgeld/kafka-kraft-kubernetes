# kafka-kraft-kubernetes

KRaft Kafka on a Microk8s-based Kubernetes cluster.

## Docker

### Build
`docker build -t marcgeld/kafka .`

### Run
`docker run -it marcgeld/kafka`

## Docker on Apple silicon (...or on other ARM environment)
Build an Amd64 version and run on an ARM environment.

### Build
`docker build --platform linux/amd64 -t marcgeld/kafka .`

### Run
`docker run --platform linux/amd64 marcgeld/kafka`


## Docker commands…

`-d, --detach `     Run container in background and print container ID <br>
`-i, --interactive` Keep STDIN open even if not attached <br>
`-t, --tty`         Allocate a pseudo-TTY <br>

__`Ctrl + p`, `Ctrl + q` will turn interactive mode into daemon mode__


#### Docker processes
`docker ps`

#### Docker connect
`docker exec -it \<container-id\> /bin/bash`

#### Docker logs
`docker logs --follow \<container-id\>`

#### Docker copy
`docker cp \<container-id\>:<filepath> .`

### References
- [https://microk8s.io](https://microk8s.io) MicroK8s is the simplest production-grade conformant K8s.
- [docker](https://www.docker.com) Docker provides a suite of development tools, services, trusted content, and automations
- [kubernetes](https://kubernetes.io) Kubernetes, also known as K8s, is an open-source system for automating deployment, scaling, and management of containerized applications.
