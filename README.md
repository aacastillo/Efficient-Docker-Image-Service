### TO DO
- Include the image as part of a daemon set so that it automatically run on all current and new worker nodes.

### Description
We are going to create an image that runs a simple helloworld Golang script, that prints out to the terminal. We will make an innefficient image and and efficient image and show the difference in memory consumption. This is done by using multi-stage builds, which uses multiple FROM directives. Each FROM directive creates a new set of file system layers. However the effective comes from the ability for us to selectively copy the files from previous stages/file systems that have been overwritten. 
We will then create a service with three replicas for the efficient image.
<br>

*The takeaway is that the inefficient copy takes up 700+ MB of space while the efficient copy takes 7+ MB of space.*

### Prerequisites
- you should have a swarm manager and two worker nodes set up (check this [link](https://github.com/aacastillo/DockerSwarmSetUp) for help creating the cluster)
- Sign into the swarm manager

### Installation
1. Create project directories
```
cd ~/
mkdir efficient
mkdir inefficient
cd inefficient
```

2. Create source code file
```
vi helloworld.go
```
```GOLANG
package main
import "fmt"
func main() {
  fmt.Println("hello world")
}
```

3. Create the dockerfile
```
vi Dockerfile
```
```
FROM golang:1.12.4
WORKDIR /helloworld
COPY helloworld.go .
RUN GOOS=linux go build -a -installsuffix cgo -o helloworld .
CMD ["./helloworld"]
```

4. Build and test the inefficient image
```
docker build -t inefficient .
docker run inefficient
docker image ls
```

5. switch to the efficient directory and copy the files from the inefficient project directory
```
cd ~/efficient
cp ../inefficient/helloworld.go ./
cp ../inefficient/Dockerfile ./
```

6. Change the Dockerfile to include the multi-stage build
```
vi Dockerfile
```
```
FROM golang:1.12.4 AS compiler
WORKDIR /helloworld
COPY helloworld.go .
RUN GOOS=linux go build -a -installsuffix cgo -o helloworld .
FROM alpine:3.9.3
WORKDIR /root
COPY --from=compiler /helloworld/helloworld .
CMD ["./helloworld"]
```

7. Build and test the efficient build
```
docker build -t efficient .
docker run efficient
docker image ls
```

8. Create the Docker service for the image
```
Docker create service --name efficient-service --replicas 3 efficient 
```

