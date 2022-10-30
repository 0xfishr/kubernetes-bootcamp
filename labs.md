### Lab 1.1. Hello docker

1. Run `docker info` to confirm docker client and server statuses
2. Run `docker run hello-world`


### Lab 1.2. First container interaction

1. Run `ps aux` to review running processes on your host device
2. Run a `busybox` container in interactive mode `docker run -it busybox`
3. Review the container kernel details
4. Review the running processes in the container and note PID
5. Exit the container
6. List running containers
7. List all containers
8. Repeat [2] and exit the container without stopping it
9. List running containers
10. List all containers
11. Delete the containers

</div>

<div id="soln-lab1-2">

<details>
  <summary>lab1.2 solution</summary>
  
```sh
# host terminal
ps aux
docker run --name box1 -it busybox
# container terminal
ps aux
uname -r
cat /proc/version
hostnamectl # not found
cat /etc/*-release # not found
busybox | head
exit
# host terminal
docker ps
docker ps -a
docker run --name box2 -it busybox
# container terminal
ctrl+p ctrl+q
# host terminal
docker ps
docker ps -a
docker stop box2
docker rm box1 box2
```
</details>

> `docker ps` showing STATUS of `Exited (0)` means exit OK, but an Exit STATUS that's not 0 should be investigated `docker logs`
>
> `CTRL+P, CTRL+Q` only works when running a container in interactive mode, see [how to attach/detach containers](https://stackoverflow.com/a/19689048/1235675) for more details

</div>


<div id="lab1-3">

### Lab 1.3. Entering and exiting containers

1. Run a `nginx` container
2. List running containers (use another terminal if stuck)
3. Exit the container
4. List running containers
5. Run another `nginx` container in interactive mode
6. Review container kernel details
7. Review running processes in the container
8. Exit container
9. Run another `nginx` container in detached mode
10. List running containers
11. Connect a shell to the new container interactively
12. View open ports in the container
13. Exit the container
14. List running containers
15. Delete all containers

</div>

<div id="soln-lab1-3">

<details>
<summary>lab1.3 solution</summary>

```sh
# host terminal
docker run --name webserver1 nginx
# host second terminal
docker ps
# host terminal
ctrl+c
docker ps
docker run --name webserver2 -it --rm nginx bash
# container terminal
cat /etc/*-release
ps aux # not found
ls /proc
ls /proc/1 # list processes running on PID 1
cat /proc/1/$PROCESS_NAME
exit
# host terminal
docker run --name webserver3 -d nginx
docker ps
docker exec -it webserver3 bash
# container terminal
netstat -tupln
ss -tulpn
exit
# host terminal
docker ps
docker stop webserver3
docker rm webserver1 webserver2 webserver3
```

</details>

> Containers may not always have `bash` shell, but will usually have the dash shell `sh`

</div>

<div id="lab1-4">

### Lab 1.4. Container arguments

1. Run a `busybox` container with command `sleep 30` as argument, see `sleep --help`
2. List running containers (use another terminal if stuck)
3. Exit container (note that container will auto exit after 30s)
4. Run another `busybox` container in detached mode with command `sleep 300` as argument
5. List running containers
6. Connect to the container to execute commands
7. Exit container
8. List running containers
9. Run another `busybox` container in detached mode, no commands
10. List running containers
11. List all containers
12. Delete all containers

</div>

<div id="soln-lab1-4">

<details>
<summary>lab1.4 solution</summary>

```sh
# host terminal
docker run --name box1 busybox sleep 30
# host second terminal
docker ps
docker stop box1
# host terminal
docker run --name box2 -d busybox sleep 300
docker ps
docker exec -it box2 sh
# container terminal
exit
# host terminal
docker ps
docker run --name box3 -d busybox
docker ps
docker ps -a
docker stop box2
docker rm box1 box2 box3
```

</details>

> The `Entrypoint` of a container is [the init process](https://en.wikipedia.org/wiki/Init) and allows the container to run as an executable. Commands passed to a container are passed to the container's entrypoint process.
>
> Note that `docker` commands after `$IMAGE_NAME` are passed to the container's entrypoint as arguments. \
> ❌ `docker run -it mysql -e MYSQL_PASSWORD=hello` will pass `-e MYSQL_PASSWORD=hello` to the container \
> ✔️ `docker run -it -e MYSQL_PASSWORD=hello mysql`

</div>


<div id="lab1-5">

### Lab 1.5. Container ports and IP

1. Run a `nginx` container with name `webserver`
2. Inspect the container (use `| less` to avoid console clutter) and review the `State` and `NetworkSettings` fields, quit with `q`
3. Visit `http://$CONTAINER_IP_ADDRESS` in your browser (this may not work depending on your envrionment network settings)
4. Run another `nginx` container with name `webserver` and exposed on port 80
5. Visit http://localhost in your browser
6. Delete the containers

</div>

<div id="soln-lab1-5">

<details>
<summary>lab1.5 solution</summary>

```sh
# host terminal
docker run -d --name webserver nginx
docker inspect webserver | grep -A 13 '"State"' | less
docker inspect webserver | grep -A 50 '"NetworkSettings"' | less
curl http://$(docker inspect webserver --format "{{.NetworkSettings.IPAddress}}") | less
docker stop webserver
docker rm webserver
docker run -d --name webserver -p 80:80 nginx
curl localhost | less
docker ps
docker ps -a
docker stop webserver
docker rm webserver
```

</details>

> Always run containers in detached mode to avoid getting stuck in the container `STDOUT`

</div>

<div id="lab1-6">

### Lab 1.6. Container volumes

1. Create an `html/index.html` file with some content
2. Run any webserver containers on port 8080 and mount the `html` folder to the [DocumentRoot](https://serverfault.com/a/588388)
   - option `nginx` DocumentRoot - `/usr/share/nginx/html`
   - option `httpd` DocumentRoot - `/usr/local/apache2/htdocs`
3. Visit http://localhost:8080
4. List running containers
5. List all containers
6. Delete containers

</div>

<div id="soln-lab1-6">

<details>
<summary>lab1.6 solution</summary>

```sh
# host terminal
cd ~
mkdir html
echo "Welcome to Lab 1.6 Container volumes" >> html/index.html
# with nginx
docker run -d --name webserver -v ~/html:/usr/share/nginx/html -p 8080:80 nginx
# with httpd
# docker run -d --name webserver -v ~/html:/usr/local/apache2/htdocs -p 8080:80 httpd
curl localhost:8080
docker ps
docker ps -a
docker stop webserver
docker rm webserver
```

</details>

</div>

<div id="lab1-7">

### Lab 1.7. Container environment variables

1. Run a `mysql` container in detached mode
2. Connect to the container
3. Review the container logs and resolve the output message regarding environment variable
4. Confirm issue resolved by connecting to the container
5. Exit the container
6. List running containers
7. List all containers
8. List all images
9. List all volumes
10. Clean up with [`docker system prune`](https://docs.docker.com/engine/reference/commandline/system_prune/)
11. Check all resources are deleted, containers, images and volumes.

</div>

<div id="soln-lab1-7">

<details>
<summary>lab1.7 solution</summary>

```sh
# host terminal
docker run -d --name db mysql
docker exec -it db bash # error not running
docker logs db
docker rm db
docker run -d --name db -e MYSQL_ROOT_PASSWORD=secret mysql
docker ps
docker ps -a
docker image ls
docker volume ls
docker stop db
docker ps # no containers running
docker system prune --all --volumes
docker image ls
docker volume ls
```

</details>

> You don't always have to run a new container, we have had to do this to apply new configuration. You can restart an existing container `docker ps -a`, if it meets your needs, with `docker start $CONTAINER`

</div>

<div id="lab1-8">

### Lab 1.8. Container registries

Explore [Docker Hub](https://hub.docker.com/) and search for images you've used so far or images/applications you use day-to-day, like databases, environment tools, etc.

> Container images are created with instructions that determine the default container behaviour at runtime. A familiarity with specific images/applications may be required to understand their default behaviours

</div>


<div id="lab2-1">

### Lab 2.1. Working with images

1. List all images (if you've just finished [lab1.7](#lab-17-container-environment-variables), run new container to download an image)
2. Inspect one of the images with `| less` and review the `ContainerConfig` and `Config`
3. View the image history
4. Tag the image with the repository `localhost` and a version
5. List all images
6. View the tagged image history
7. Delete tagged image by ID
8. Lets try that again, delete tagged image by tag

</div>

<div id="soln-lab2-1">

<details>
<summary>lab2.1 solution</summary>

```sh
# host terminal
docker image ls
# using nginx image
docker image inspect nginx | grep -A 40 ContainerConfig | less
docker image inspect nginx | grep -A 40 '"Config"' | less
docker image history nginx
docker tag nginx localhost/nginx:1.1
docker image ls
docker image history localhost/nginx:1.1 # tagging isn't a change
docker image rm $IMAGE_ID # error conflict
docker image rm localhost/nginx:1.1 # deleting removes tag
```

</details>

</div>

<div id="lab2-2">

### Lab 2.2. Custom images

Although, we can also [create an image from a running container using `docker commit`](https://docs.docker.com/engine/reference/commandline/commit/), we will only focus on using a Dockerfile, which is the recommended method.

Build the below Dockerfile with `docker build -t $IMAGE_NAME:$TAG /path/to/Dockerfile/directory`, see `docker build --help

```Dockerfile
# Example Dockerfile
FROM ubuntu
MAINTAINER Piouson
RUN apt-get update && \
    apt-get install -y nmap iproute2 && \
    apt-get clean
ENTRYPOINT ["/usr/bin/nmap"]
CMD ["-sn", "172.17.0.0/16"] # nmap will scan docker network subnet `172.17.0.0/16` for running containers
```

</div>


<div id="lab2-3">

### Lab 2.3. Create image from Dockerfile

See [best practices for writing Dockerfile](https://docs.docker.com/develop/develop-images/dockerfile_best-practices).

```sh
# find a package containing an app (debian-based)
apt-file search --regex <filepath-pattern> # requires `apt-file` installation, see `apt-file --help`
apt-file search --regex ".*/sshd$"
# find a package containing an app, if app already installed (debian-based)
dpkg -S /path/to/file/or/pattern # see `dpkg --help`
dpkg -S */$APP_NAME
# find a package containing an app (rpm-based)
dnf provides /path/to/file/or/pattern
dnf provides */sshd
```

1. Create a Dockerfile based on the following:
   - Base image should be debian-based or rpm-based
   - Should include packages containing `ps` application and network utilities like `ip`, `ss` and `arp`
   - Should run the `nmap` process as the `ENTRYPOINT` with arguments `-sn 172.17.0.0/16`
2. Build the Dockerfile with repository `local` and version `1.0`
3. List images
4. Run separate containers from the image as follows and review behaviour
   - do not specify any modes
   - in interactive mode with a shell
   - in detached mode, then check the logs
5. Edit the Dockerfile to run the same process and arguments but **not** as `ENTRYPOINT`
6. Repeat all three options in [4] and compare the behaviour
7. Clean up

</div>

<div id="soln-lab2-3">

<details>
<summary>lab2.3 solution</summary>

```sh
# run ubuntu container to find debian-based packages
docker run -it --rm ubuntu
# container terminal
apt update
apt install -y apt-file
apt-file update
apt-file search --regex "bin/ip$"
apt-file search --regex "bin/ss$"
apt-file search --regex "bin/arp$"
# found `iproute2` and `net-tools`
exit
```

```sh
# alternatively, run fedora container to find rpm-based packages
docker run -it --rm fedora
# container terminal
dnf provides *bin/ip
dnf provides *bin/ss
dnf provides *bin/arp
# found `iproute` and `net-tools`
exit
```

```sh
# host terminal
mkdir test
nano test/Dockerfile
```

```Dockerfile
# Dockerfile
FROM alpine
RUN apk add --no-cache nmap iproute2 net-tools
ENTRYPOINT ["/usr/bin/nmap"]
CMD ["-sn", "172.17.0.0/16"]
```

```sh
# host terminal
docker build -t local/alpine:1.0 ./test
docker run --name alps1 local/alpine:1.0
docker run --name alps2 -it local/alpine:1.0 sh
docker run --name alps3 -d local/alpine:1.0
docker log alps3
nano test/Dockerfile
```

```Dockerfile
# Dockerfile
FROM alpine
RUN apk add --no-cache nmap iproute2 net-tools
CMD ["/usr/bin/nmap", "-sn", "172.17.0.0/16"]
```

```sh
# host terminal
docker build -t local/alpine:1.1 ./test
docker run --name alps4 local/alpine:1.0
docker run --name alps5 -it local/alpine:1.0 sh
# container terminal
exit
# host terminal
docker run --name alps6 -d local/alpine:1.0
docker log alps6
docker stop alps3 alps5 alps6
docker rm alps1 alps2 alps3 alps4 alps5 alps6
docker image rm local/alpine:1.0 local/alpine:1.1
```

</details>

> In most cases, building an image goes beyond a successful build. Some installed packages require additional steps to run containers successfully

</div>

<div id="lab2-4">

### Lab 2.4. Containerise your application

See the [official language-specific getting started guides](https://docs.docker.com/language/) which includes NodeJS, Python, Java and Go examples.

1. Bootstrap a frontend/backend application project, your choice of language
2. Install all dependencies and test the app works
3. Create a Dockerfile to containerise the project
4. Build the Dockerfile
5. Run a container from the image exposed on port 8080
6. Confirm you can access the app on http://localhost:8080

</div>

<div id="soln-lab2-4">

<details>
<summary>lab2.4 nodejs solution</summary>

```sh
# host terminal
npx express-generator --no-view test-app
cd test-app
yarn
yarn start # visit localhost:3000 if OK, ctrl+c to exit
echo node_modules > .dockerignore
nano Dockerfile
```

```Dockerfile
# Dockerfile
FROM node:alpine
ENV NODE_ENV=production
WORKDIR /app
COPY ["package.json", "yarn.lock", "./"]
RUN yarn --frozen-lockfile --prod
COPY . .
CMD ["node", "bin/www"]
EXPOSE 3000
```

```sh
# host terminal
docker build -t local/app:1.0 .
docker run -d --name app -p 8080:3000 local/app:1.0
curl localhost:8080
docker stop app
docker rm app
docker image rm local/app:1.0
cd ..
rm -rf test-app
```

</details>

</div>

<div id="lab2-5">

### Lab 2.5. Container privileges

1. Display your system's current user
2. Display the current user's UID, GID and group memberships
3. Run a `ubuntu` container interactively, and in the container shell:
   - display the current user
   - display the current user's UID, GID and group memberships
   - list existing user accounts
   - list existing groups
   - create a file called `test-file` and display the file ownership info
   - exit the container
4. Run a new `ubuntu` container interactively with UID 1004, and in the container shell:
   - display the current user
   - display the current user's UID, GID and group memberships
   - exit the container
5. Create a docker image based on `ubuntu` with a non-root user as default user
6. Run a container interactively using the image, and in the container shell:
   - display the current user
   - display the current user's UID, GID and group memberships
   - exit the container
7. Delete created resources

</div>

<div id="lab2-5">

<details>
<summary>lab2.5 solution</summary>

```sh
# host terminal
whoami
id
docker run -it --rm ubuntu
# container terminal
whoami
id
cat /etc/passwd
cat /etc/group
touch test-file
ls -l
ls -ln
exit
# host terminal
docker run -it --rm --user 1004 ubuntu
# container terminal
whoami
id
exit
```

```Dockerfile
# test/Dockerfile
FROM ubuntu
RUN groupadd --gid 1000 piouson && useradd --uid 1000 piouson --gid 1000
USER piouson
```

```sh
# host terminal
docker build -t test-image test/
docker run -it --rm test-image
# container terminal
whoami
id
exit
# host terminal
docker image rm test-image
```

</details>

> If a containerized application can run without privileges, change to a non-root user \
> It is recommended to explicitly specify GID/UID when creating a group/user

</div>


<div id="lab5-1">

### Lab 5.1. Creating Pods

1. Create a Pod with `nginx:alpine` image and confirm creation
2. Review full details of the Pod in YAML form
3. Display details of the Pod in readable form and review the Node, IP, container start date/time and Events
4. List pods but only show resource names
5. Connect a shell to the Pod and confirm an application is exposed
   - By default, Nginx exposes applications on port 80
   - confirm exposed ports
5. Delete the Pod
6. Review the Pod spec
7. Have a look at the Kubernetes API to determine when pods were introduced

> Not all images expose their applications on port 80. Kubernetes doesn't have a native way to check ports exposed on running container, however, you can connect a shell to a Pod with `kubectl exec` and try one of `netstat -tulpn` or `ss -tulpn` in the container, if installed, to show open ports.

</div>

<div id="soln-lab5-1">

<details>
<summary>lab5.1 solution</summary>

```sh
# host terminal
kubectl run mypod --image=nginx:alpine
kubectl get pods
kubectl describe pods mypod | less
kubectl get pods -o name
kubectl exec -it mypod -- sh
# container terminal
curl localhost # or curl localhost:80, can omit since 80 is the default
netstat -tulpn
ss -tulpn
exit
# host terminal
kubectl delete pods mypod
kubectl explain pod.spec
kubectl api-resources # pods were introduced in v1 - the first version of kubernetes
```

</details>

</div>

<div id="lab5-2">

### Lab 5.2. Managing Pods with YAML file

> In the official k8s docs, you will often find example code with a URL, e.g. `pods/commands.yaml`. The file can be downloaded by appending `https://k8s.io/examples` to the URL, thus: `https://k8s.io/examples/pods/commands.yaml`

```sh
# download file `pods/commands.yaml`
wget https://k8s.io/examples/pods/commands.yaml
# save downloaded file with a new name `comm.yaml`
wget https://k8s.io/examples/pods/commands.yaml -O comm.yaml
# hide output while downloading
wget -q https://k8s.io/examples/pods/commands.yaml
# view contents of a downloaded file without saving
wget -O- https://k8s.io/examples/pods/commands.yaml
# view contents quietly without saving
wget -qO- https://k8s.io/examples/pods/commands.yaml
```

1. Generate a YAML file of a `busybox` Pod that runs the command `sleep 60`, see [create Pod with command and args docs](https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/#define-a-command-and-arguments-when-you-create-a-pod)
2. Apply the YAML file.
3. List created resources
4. View details of the Pod
5. Delete the Pod

</div>

<div id="soln-lab5-2">

<details>
<summary>lab5.2 solution</summary>

```sh
kubectl run mypod --image=busybox --dry-run=client -o yaml --command -- sleep 60 > lab5-2.yaml
kubectl apply -f lab5-2.yaml
kubectl get pods
kubectl describe pods mypod | less
kubectl delete -f lab5-2.yaml
```

</details>

> Some images, like busybox, do not remain in running state by default. An extra command is required, e.g. `sleep 60`, to keep containers using these images in running state for as long as you need. In the CKAD exam, make sure your Pods remain in running states unless stated otherwise

</div>

<div id="lab5-3">

### Lab 5.3. Init Containers

Note that the main container will only be started after the _init container_ enters `STATUS=completed`

```sh
# view logs of pod `mypod`
kubectl logs mypod
# view logs of specific container `mypod-container-1` in pod `mypod`
kubectl logs mypod -c mypod-container-1
```

1. Create a Pod that logs `App is running!` to STDOUT
   - use `busybox:1.28` image
   - the application should `Never` restart
   - the application should use a _Init Container_ to wait for 60secs before starting
   - the _Init Container_ should log `App is initialising...` to STDOUT
   - see [init container docs](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/#init-containers-in-use).
2. List created resources and note Pod `STATUS`
3. View the logs of the main container
4. View the logs of the _init container_
5. View more details of the Pod and note the `State` of both containers.
6. List created resources and confirm Pod `STATUS`
7. Delete Pod

</div>

<div id="soln-lab5-3">

<details>
<summary>lab5.3 solution</summary>

```sh
# partially generate pod manifest
kubectl run myapp --image=busybox:1.28 --restart=Never --dry-run=client -o yaml --command -- sh -c "echo App is running!" > lab5-3.yaml
```

```yaml
# edit lab5-3.yaml to add init container spec
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: myapp
  name: myapp
spec:
  containers:
    - name: myapp
      image: busybox:1.28
      command: ["sh", "-c", "echo App is running!"]
  initContainers:
    - name: myapp-init
      image: busybox:1.28
      command: ["sh", "-c", 'echo "App is initialising..." && sleep 60']
  restartPolicy: Never
```

```sh
kubectl apply -f lab5-3.yaml
kubectl get pods
kubectl logs myapp # not created until after 60secs
kubectl logs myapp -c myapp-init
kubectl describe -f lab5-3.yaml | less
kubectl get pods
kubectl delete -f lab5-3.yaml
```

</details>

</div>

<div id="lab5-4">

### Lab 5.4. Multi-container Pod

1. Create a Pod with 2 containers and a volumne shared by both containers, see [multi-container docs](https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/#creating-a-pod-that-runs-two-containers).
2. List created resources
3. View details of the Pod
4. Delete the Pod

<details>
<summary>lab5.4 solution</summary>

```yaml
# lab5-4.yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
    - name: myapp-1
      image: busybox:1.28
      volumeMounts:
        - name: logs
          mountPath: /var/log
    - name: myapp-2
      image: busybox:1.28
      volumeMounts:
        - name: logs
          mountPath: /var/log
  volumes:
    - name: logs
      emptyDir: {}
```

```sh
kubectl apply -f lab5-4.yaml
kubectl get pods
kubectl describe pods myapp | less
kubectl logs myapp -c myapp-1
kubectl logs myapp -c myapp-2
kubectl delete -f lab5-4.yaml
```

</details>

> Always create single container Pods!

</div>

<div id="lab5-5">

### Lab 5.5. Sidecar Containers

> Remember you can prepend [`https://k8s.io/examples/`](https://kubernetes.io/docs/concepts/workloads/pods/#using-pods) to any example manifest names from the official docs for direct download of the YAML file

1. Create a `busybox` Pod that logs `date` to a file every second
   - expose the logs with a _sidecar container_'s STDOUT to prevent direct access to the main application
   - see example _sidecar container_ manifest `https://k8s.io/examples/admin/logging/two-files-counter-pod-streaming-sidecar.yaml`
2. List created resources
3. View details of the Pod
4. View the logs of the main container
5. View the logs of the _sidecar container_
6. Delete created resources

</div>

<div id="soln-lab5-5">

<details>
<summary>lab5.5 solution</summary>

```yaml
# lab5-5.yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
    - name: myapp
      image: busybox:1.28
      args:
        - /bin/sh
        - -c
        - >
          while true;
          do
            echo $(date) >> /var/log/date.log;
            sleep 1;
          done
      volumeMounts:
        - name: logs
          mountPath: /var/log
    - name: myapp-logs
      image: busybox:1.28
      args: [/bin/sh, -c, "tail -F /var/log/date.log"]
      volumeMounts:
        - name: logs
          mountPath: /var/log
  volumes:
    - name: logs
      emptyDir: {}
```

```sh
kubectl apply -f lab5-5.yaml
kubectl get pods
kubectl describe pods myapp | less
kubectl logs myapp -c myapp
kubectl logs myapp -c myapp-logs
kubectl delete -f lab5-5.yaml
```

</details>

</div>

<div id="lab5-6">

### Lab 5.6 Namespaces

You can also follow the [admin guide doc for namespaces](https://kubernetes.io/docs/tasks/administer-cluster/namespaces/)

> Remember you can connect a shell to a Pod with `kubectl exec` and try one of `netstat -tulpn` or `ss -tulpn` in the container, if installed, to show open ports.

1. Create a Namespace `myns`
2. Create a webserver Pod in the `myns` Namespace
3. Review created resources and confirm `myns` Namespace is assigned to the Pod
4. Delete resources created
5. Review the `NAMESPACED` column of the Kubernetes API resources
6. Review the Namespace object and the Namespace spec

</div>

<div id="soln-lab5-6">

<details>
<summary>lab5.6 solution</summary>

```sh
kubectl create ns myns --dry-run=client -o yaml > lab5-6.yaml
echo --- >> lab5-6.yaml
kubectl run mypod --image=httpd:alpine -n myns --dry-run=client -o yaml >> lab5-6.yaml
kubectl apply -f lab5-6.yaml
kubectl get pods
kubectl describe -f lab5-6.yaml | less
kubectl delete -f lab5-6.yaml
kubectl api-resources | less
kubectl explain namespace | less
kubectl explain namespace --recursive | less
kubectl explain namespace.spec | less
```

</details>

> Remember that namespaced resources are not visible by default unless the namespace is specified \
> 💡 `kubectl get pods` - only shows resources in the `default` namespace \
> 💡 `kubectl get pods -n mynamespace` - shows resources in the `mynamespace` namespace

</div>

<div id="lab6-1">

### Lab 6.1 Troubleshoot failing Pod

1. Create a Pod with mysql image and confirm Pod state
2. Get detailed information on the Pod and review Events (any multiple attempts?), 'State', 'Last State' and their Exit codes.
   - Note that Pod `STATES` might continue to change for containers in error due to default `restartPolicy=Always`
3. Review cluster logs for the Pod
4. Apply relevant fixes until you have a mysql Pod in 'Running' state
5. Delete created resources

</div>

<div id="soln-lab6-1">

<details>
<summary>lab6.1 solution</summary>

```sh
kubectl run mydb --image=mysql --dry-run=client -o yaml > lab6-1.yaml
kubectl apply -f lab6-1.yaml
kubectl get pods
kubectl describe -f lab6-1.yaml | less
kubectl get pods --watch # watch pods for changes
ctrl+c
kubectl delete -f lab6-1.yaml
kubectl run mydb --image=mysql --env="MYSQL_ROOT_PASSWORD=secret" --dry-run=client -o yaml > lab6-1.yaml
kubectl apply -f lab6-1.yaml
kubectl get pods
kubectl describe -f lab6-1.yaml | less
kubectl delete -f lab6-1.yaml
```

</details>

</div>

<div id="lab6-2">

### Lab 6.2. Use port forwarding to access applications

1. Create a webserver Pod
2. List created resources and determine Pod IP address
3. Access the webserver with the IP address (you can use `curl`)
4. Use port forwarding to access the webserver on http://localhost:5000
5. Terminate port forwarding and delete created resources

</div>

<div id="soln-lab6-2">

<details>
<summary>lab6.2 solution</summary>

```sh
kubectl run webserver --image=httpd
kubectl get pods -o wide
curl $POD_IP_ADDRESS
kubectl port-forward webserver 5000:80 &
curl localhost:5000
fg 1
ctrl+c
kubectl delete pods webserver
```

</details>

</div>

<div id="lab6-3">

### Lab 6.3. Set Pod security context

Using the official docs manifest example `pods/security/security-context.yaml` as base to:

1. Use the official manifest example `pods/security/security-context.yaml` as base to create a Pod manifest with these security context options:
   - all containers have a logged-in user of `UID: 1010, GID: 1020`
   - all containers set to run as non-root user
   - mounted volumes for all containers in the pod have group `GID: 1110`
   - escalating to root privileges is disabled ([more on privilege escalation](https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/))
2. Apply the manifest file and review details of created pod
3. Review pod details and confirm security context applied at pod-level and container-level
4. Connect an interactive shell to a container in the pod and confirm the following:
   - current user
   - group membership of current user
   - ownership of entrypoint process
   - ownership of the mounted volume `/data/demo`
   - create a new file `/data/demo/new-file` and confirm file ownership
   - escalate to a shell with root privileges `sudo su`
5. Edit the pod manifest file to the following:
   - do not set logged-in user UID/GID
   - do not set root privilege escalation
   - all containers set to run as non-root user
6. Create a new pod with updated manifest
7. Review pod details and confirm events and behaviour
   - what were your findings?
8. Delete created resources
9. Explore the Pod spec and compare the `securityContext` options available at pod-level vs container-level

</div>

<div id="soln-lab6-3">

<details>
<summary>lab6.3 solution</summary>

```sh
# host terminal
kubectl explain pod.spec.securityContext | less
kubectl explain pod.spec.containers.securityContext | less
wget -qO lab6-3.yaml https://k8s.io/examples/pods/security/security-context.yaml
nano lab6-3.yaml
```

```yaml
# lab6-3.yaml
spec:
  securityContext:
    runAsUser: 1010
    runAsGroup: 1020
    fsGroup: 1110
  containers:
    - name: sec-ctx-demo
      securityContext:
        allowPrivilegeEscalation: false
# etc
```

```sh
# host terminal
kubectl apply -f lab6-3.yaml
kubectl describe pods security-context-demo | less
kubectl get pods security-context-demo -o yaml | grep -A 4 -E "spec:|securityContext:" | less
kubectl exec -it security-context-demo -- sh
# container terminal
whoami
id # uid=1010 gid=1020 groups=1110
ps
ls -l /data # root 1110
touch /data/demo/new-file
ls -l /data/demo # 1010 1110
sudo su # sudo not found - an attacker might try other ways to gain root privileges
exit
# host terminal
nano lab6-3.yaml
```

```yaml
# lab6-3.yaml
spec:
  securityContext:
    runAsNonRoot: true
    fsGroup: 1110
  containers:
    - name: sec-ctx-demo
      securityContext:
        allowPrivilegeEscalation: false
# etc
```

```sh
# host terminal
kubectl delete -f lab6-3.yaml
kubectl apply -f lab6-3.yaml
kubectl get pods security-context-demo
kubectl describe pods security-context-demo | less
# found error creating container - avoid conflicting rules, enforcing non-root user `runAsNonRoot: true` requires a non-root user specified `runAsUser: $UID`
```

</details>

</div>

<div id="lab6-4">

### Lab 6.4. Working with Jobs

1. Create a Job `myjob1` with a suitable image that runs the command `echo Lab 6.4. Jobs!`
2. List jobs and pods
3. Review the details of `myjob1`
4. Review the yaml form of `myjob1`
5. Create another Job `myjob2` with a suitable image that runs the command `date`
6. List jobs and pods
7. Repeat [4] using a manifest file with name `myjob3`
8. List jobs and pods
9. Delete all jobs created
10. List jobs and pods
11. Edit the manifest file and add the following:
   - 5 pods successfully run the command
   - pods are auto deleted after 30secs
12. Apply the new manifest and:
   - confirm the new changes work as expected
   - note the total number of resources created
   - note the behaviour after 30secs
13. Delete created resources
14. Review the Job spec to understand fields related to working with jobs
15. Review the Kubernetes API Resources to determine when jobs was introduced

</div>

<div id="soln-lab6-4">

<details>
<summary>lab6.4 solution</summary>

```sh
kubectl explain job.spec | less
kubectl create job myjob1 --image=busybox -- echo Lab 6.4. Jobs!
kubectl get jobs,pods
kubectl describe job myjob1
kubectl get jobs myjob1 -o yaml
kubectl create job myjob2 --image=busybox -- date
kubectl get jobs,pods
kubectl create job myjob3 --image=busybox --dry-run=client -o yaml -- date >> lab6-4.yaml
kubectl apply -f lab6-4.yaml
kubectl get jobs,pods # so many pods!
kubectl delete jobs myjob1 myjob2 myjob3
kubectl get jobs,pods # pods auto deleted!
nano lab6-4.yaml
```

```yaml
# lab6-4.yaml
kind: Job
spec:
  template:
    spec:
      completions: 5
      ttlSecondsAfterFinished: 30
      containers:
# etc
```

```sh
kubectl apply -f lab6-4.yaml
kubectl get jobs,pods
kubectl get pods --watch # watch pods for 30secs
```

</details>

</div>


<div id="lab6-5">

### Lab 6.5. Working with CronJobs

1. Create a job with a suitable image that runs the `date` command every minute
2. Review details of the created CronJob
3. Review the YAML form of the created CronJob
4. List created resources and compare results before and after 1 minute
5. Delete created resources
6. Review the CronJob spec to understand fields related to working with cronjobs
7. Review the Job spec of a CronJob and compare this to a standard Job spec
8. Review the Kubernetes API Resources to determine when jobs was introduced

</div>

<div id="soln-lab6-5">

<details>
<summary>lab6.5 solution</summary>

```sh
kubectl explain cronjob.spec | less
kubectl explain cronjob.spec.jobTemplate.spec | less
kubectl create cronjob mycj --image=busybox --schedule="* * * * *" -- date
kubectl describe cj mycj | less
kubectl get cj mycj -o yaml | less
kubectl get all
kubectl get pods --watch # watch pods for 60s to see changes
kubectl delete cj mycj # deletes associated jobs and pods!
kubectl api-resources # cronjobs was introduced in batch/v1
```

</details>

> All CronJob `schedule` times are based on the timezone of the [_kube-controller-manager_](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/) \
> Since a CronJob runs a Job periodically, the Job spec auto delete feature `ttlSecondsAfterFinished` is quite handy

</div>

<div id="lab6-6">

### Lab 6.6. Resource limitation

You may use the [official container resource example manifest](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#example-1) or generate a manifest file with `kubectl set resources`.

1. Create a Pod with the following spec:
   - runs in `dev` namespace
   - runs two containers, MongoDB database and webserver frontend
   - restart only on failure, see `pod.spec.restartPolicy`
   - both containers starts with 0.25 CPU, 64 mebibytes RAM
   - both containers does not exceed 1 CPU, 256 mebibytes RAM
2. List created pods
3. Review pod details and confirm the specified resource quotas are applied
4. Edit the Pod manifest as follows:
   - both containers starts with an insufficient amount RAM, e.g 4 mebibytes
   - both containers does not exceed 8 mebibytes RAM
5. Apply the manifest and review behaviour
6. Review logs for both containers
7. Compare the logs output in [6] to details from `kubectl describe`
8. Edit the Pod manifest as follows:
   - both containers starts with an amount of RAM equal to host RAM (run `cat /proc/meminfo` or `free -h`)
   - both containers starts with an amount CPU equal to host CPU (run `cat /proc/cpuinfo` or `lscpu`)
   - both containers does not exceed x2 the amount of host RAM
9. Apply the manifest and review behaviour
10. Delete created resources
11. Review the Pod spec fields related to limits and requests

</div>

<div id="soln-lab6-6">

<details>
<summary>lab6.6 solution</summary>

```sh
kubectl create ns dev --dry-run=client -o yaml >> lab6-6.yaml
echo --- >> lab6-6.yaml
# add the contents of the example manifest to lab6-6.yaml and modify accordingly
nano lab6-6.yaml
```

```yaml
# lab6-6.yaml
kind: Namespace
metadata:
  name: dev
# etc
---
kind: Pod
metadata:
  name: webapp
  namespace: dev
spec:
  restartPolicy: OnFailure
  containers:
    - image: mongo
      name: database
      resources:
        requests:
          memory: "64Mi"
          cpu: "250m"
        limits:
          memory: "256Mi"
          cpu: 1
    - image: nginx
      name: frontend
      resources: # same as above
# etc
```

```sh
kubectl apply -f lab6-6.yaml
kubectl get pods -n dev
kubectl describe pods webapp -n dev | less
kubectl describe pods webapp -n dev | grep -A 4 -E "Containers:|State:|Limits:|Requests:" | less
nano lab6-6.yaml
```

```yaml
# lab6-6.yaml
kind: Pod
spec:
  containers:
    - resources:
        requests:
          memory: "4Mi"
          cpu: "250m"
        limits:
          memory: "8Mi"
          cpu: 1
# etc - use above resources for both containers
```

```sh
kubectl delete -f lab6-6.yaml
kubectl apply -f lab6-6.yaml
kubectl get pods -n dev --watch # watch for OOMKilled | CrashLoopBackOff
kubectl get logs webapp -n dev -c database # not very helpful logs
kubectl get logs webapp -n dev -c frontend
kubectl describe pods webapp -n dev | less # helpful logs - Last State: Terminated, Reason: OutOfMemory (OOMKilled)
kubectl describe pods webapp -n dev | grep -A 4 -E "Containers:|State:|Limits:|Requests:" | less
cat /proc/cpuinfo # check for host memory
cat /proc/meminfo # check for host ram
nano lab6-6.yaml
```

```yaml
# lab6-6.yaml
kind: Pod
spec:
  containers:
    - resources:
        requests:
          memory: "8Gi" # use value from `cat /proc/meminfo`
          cpu: 2 # use value from `cat /proc/cpuinfo`
        limits:
          memory: "16Gi"
          cpu: 4
# etc - use above resources for both containers
```

```sh
kubectl delete -f lab6-6.yaml
kubectl apply -f lab6-6.yaml
kubectl get pods -n dev --watch # remains in Pending until enough resources available
kubectl describe pods webapp
kubectl delete -f lab6-6.yaml
kubectl explain pod.spec.containers.resources | less
```

</details>

> Remember a multi-container Pod is not recommended in live environments but only used here for learning purposes

</div>

<div id="lab6-7">

### Lab 6.7. Resource allocation and usage

This lab requires a Metrics Server running in your cluster, please run `minikube addons enable metrics-server` to enable Metrics calculation.

```sh
# enable metrics-server on minikube
minikube addons enable metrics-server
# list available nodes
kubectl get nodes
# view allocated resources for node and % resource usage for running (non-terminated) pods
kubectl describe node $NODE_NAME
# view nodes resource usage
kubectl top node
# view pods resource uage
kubectl top pod
```

<details>
<summary>output from <code>kubectl describe node</code></summary>

![image](https://user-images.githubusercontent.com/17856665/191446804-8bbf8678-70ef-4f1c-a88c-bcbe01f8b232.png)
</details>

1. Enable Metrics Server in your cluster
2. What is the cluster Node's minimum required CPU and memory?
3. Create a Pod as follows:
   - image `nginx:alpine`
   - does not restart, see `kubectl explain pod.spec`
   - only pulls a new image if not present locally, see `kubectl explain pod.spec.containers`
   - requires 0.2 CPU to start but does not exceed half of the cluster Node's CPU
   - requires 64Mi memory to start but does not exceed half of the cluster Node's memory
4. Review the running Pod and confirm resources configured as expected
5. Delete created resources

</div>

<div id="soln-lab6-7">

<details>
<summary>lab 6.7 solution</code></summary>

```sh
minikube addons enable metrics-server
kubectl get node # show node name
kubectl describe node $NODE_NAME | grep -iA10 "allocated resources:" # cpu 0.95, memory 460Mi
kubectl run mypod --image=nginx:alpine --restart=Never --image-pull-policy=IfNotPresent --dry-run=client -oyaml>lab6-7.yml
kubectl apply -f lab6-7.yml # cannot use `kubectl set` if pod don't exist
kubectl set resources pod mypod --requests=cpu=200m,memory=64Mi --limits=cpu=475m,memory=230Mi --dry-run=client -oyaml|less
nano lab6-7.yml # copy resources section of above output to pod yaml
```

```yaml
kind: Pod
spec:
  containers:
  - name: mypod
    imagePullPolicy: IfNotPresent
    resources:
      limits:
        cpu: 475m
        memory: 230Mi
      requests:
        cpu: 200m
        memory: 64Mi
```

```sh
kubectl delete -f lab6-7.yml
kubectl apply -f lab6-7.yml
kubectl describe -f lab6-7.yml | grep -iA6 limits:
kubectl delete -f lab6-7.yml
```
</details>

</div>

<div id="lab7-1">

### Lab 7.1. Deploy an app with a replicaset

Deployments can be used to rollout a ReplicaSet which manages the number of Pods. In [CKAD](https://www.cncf.io/certification/ckad/) you will only work with ReplicaSets via Deployments

1. Create a deployment with three replicas using a suitable image
2. Show more details of the deployment and review available fields:
   - namespace, labels, selector, replicas, update strategy type, pod template, conditions, replicaset and events
3. List all created resources
4. Delete a Pod and monitor results
5. Compare results to using _naked_ Pods (run a pod and delete it)
6. Delete the ReplicaSet with `kubectl delete rs $rsName` and monitor results
7. Delete created resources
8. Explore the deployment spec
9. Explore the Kubernetes API Resources to determine when deployments and replicasets was introduced

</div>

<div id="soln-lab7-1">

<details>
<summary>lab7.1 solution</summary>
  
```sh
kubectl create deploy myapp --image=httpd --replicas=3
kubectl describe deploy myapp | less
kubectl get all
kubectl delete pod $POD_NAME
kubectl get all
kubectl get pods --watch # watch replicaset create new pod to replace deleted
kubectl run mypod --image=httpd
kubectl get all
kubectl delete pod mypod
kubectl get all # naked pod not recreated
kubectl delete replicaset $REPLICASET_NAME # pods and replicaset deleted
kubectl get all
kubectl get pods --watch # deployment creates new replicaset, and replicaset creates new pods
kubectl delete deploy myapp nginx-deployment
kubectl explain deploy.spec
kubectl api-resources # deployments & replicasets were introduced in apps/v1
# replicasets replaced v1 replicationcontrollers
```
</details>

</div>

<div id="lab7-2">

### Lab 7.2. Scale deployment

A deployment creates a ReplicaSet that manages scalability. Do not manage replicasets outside of deployments.

1. Create a deployment using the official deployment manifest example `controllers/nginx-deployment.yaml`
2. List created resources
3. Edit the deployment with `kubectl edit` and change the `namespace` to dev
4. Save the editor and confirm behaviour
5. Edit the deployment again using a different editor, change the replicas to 12 and upgrade the image version
6. Save the editor and confirm behaviour, then immediately list all resources and review:
   - deployment status for `READY`, `UP-TO-DATE` and `AVAILABLE`
   - replicaset status for `DESIRED`, `CURRENT` and `READY`
   - pod status for `NAME`, `READY` and `STATUS`
   - compare the _ID-suffix_ in the Pods name to the ReplicaSets name
7. View details of deployment to confirm edit applied, including image change
8. Scale down the deployment back to 3 replicas using `kubectl scale` and review same in [6]
9. List all resources and confirm scaling applied
10. Delete created resources
11. Edit the `apiVersion` of the manifest example file to `apps/v0`
12. Apply the edited manifest and confirm behaviour

</div>

<div id="soln-lab7-2">

<details>
<summary>lab7.2 solution</summary>
  
```sh
wget -O lab7-2.yaml https://k8s.io/examples/controllers/nginx-deployment.yaml
kubectl apply -f lab7-2.yaml
kubectl get all
kubectl edit -f lab7-2.yaml
```

```yaml
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: dev
# etc (save failed: not all fields are editable - cancel edit)
```

```sh
KUBE_EDITOR=nano kubectl edit -f lab7-2.yaml
```

```yaml
kind: Deployment
spec:
  replicas: 12
  template:
    spec:
      containers:
        - image: nginx:1.3
# etc - save successful
```

```sh
kubectl get all
kubectl describe -f lab7-2.yaml | less
kubectl scale deploy myapp --replicas=3
kubectl get all
kubectl delete -f lab7-2.yaml
nano lab7-2.yaml
```

```yaml
apiVersion: apps/v0
kind: Deployment
# etc
```

```sh
kubectl apply -f lab7-2.yaml # recognise errors related to incorrect manifest fields
```

</details>

</div>

<div id="lab7-3">

### Lab 7.3. Working with labels

1. Create a deployment `myapp` with three replicas using a suitable image
2. List all deployments and their labels to confirm default labels assigned
3. Add a new label `pipeline: test` to the deployment
4. List all deployments and their labels
5. View more details of the deployment and review labels/selectors
6. View the YAML form of the deployment to see how labels are added in the manifest
7. Verify the default label/selector assigned when you created a new Pod
8. List all resources and their labels filtered by default label of the deployment
9. List all resources and their labels, filtered by new label added, compare with above
10. Remove the default label from one of the pods in the deployment and review behaviour
11. List all pods and their labels
12. List all pods filtered by the default label
13. Delete the deployment
14. Delete the _naked_ Pod from [10]

</div>

<div id="soln-lab7-3">

<details>
<summary>lab7.3 solution</summary>
  
```sh
kubectl create deploy myapp --image=httpd --dry-run=client -o yaml >> lab7-3.yaml
kubectl apply -f lab7-3.yaml
kubectl get deploy --show-labels
kubectl label deploy myapp pipeline=test
kubectl get deploy --show-labels
kubectl describe -f lab7-3.yaml
kubectl get -o yaml -f lab7-3.yaml | less
kubectl run mypod --image=nginx --dry-run=client -o yaml | less
kubectl get all --selector="app=myapp"
kubectl get all --selector="pipeline=test"
kubectl label pod $POD_NAME app- # pod becomes naked/dangling and unmanaged by deployment
kubectl get pods --show-labels # new pod created to replace one with label removed
kubectl get pods --selector="app=myapp" # shows 3 pods
kubectl delete -f lab7-3.yaml # $POD_NAME not deleted! `deploy.spec.selector` is how a deployment find pods to manage!
```
</details>

</div>

<div id="lab7-4">

### Lab 7.4. Rolling updates

1. Review the update strategy field under the deployment spec
2. Create a deployment with a suitable image
3. View more details of the deployment
   - by default, how many pods can be upgraded simultaneously during update?
   - by default, how many pods can be created in addition to the number of replicas during update?
4. Create a new deployment with the following parameters:
   - 5 replicas
   - image `nginx:1.18`
   - additional deployment label `update: feature`
   - maximum of 2 Pods can be updated simultaneously
   - no more than 3 additional Pod created during updates
5. List all resources filtered by the default label
6. List all resources filtered by the additional label
7. List rollout history for all deployments - how many revisions does the new deployment have?
8. Upgrade/downgrade the image version
9. List all resources specific to the new deployment
10. List rollout history specific to the new deployment - how many revisions?
11. View more details of the deployment and note the image and _Events messages_
12. Compare the latest change revision of the new deployment's rollout history to the previous revision
13. Revert the new deployment to its previous revision
14. List all resources specific to the new deployment twice or more to track changes
15. List rollout history specific to the new deployment - any new revisions?
16. Scale the new deployment to 0 Pods
17. List rollout history specific to the new deployment - any new revisions?
18. List all resources specific to the new deployment
19. Delete created resources

</div>

<div id="soln-lab7-4">

<details>
<summary>lab7.4 solution</summary>
  
```sh
kubectl explain deploy.spec.strategy | less
kubectl create deploy myapp --image=nginx --dry-run=client -o yaml > lab7-4.yaml
kubectl apply -f lab7-4.yaml
kubectl describe -f lab7-4.yaml
kubectl get deploy myapp -o yaml | less # for manifest example to use in next step
nano lab7-4.yaml # edit to new parameters
```

```yaml
kind: Deployment
metadata:
  labels: # labels is `map` not `array` so no `-` like containers
    app: myapp
    updates: feature
  name: myapp
spec:
  replicas: 5
  strategy:
    rollingUpdate:
      maxSurge: 3
      maxUnavailable: 2
  template:
    spec:
      containers:
        - image: nginx:1.18
          name: webserver
# etc
```

```sh
kubectl get all --selector="app=myapp"
kubectl get all --selector="updates=feature" # extra deployment label not applied on pods
kubectl rollout history deploy
kubectl set image deploy myapp nginx=n -f lab7-4.yaml
kubectl set image deploy myapp webserver=nginx:1.23
kubectl get all --selector="app=myapp"
kubectl rollout history deploy myapp # 2 revisions
kubectl describe deploy myapp
kubectl rollout history deploy myapp --revision=2
kubectl rollout history deploy myapp --revision=1
kubectl rollout undo deploy myapp --to-revision=1
kubectl get all --selector="app=myapp"
kubectl rollout history deploy myapp # 2 revisions, but revision count incremented
kubectl scale deploy myapp --replicas=0
kubectl rollout history deploy myapp # replicas change does not trigger rollout, only `deploy.spec.template` fields
kubectl get all --selector="app=myapp"
kubectl delete -f lab7-4.yaml
```

</details>

</div>


<div id="lab7-5">

### Lab 7.5. Exploring DaemonSets

DaemonSets can only be created by YAML file, see an official example manifest `controllers/daemonset.yaml`.

1. Compare the DaemonSet manifest to a Deployment manifest - differences/similarities?
2. Apply the example manifest
3. List all resources and note resources created by the DaemonSet
4. View more details of the DaemonSet
5. Delete created resources
6. Review the Kubernetes API Resources to determine when DaemonSets was introduced
7. List existing DaemonSets in the _kube-system_ namespace and their labels
   - what does Kubernetes use a DaemonSet for?
8. List all resources in the _kube-system_ namespace matching the DaemonSet label
9. Review the DaemonSet spec

</div>

<div id="soln-lab7-5">

<details>
<summary>lab7.5 solution</summary>
  
```sh
kubectl create deploy myapp --image=nginx --dry-run=client -o yaml | less # view fields required
wget -qO- https://k8s.io/examples/controllers/daemonset.yaml | less # similar to deployment, except Kind and replicas
kubectl apply -f https://k8s.io/examples/controllers/daemonset.yaml
kubectl get all # note daemonset and related pod
kubectl describe -f https://k8s.io/examples/controllers/daemonset.yaml
kubectl delete -f https://k8s.io/examples/controllers/daemonset.yaml
kubectl api-resources # introduced in version apps/v1
kubectl get ds -n=kube-system --show-labels # used to add network agent `kube-proxy` to all cluster nodes
kubectl get all -n=kube-system --selector="k8s-app=kube-proxy"
kubectl explain daemonset.spec | less
kubectl explain daemonset.spec --recursive | less
```
</details>

</div>

<div id="lab7-6">

### Lab 7.6. Resource usage and Autoscaling

Autoscaling is very important in live environments but not covered in [CKAD](https://www.cncf.io/certification/ckad/). Visit [HorizontalPodAutoscaler Walkthrough](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/#run-and-expose-php-apache-server) for a complete lab on autoscaling.

> The lab requires a _metrics-server_ so install one via Minikube if you plan to complete the lab

```sh
# list minikube addons
minikube addons list
# enable minikube metrics-server
minikube addons enable metrics-server
# disable minikube metrics-server
minikube addons disable metrics-server
```

</div>

<div id="lab8-1">

### Lab 8.1. Connecting applications with services

1. Create a simple deployment with name `webserver`
2. List created resources
3. List endpoints, and pods with their IPs
   - Can you spot the relationship between the Service, Endpoints and Pods?
4. Create a Service for the deployment, exposed on port 80
5. List created resources and note services fields `TYPE`, `CLUSTER-IP`, `EXTERNAL-IP` and `PORT(S)`
6. View more details of the Service and note fields `IPs`, `Port`, `TargetPort` and `Endpoints`
7. View the YAML form of the Service and compare info shown with output in [6]
8. Print the Service env-vars from one of the pods
9. Scale the deployment down to 0 replicas first, then scale up to 2 replicas
10. List all pods and their IPs
11. Print the Service env-vars from one of the pods and compare to results in [3]
12. List endpoints, and pods with their IPs
13. Access the app by the Service: `curl $ClusterIP:$Port`
14. Access the app by the Service from the container host: `minikube ssh` then `curl $ClusterIP:$Port`
15. Run a `busybox` Pod with a shell connected interactively and perform the following commands:
   - run `cat /etc/resolv.conf` and review the output
   - run `nslookup webserver` (service name) and review the output
   - what IPs and/or qualified names do these match?
16. Run a temporary `nginx:alpine` Pod to query the Service by name:
   - first run `kubectl run mypod --rm -it --image=nginx:alpine -- sh`
   - then once in container, run `curl $SERVICE_NAME:$PORT`
   - you should run `curl $SERVICE_NAME.$SERVICE_NAMESPACE:$PORT` if the Service and the temporary Pod are in separate Namespaces
17. Delete created resources
18. Explore the Service object and the Service spec

</div>

<div id="soln-lab8-1">

<details>
<summary>lab 8.1 solution</summary>

```sh
# host terminal
kubectl create deploy webserver --image=httpd --dry-run=client -o yaml > lab8-1.yaml
kubectl apply -f lab8-1.yaml
kubectl get all
kubectl get svc,ep,po -o wide # endpoints have <ip_address:port> of pods targeted by service
echo --- >> lab8-1.yaml
kubectl expose deploy webserver --port=80 --dry-run=client -o yaml >> lab8-1.yaml
kubectl apply -f lab8-1.yaml
kubectl get svc,pods
kubectl describe svc webserver | less
kubectl get svc webserver -o yaml | less # missing endpoints IPs
kubectl exec $POD_NAME -- printenv | grep SERVICE # no service env-vars
kubectl scale deploy webserver --replicas=0; kubectl scale deploy webserver --replicas=2
kubectl get pods -o wide # service env-vars applied to pods created after service
kubectl exec $POD_NAME -- printenv | grep SERVICE
kubectl get endpoints,pods -o wide
curl $CLUSTER_IP # docker-desktop connection error, docker-engine success
minikube ssh
# cluster node terminal
curl $CLUSTER_IP # success with both docker-desktop and docker-engine
exit
# host terminal
kubectl run mypod --rm -it --image=busybox
# container terminal
cat /etc/resolv.conf # shows service ip as dns server
nslookup webserver # shows dns search results, read more at https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#namespaces-of-services
exit
# host terminal
kubectl run mypod --rm -it --image=nginx:alpine -- sh
# container terminal
curl webserver # no need to add port cos default is 80
curl webserver.default # this uses the namespace of the service
exit
# host terminal
kubectl delete -f lab8-1.yaml
kubectl explain service | less
kubectl explain service.spec | less
```

</details>

</div>

<div id="lab8-2">

### Lab 8.2. Connect a frontend to a backend using services

In this lab, we will implement a naive example of a _backend-frontend_ microservices architecture - expose frontend to external traffic with `NodePort` Service while keeping backend hidden with `ClusterIP` Service.

> Note that live environments typically use [_Ingress_](#9-ingress) (covered in the next chapter) to expose applications to external traffic

<details>
<summary><i>Backend-Frontend</i> microservices architecture example</summary>

![backend-frontend microservice architecture](https://user-images.githubusercontent.com/17856665/187857523-8d0fd28f-d540-4453-bae9-ff5481d63e07.png)

</details>

1. Create a simple Deployment, as our `backend` app, with the following spec:
   - image `httpd` (for simplicity)
   - name `backend`
   - has Labels `app: backend` and `tier: webapp`
   - has Selectors `app: backend` and `tier: webapp`
2. Create a Service for the backend app with the following spec:
   - type `ClusterIP`
   - port 80
   - same name, Labels and Selectors as backend Deployment
3. Confirm you can access the app by `$CLUSTER-IP` or `$SERVICE_NAME`
4. Create an [nginx _server block_ config file `nginx/default.conf`]() to redirect traffic for the `/` route to the backend service

   ```sh
   # nginx/default.conf
   upstream backend-server {
       server backend; # dns service discovery within the same namespace use service name
   }

   server {
       listen 80;

       location / {
           proxy_pass http://backend-server;
       }
   }
   ```

5. Create a simple Deployment, as our `frontend` app, with the following spec:
   - image `nginx`
   - name `frontend`
   - has Labels `app: webapp` and `tier: frontend`
   - has Selectors `app: webapp` and `tier: frontend`
   - Remember that Services target Pods by Selector (the Label Selector of the Service must match the Label of the Pod)
   - mounts the nginx config file to `/etc/nginx/conf.d/default.conf` (use fullpath `$(pwd)/nginx/default.conf`)
   - see example [_hostPath volume mount manifest_](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath-configuration-example)
6. Create a Service for the frontend app with the following spec:
   - type `NodePort`
   - port 80
   - same name, Labels and Selectors as frontend Deployment
   - Remember that Services target Pods by Selector
7. Confirm you can access the backend app from the Minikube Node `$(minikube ip):NodePort`
8. Delete created resources

</div>

<div id="soln-lab8-2">

<details>
<summary>lab 8.2 solution</summary>

```sh
kubectl create deploy backend --image=httpd --dry-run=client -o yaml > lab8-2.yaml
echo --- >> lab8-2.yaml
kubectl expose deploy backend --port=80 --dry-run=client -o yaml >> lab8-2.yaml
nano lab8-2.yaml
```

```yaml
# backend deploymemt
kind: Deployment
metadata:
  labels:
    app: backend
    tier: webapp
  name: backend
spec:
  selector:
    matchLabels:
      app: backend
      tier: webapp
  template:
    metadata:
      labels:
        app: backend
        tier: webapp
# backend service
kind: Service
metadata:
  labels:
    app: backend
    tier: webapp
  name: backend
spec:
  selector:
    app: backend
    tier: webapp
```

```sh
kubectl apply -f lab8-2.yaml
curl $CLUSTER_IP # or run in node terminal `minikube ssh`
mkdir nginx
nano nginx/default.conf # use snippet from step [4]
echo --- >> lab8-2.yaml
kubectl create deploy frontend --image=nginx --dry-run=client -o yaml >> lab8-2.yaml
echo --- >> lab8-2.yaml
kubectl expose deploy frontend --port=80 --dry-run=client -o yaml >> lab8-2.yaml
nano lab8-2.yaml
```

```yaml
# frontend deploymemt
kind: Deployment
metadata:
  labels:
    app: frontend
    tier: webapp
  name: frontend
spec:
  selector:
    matchLabels:
      app: frontend
      tier: webapp
  template:
    metadata:
      labels:
        app: frontend
        tier: webapp
    spec:
      containers:
      - image: nginx
        volumeMounts:
        - mountPath: /etc/nginx/conf.d/default.conf
          name: conf-volume
      volumes:
      - name: conf-volume
        hostPath:
          path: /full/path/to/nginx/default.conf # `$(pwd)/nginx/default.conf`
# frontend service
kind: Service
metadata:
  labels:
    app: frontend
    tier: webapp
  name: frontend
spec:
  type: NodePort
  selector:
    app: frontend
    tier: webapp
```

```sh
kubectl apply -f lab8-2.yaml
kubectl get svc,pods
curl $(minikube ip):$NODE_PORT # shows backend httpd page
kubectl delete -f lab8-2.yaml
```

</details>

</div>


<div id="lab9-1">

### Lab 9.1 Enable Ingress

1. List existing namespaces
2. Enable Ingress on minikube
3. List Namespaces and confirm new Ingress Namespace added
4. List all resources in the Ingress Namespace, including the _ingressclass_
5. Review the _ingress-nginx-controller_ Service in YAML form, note service type and ports
6. Review the _ingressclass_ in YAML form, is this marked as the default?
7. Review the Ingress spec

</div>

<div id="soln-lab9-1">

<details>
<summary>lab9.1 solution</summary>
  
```sh
kubectl get ns # not showing ingress-nginx namespace 
minikube addons list # ingress not enable
minikube addons enable ingress
minikube addons list # ingress enabled
kubectl get ns # shows ingress-nginx namespace
kubectl get all,ingressclass -n ingress-nginx # shows pods, services, deployment, replicaset, jobs and ingressclass
kubectl get svc ingress-nginx-controller -o yaml | less
kubectl get ingressclass nginx -o yaml | less # annotations - ingressclass.kubernetes.io/is-default-class: "true"
kubectl explain ingress.spec | less
```
</details>

</div>



<div id="lab9-2">

### Lab 9.2 Understanding ingress

1. Create a Deployment called `web` using a `httpd` image
2. Expose the deployment with a _Cluster-IP_ Service called `web-svc`
3. Create an Ingress called `web-ing` with a _Prefix_ rule to redirect `/` requests to the Service
4. List all created resources - what is the value of Ingress `CLASS`, `HOSTS` & `ADDRESS`?
   - think about why the `CLASS` and `HOSTS` have such values..
5. Access the app `web` via ingress `curl $(minikube ip)`
   - note that unlike Service, a _NodePort_ isn't specified
6. What if we want another application on `/test` path, will this work? Repeat steps 3-7 to confirm:
   - create a new deployment `web2` with image `httpd`
   - expose the new deployment `web2-svc`
   - add new _Prefix_ path to existing ingress rule to redirect `/test` to `web2-svc`
   - are you able to access the new `web2` app via `curl $(minikube ip)/test`?
   - are you still able to access the old `web` app via `curl $(minikube ip)`?
   - what's missing?
7. Let's fix this by adding the correct _Annotation_ to the Ingress config, `kubectl edit ingress web-ing`:
   <details>
   <summary>fix ingress</summary>

   ```yaml
   metadata:
     name: web-ing
     annotations:
       nginx.ingress.kubernetes.io/rewrite-target: /
   ```
   </details>
8. Try to access both apps via URLs `curl $(minikube ip)/test` and `curl $(minikube ip)`
9. Can you access both apps using HTTPS?
10. Review the _ingress-nginx-controller_ by running: `kubectl get svc -n ingress-nginx`
   - what is the _ingress-nginx-controller_ Service type?
   - what are the ports related to HTTP `80` and HTTPS `443`?
11. Can you access both apps via the _ingress-nginx-controller_ NodePorts for HTTP and HTTPS?
12. Delete all created resources

</div>

<div id="lab9-2">

<details>
<summary>lab9.2 solution</summary>
  
```sh
kubectl create deploy web --image=httpd --dry-run=client -oyaml > lab9-2.yml
kubectl apply -f lab9-2.yml
echo --- >> lab9-2.yml
kubectl expose deploy web --name=web-svc --port=80 --dry-run=client -oyaml >> lab9-2.yml
echo --- >> lab9-2.yml
kubectl create ingress web-ing --rule="/*=web-svc:80" --dry-run=client -oyaml >> lab9-2.yml
kubectl apply -f lab9-2.yml
kubectl get deploy,po,svc,ing,ingressclass # CLASS=nginx, HOSTS=*, ADDRESS starts empty then populated later
curl $(minikube ip) # it works
echo --- >> lab9-2.yml
kubectl create deploy web2 --image=httpd --dry-run=client -oyaml > lab9-2.yml
kubectl apply -f lab9-2.yml
echo --- >> lab9-2.yml
kubectl expose deploy web2 --name=web2-svc --port=80 --dry-run=client -oyaml >> lab9-2.yml
KUBE_EDITOR=nano kubectl edit ingress web-ing
```

```yaml
Kind: Ingress
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        ...
      - path: /test
        pathType: Prefix
        backend:
          service:
            name: web2-svc
            port:
              number: 80
# etc
```

```sh
curl $(minikube ip)/test # 404 not found ???
curl $(minikube ip) # it works
KUBE_EDITOR=nano kubectl edit ingress web-ing
```

```yaml
Kind: Ingress
metadata:
  name: web-ing
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
# etc
```

```sh
curl $(minikube ip)/test # it works
curl $(minikube ip) # it works
curl https://$(minikube ip)/test --insecure # it works, see `curl --help`
curl https://$(minikube ip) --insecure # it works
kubectl get svc -n ingress-nginx # NodePort, 80:$HTTP_NODE_PORT/TCP,443:$HTTPS_NODE_PORT/TCP
curl $(minikube ip):$HTTP_NODE_PORT
curl $(minikube ip):$HTTP_NODE_PORT/test
curl https://$(minikube ip):$HTTPS_NODE_PORT --insecure
curl https://$(minikube ip):$HTTPS_NODE_PORT/test --insecure
kubectl delete deploy web web2
kubectl delete svc web-svc web2-svc
kubectl delete ingress web-ing web2-ing
```
</details>

> Ingress relies on Annotations to specify additional configuration. The supported Annotations depends on the Ingress controller type in use - in this case _Ingress-Nginx_ \
> Please visit the [_Ingress-Nginx_ official _Rewrite_ documentation](https://github.com/kubernetes/ingress-nginx/blob/main/docs/examples/rewrite/README.md) for more details

</div>

<div id="lab9-3">

### Lab 9.3. Simple fanout Ingress

1. Create an Ingress `webapp-ingress` that:
   - redirects requests for path `myawesomesite.com/` to a Service `webappsvc:80`
   - redirects requests for path `myawesomesite.com/hello` to a Service `hellosvc:8080`
   - remember to add the _Rewrite_ Annotation
2. List created resources - compare the value of Ingress `HOSTS` to the previous lab
3. View more details of the Ingress and review the notes under _Rules_
4. View the Ingress in YAML form and review the structure of the Rules
5. Create a Deployment `webapp` with image `httpd`
6. Expose the `webapp` Deployment as _NodePort_ with service name `webappsvc`
7. List all created resources - ingress, service, deployment and other resources associated with the deployment
8. View more details of the Ingress and review the notes under _Rules_
9. Can you access `webapp` via the minikube Node `curl $(minikube ip)` or `curl myawesomesite.com`?
10. Create a second Deployment `hello` with image `gcr.io/google-samples/hello-app:1.0`
11. Expose `hello` as _NodePort_ with service name `hellosvc`
12. List newly created resources - service, pods, deployment etc
13. View more details of the Ingress and review the notes under _Rules_
14. Can you access `hello` via `curl $(minikube ip)/hello` or `myawesomesite.com/hello`?
15. Add an entry to `/etc/hosts` that maps the minikube Node IP to an hostname `$(minikube ip) myawesomesite.com`
16. Can you access `webapp` via `curl $(minikube ip)` or `myawesomesite.com` with HTTP and HTTPS 
17. Can you access `hello` via `curl $(minikube ip)/hello` or `myawesomesite.com/hello` with HTTP and HTTPS
18. Can you access `webapp` and `hello` on `myawesomesite.com` via the NodePorts specified by the `ingress-nginx-controller`, `webappsvc` and `hellosvc` Services?
19. Delete created resources

</div>

<div id="soln-lab9-3">

<details>
<summary>lab9.3 solution</summary>
  
```sh
kubectl create ingress webapp-ingress --rule="myawesomesite.com/*=webappsvc:80" --rule="myawesomesite.com/hello/*=hellosvc:8080" --dry-run=client -oyaml > lab9-3.yaml
echo --- >> lab9-3.yaml
kubectl apply -f lab9-3.yaml
kubectl get ingress
kubectl describe ingress webapp-ingress | less # endpoints not found
kubectl get ingress webapp-ingress -oyaml | less
kubectl create deploy webapp --image=httpd --dry-run=client -oyaml >> lab9-3.yaml
echo --- >> lab9-3.yaml
kubectl apply -f lab9-3.yaml
kubectl expose deploy webapp --name=webappsvc --type=NodePort --port=80 --dry-run=client -o yaml >> lab9-3.yaml
echo --- >> lab9-3.yaml
kubectl apply -f lab9-3.yaml
kubectl get ingress,all
kubectl describe ingress webapp-ingress | less # only webappsvc endpoint found
curl $(minikube ip) # 404 not found
curl myawesomesite.com # 404 not found
kubectl create deploy hello --image=gcr.io/google-samples/hello-app:1.0 --dry-run=client -o yaml >> lab9-3.yaml
echo --- >> lab9-3.yaml
kubectl apply -f lab9-3.yaml
kubectl expose deploy hello --name=hellosvc --type=NodePort --port=8080 --dry-run=client -o yaml >> lab9-3.yaml
echo --- >> lab9-3.yaml
kubectl apply -f lab9-3.yaml
kubectl get all --selector="app=hello"
kubectl describe ingress webapp-ingress | less # both endpoints found
curl $(minikube ip)/hello # 404 not found
curl myawesomesite.com/hello # 404 not found
echo "$(minikube ip) myawesomesite.com" | sudo tee -a /etc/hosts # see `tee --help`
curl $(minikube ip) # 404 not found
curl $(minikube ip)/hello # 404 not found
curl myawesomesite.com # it works
curl myawesomesite.com/hello # hello world
curl https://myawesomesite.com --insecure # it works
curl https://myawesomesite.com/hello --insecure # hello world
kubectl get svc -A # find NodePorts for ingress-nginx-controller, webappsvc and hellosvc
curl myawesomesite.com:$NODE_PORT_FOR_WEBAPPSVC # it works
curl myawesomesite.com:$NODE_PORT_FOR_HELLOSVC # hello world
curl myawesomesite.com:$HTTP_NODE_PORT_FOR_NGINX_CONTROLLER # it works
curl myawesomesite.com:$HTTP_NODE_PORT_FOR_NGINX_CONTROLLER/hello # hello world
curl https://myawesomesite.com:$HTTPS_NODE_PORT_FOR_NGINX_CONTROLLER --insecure
curl https://myawesomesite.com:$HTTPS_NODE_PORT_FOR_NGINX_CONTROLLER/hello --insecure
kubectl delete -f lab9-3.yaml
```
</details>

</div>


<div id="lab9-4">

### Lab 9.4. Multiple hosts ingress

1. Review the _IngressClass_ resource object
2. List the Ingress classes created by the minikube ingress addon
3. Create two Deployments `nginx` and `httpd`
4. Expose both deployments as `Cluster-IP` on port 80
5. Create an Ingress with the following:
   - redirects requests for `nginx.yourchosenhostname.com` to the `nginx` Service
   - redirects requests for `httpd.yourchosenhostname.com` to the `httpd` Service
   - both rules should use a `Prefix` path type
6. Review created resources
7. Confirm Ingress _PathType_ and _IngressClass_
8. Review the _IngressClass_ resource YAML form to determine why it was assigned by default
9. Add an entry to `/etc/hosts` that maps the minikube Node IP to hostnames below:
   - `$(minikube ip)  nginx.yourchosenhostname.com`
   - `$(minikube ip)  httpd.yourchosenhostname.com`
10. Verify you can access both deployments via their subdomains
11. Delete created resources

</div>

<div id="soln-lab9-4">

<details>
<summary>lab9.4 solution</summary>
  
```sh
kubectl explain ingressclass | less
kubectl explain ingressclass --recursive | less
kubectl create deploy nginx --image=nginx --dry-run=client -o yaml > lab9-4.yaml
echo --- >> lab9-4.yaml
kubectl expose deploy nginx --port=80 --dry-run=client -o yaml >> lab9-4.yaml
echo --- >> lab9-4.yaml
kubectl create deploy httpd --image=httpd --dry-run=client -o yaml >> lab9-4.yaml
echo --- >> lab9-4.yaml
kubectl expose deploy httpd --port=80 --dry-run=client -o yaml >> lab9-4.yaml
echo --- >> lab9-4.yaml
kubectl create ingress myingress --rule="nginx.yourchosenhostname.com/*=nginx:80" --rule="httpd.yourchosenhostname.com/*=httpd:80" --dry-run=client -o yaml > lab9-4.yaml
echo --- >> lab9-4.yaml
kubectl apply -f lab9-4.yaml
kubectl get ingress,all
kubectl get ingress myingress -o yaml | less # `pathType: Prefix` and `ingressClassName: nginx`
kubectl get ingressclass nginx -o yaml | less # annotation `ingressclass.kubernetes.io/is-default-class: "true"` makes this class the default
echo "
$(minikube ip)  nginx.yourchosenhostname.com
$(minikube ip)  httpd.yourchosenhostname.com
" | sudo tee -a /etc/hosts
curl nginx.yourchosenhostname.com
curl httpd.yourchosenhostname.com
kubectl delete -f lab9-4.yaml
# note that when specifying ingress path, `/*` creates a `Prefix` path type and `/` creates an `Exact` path type
```
</details>

</div>

<div id="lab9-5">

### Lab 9.5. Declare network policy

You may follow the [official declare network policy walkthrough](https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/)

> Minikube Calico plugin might conflict with the rest of the labs, so disabled Calico addon after this lab \
> Prepend `https://k8s.io/examples/` to any example files in the official docs to use the file locally

1. Create a kubernetes cluster in minikube with Calico enabled
2. Confirm Calico is up and running
3. Create a Deployment called `webapp` using image `httpd`
4. Expose the Deployment on port 80
5. Review created resources and confirm pods running
6. Create a busybox Pod and connect an interactive shell
7. Run command in the Pod container `wget --spider --timeout=1 nginx`
8. Limit access to the Service so that only Pods with label `tier=frontend` have access - see official manifest example `service/networking/nginx-policy.yaml`
9. View more details of the _NetworkPolicy_ created 
10. Create a busybox Pod and connect an interactive shell
11. Run command in the Pod container `wget --spider --timeout=1 nginx`
12. Create another busybox Pod with label `tier=frontend` and connect an interactive shell
13. Run command in the Pod container `wget --spider --timeout=1 nginx`
14. Delete created resources
15. Revert to a cluster without Calico

</div>

<div id="soln-lab9-5">

<details>
<summary>lab9.5 solution</summary>
  
```sh
# host terminal
minikube stop
minikube delete
minikube start --kubernetes-version=1.23.9 --driver=docker --cni=calico
kubectl get pods -n kube-system --watch # allow enough time, under 5mins if lucky, more than 10mins if you have bad karma 😼
kubectl create deploy webapp --image=httpd --dry-run=client -o yaml > lab9-5.yaml
kubectl apply -f lab9-5.yaml
echo --- >> lab9-5.yaml
kubectl expose deploy webapp --port=80 --dry-run=client -o yaml > lab9-5.yaml
kubectl apply -f lab9-5.yaml
kubectl get svc,pod
kubectl get pod --watch # wait if pod not in running status
kubectl run mypod --rm -it --image=busybox
# container terminal
wget --spider --timeout=1 webapp # remote file exists
exit
# host terminal
echo --- >> lab9-5.yaml
wget -qO- https://k8s.io/examples/service/networking/nginx-policy.yaml >> lab9-5.yaml
nano lab9-5.yaml
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: mynetpol
spec:
  podSelector:
    matchLabels:
      app: webapp
  ingress:
  - from:
    - podSelector:
        matchLabels:
          tier: frontend
```

```sh
kubectl apply -f lab9-5.yaml
kubectl describe networkpolicy mynetpol | less
kubectl run mypod --rm -it --image=busybox
# container terminal
wget --spider --timeout=1 webapp # wget: download timed out
exit
# host terminal
kubectl run mypod --rm -it --image=busybox --labels="tier=frontend"
# container terminal
wget --spider --timeout=1 webapp # remote file exists
exit
# host terminal
kubectl delete -f lab9-5.yaml
minikube stop
minikube delete
minikube start --kubernetes-version=1.23.9 --driver=docker
```
</details>

</div>

<div id="lab10-1">

### Lab 10.1. PVs and PVCs

```sh
# list PVCs, PVs
kubectl get {pvc|pv|storageclass}
# view more details of a PVC
kubectl decribe {pvc|pv|storageclass} $NAME
```

1. Create a PV with 3Gi capacity using official docs `pods/storage/pv-volume.yaml` manifest file as base
2. Create a PVC requesting 1Gi capacity using official docs `pods/storage/pv-claim.yaml` manifest file as base
3. List created resources
   - What `STATUS` and `VOLUME` does the PVC have?
   - Does the PVC use the existing PV and why or why not?
4. What happens when a PV and PVC are created without specifying a `StorageClass`?
   - repeat steps 1-3 after removing `storageClassName` from both YAML files
   - what was the results?

</div>

<div id="soln-lab10-1">

<details>
<summary>lab 10.1 solution</summary>

```sh
wget -q https://k8s.io/examples/pods/storage/pv-volume.yaml
wget -q https://k8s.io/examples/pods/storage/pv-claim.yaml
nano pv-volume.yaml
nano pv-claim.yaml
```

```yaml
# pv-volume.yaml
kind: PersistentVolume
spec:
  storageClassName: manual
  capacity:
    storage: 3Gi
# pv-claim.yaml
kind: PersistentVolumeClaim
spec:
  storageClassName: manual
  resources:
    requests:
      storage: 1Gi
# etc
```

```sh
kubectl get pv,pvc # STATUS=Bound, task-pv-volume uses task-pv-claim
# when `storageClassName` is not specified, the StorageClass creates a new PV for the PVC
```
</details>

</div>

<div id="lab10-2">

### Lab 10.2. Configuring Pods storage with PVs and PVCs

The benefit of configuring Pods with PVCs is to decouple site-specific details.

You can follow the [official _configure a Pod to use a PersistentVolume for storage_ docs](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolume) to complete this lab.

1. Create a `/mnt/data/index.html` file on cluster host `minikube ssh` with some message, e.g. "Hello, World!"
2. Create a PV with the following parameters, see `https://k8s.io/examples/pods/storage/pv-volume.yaml`
   - uses `hostPath` storage
   - allows multiple pods in the Node access the storage
3. Create a Pod running a webserver to consume the storage, see `https://k8s.io/examples/pods/storage/pv-pod.yaml`
   - uses PVC, see `https://k8s.io/examples/pods/storage/pv-claim.yaml`
   - image is `httpd` and default documentroot is `/usr/local/apache2/htdocs` or `/var/www/html`
4. Verify all resources created `pod,pv,pvc,storageclass`, and also review each detailed information
   - review `STATUS` for PV and PVC
   - did the PVC in [3] bind to the PV in [2], why or why not?
5. Connect to the Pod via an interactive shell and confirm you can view the contents of cluster host file `curl localhost`
6. Clean up all resources created

</div>

<div id="soln-lab10-2">

<details>
  <summary>lab 10.2 solution</summary>

```sh
# host terminal
minikube ssh
# node terminal
sudo mkdir /mnt/data
sudo sh -c "echo 'Hello from Kubernetes storage' > /mnt/data/index.html"
cat /mnt/data/index.html
exit
# host terminal
echo --- > lab10-2.yaml
wget https://k8s.io/examples/pods/storage/pv-volume.yaml -O- >> lab10-2.yaml
echo --- >> lab10-2.yaml
wget https://k8s.io/examples/pods/storage/pv-claim.yaml -O- >> lab10-2.yaml
echo --- >> lab10-2.yaml
wget https://k8s.io/examples/pods/storage/pv-pod.yaml -O- >> lab10-2.yaml
echo --- >> lab10-2.yaml
nano lab10-2.yaml # edit the final file accordingly
kubectl apply -f lab10-2.yaml
kubectl get pod,pv,pvc,storageclass
kubectl describe pod,pv,pvc,storageclass | less
kubectl exec -it task-pv-pod -- /bin/bash
kubectl delete -f lab10-2.yaml
```

</details>

For further learning, see [mounting the same persistentVolume in two places](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#mounting-the-same-persistentvolume-in-two-places) and [access control](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#access-control)

</div>

<div id="lab11-1">

### Lab 11.1. Deployment variables

1. Create a `db` Deployment using `mysql` image
2. Troubleshoot and fix any deployment issues to get a running `STATUS`
3. View more details of the Deployment and note where env-var is specified
4. Review the Deployment in YAML form and note how the env-var is specified
5. Create a `db` Pod with an appropriate environment variable specified
6. Confirm Pod running as expected
7. View more details of the Pod and note where env-var is specified
8. Review the Pod in YAML form and note how the env-var is specified
9. Delete created resources

</div>

<div id="soln-lab11-1">

<details>
<summary>lab11.1 solution</summary>
  
```sh
kubectl create deploy db --image=mysql
kubectl get po --watch # status=containercreating->error->crashloopbackoff->error->etc, ctrl+c to quit
kubectl describe po $POD_NAME # not enough info to find issue, so check logs
kubectl logs $POD_NAME|less # found issue, must specify one of `MYSQL_ROOT_PASSWORD|MYSQL_ALLOW_EMPTY_PASSWORD|MYSQL_RANDOM_ROOT_PASSWORD`
kubectl set env deploy db MYSQL_ROOT_PASSWORD=mysecret
kubectl get po # status=running
kubectl describe deploy db # review deployment env-var format
kubectl get deploy db -oyaml|less # review deployment env-var format
kubectl run db --image=mysql --env=MYSQL_ROOT_PASSWORD=mypwd
kubectl get po # status=running
kubectl describe deploy db # review pod env-var format
kubectl describe deploy,po db | grep -iEA15 "pod template:|containers:" | less # see `grep -h`
kubectl get po db -oyaml|less # review pod env-var format
kubectl delete deploy,po db
```
</details>

> Note that you can [use Pod fields as env-vars](https://kubernetes.io/docs/tasks/inject-data-application/environment-variable-expose-pod-information/#use-pod-fields-as-values-for-environment-variables), as well as [use container fields as env-vars](https://kubernetes.io/docs/tasks/inject-data-application/environment-variable-expose-pod-information/#use-container-fields-as-values-for-environment-variables)

</div>


<div id="lab11-2">

### Lab 11.2. ConfigMaps as environment variables

1. Create a `file.env` file with the following content:
   ```sh
   MYSQL_ROOT_PASSWORD=pwd
   MYSQL_ALLOW_EMPTY_PASSWORD=true
   ```
2. Create a File ConfigMap `mycm-file` from the file using `--from-file` option
3. Create an Env-Var ConfigMap `mycm-env` from the file using `--from-env-file` option
4. Compare details of both ConfigMaps, what can you find?
5. Compare the YAML form of both ConfigMaps, what can you find?
6. Create manifest files for the two Deployments with `mysql` image using the ConfigMaps as env-vars:
   - a Deployment called `web-file` for ConfigMap `mycm-file`
   - a Deployment called `web-env` for ConfigMap `mycm-env`
7. Review both manifest files to confirm if env-vars configured correctly, what did you find?
   - any Deployment with correctly configured env-var?
   - which ConfigMap was used for the working Deployment?
   - Are you aware of the issue here?
8. Create a Deployment with two env-vars from the working ConfigMap
9. Connect a shell to a Pod from the Deployment and run `printenv` to confirm env-vars
10. Create a Pod with env-vars from the working ConfigMap
   - how will you set the env-vars for the Pod?
11. Confirm Pod running or troubleshoot/fix any issues
12. Connect a shell to the new Pod and run `printenv` to confirm env-vars
13. Delete all created resources

</div>

<div id="soln-lab11-2">

<details>
<summary>lab11.2 solution</summary>
  
```sh
echo "MYSQL_ROOT_PASSWORD=mypwd
MYSQL_ALLOW_EMPTY_PASSWORD=true" > file.env
kubectl create cm mycm-file --keys=MYSQL_ROOT_PASSWORD,MYSQL_ALLOW_EMPTY_PASSWORD --from-file=file.env
kubectl create cm mycm-env --keys=MYSQL_ROOT_PASSWORD,MYSQL_ALLOW_EMPTY_PASSWORD --from-env-file=file.env
kubectl describe cm mycm-file mycm-env |less # mycm-file has one filename key while mycm-env has two env-var keys
kubectl get cm mycm-file mycm-env -oyaml|less
kubectl create deploy web-file --image=mysql --dry-run=client -oyaml > webfile.yml
kubectl apply -f webfile.yml # need an existing deployment to generate yaml for env-vars
kubectl set env deploy web-file --keys=MYSQL_ROOT_PASSWORD,MYSQL_ALLOW_EMPTY_PASSWORD --from=configmap/mycm-file --dry-run=client -oyaml
kubectl create deploy web-env --image=mysql --dry-run=client -oyaml | less # no output = keys not found in configmap
kubectl create deploy web-env --image=mysql --dry-run=client -oyaml > webenv.yml
kubectl apply -f webenv.yml # need an existing deployment to generate yaml for env-vars
kubectl set env deploy web-env --keys=MYSQL_ROOT_PASSWORD,MYSQL_ALLOW_EMPTY_PASSWORD --from=configmap/mycm-env --dry-run=client -oyaml|less # output OK and two env-var keys set
# copy the working env-var within the container spec to webenv.yml to avoid adding unnecessary fields
kubectl apply -f webenv.yml
kubectl get deploy,po # deployment web-env shows 1/1 READY, copy pod name
kubectl exec -it $POD_NAME -- printenv # shows MYSQL_ROOT_PASSWORD,MYSQL_ALLOW_EMPTY_PASSWORD
kubectl run mypod --image=mysql --dry-run=client -oyaml > pod.yml
kubectl apply -f pod.yml # need existing pod to generate yaml for env-vars
kubectl set env pod mypod --keys=MYSQL_ROOT_PASSWORD,MYSQL_ALLOW_EMPTY_PASSWORD --from=configmap/mycm-env --dry-run=client -oyaml|less
# copy env-var from output container spec to pod.yml to avoid clutter
kubectl delete -f pod.yml # naked pod cannot update env-var, only deployment
kubectl apply -f pod.yml
kubectl get all,cm # mypod in running state
kubectl exec -it mypod -- printenv
kubectl delete deploy,po,cm mycm-file mycm-env web-file web-env mypod
rm file.env
```
</details>

</div>

<div id="lab11-3">

### Lab 11.3. Mounting ConfigMaps

In the previous lab, only the Env-Var ConfigMap worked for our use-case. In this lab we will see how we can use the File ConfigMap.

You may also follow the [offical _add ConfigMap data to a Volume_ docs](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#add-configmap-data-to-a-volume)

1. Create a `file.env` file with the following content:
   ```sh
   MYSQL_ROOT_PASSWORD=pwd
   ```
2. Create a File ConfigMap `mycm` from the file and verify resource details
3. Create a manifest file for a Pod with the following:
   - uses `mysql` image
   - specify an env-var `MYSQL_ROOT_PASSWORD_FILE=/etc/config/file.env`, see the [_Docker Secrets section of MYSQL image_](https://hub.docker.com/_/mysql)
   - mount ConfigMap `mycm` as a volume to `/etc/config/`, see [Populate a volume with ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#populate-a-volume-with-data-stored-in-a-configmap)
4. Create the Pod and verify all works and env-var set in container
5. Create a `html/index.html` file with any content
6. Create a ConfigMap from the file and verify resource details
7. Create a `webserver` deployment with an appropriate image and mount the file to the DocumentRoot via ConfigMap
   - option `nginx` DocumentRoot - /usr/share/nginx/html
   - option `httpd` DocumentRoot - /usr/local/apache2/htdocs
8. Connect a shell to the container and confirm your file is being served
9. Delete created resources

</div>

<div id="soln-lab11-3">

<details>
<summary>lab11.3 solution</summary>
  
```sh
echo "MYSQL_ROOT_PASSWORD=pwd" > file.env
kubectl create cm mycm --from-file=file.env --dry-run=client -oyaml > lab11-3.yml
echo --- >> lab11-3.yml
kubectl run mypod --image=mysql --env=MYSQL_ROOT_PASSWORD_FILE=/etc/config/file.env --dry-run=client -oyaml >> lab11-3.yml
wget -qO- https://k8s.io/examples/pods/pod-configmap-volume.yaml | less # copy relevant details to lab11-3.yml
nano lab11-3.yml
```

```yaml
kind: Pod
spec:
  volumes:
  - name: config-volume
    configMap:
      name: mycm
  containers:
  - name: mypod
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
# etc, rest same as generated
```

```sh
kubectl apply -f lab11-3.yml
kubectl get po # mypod in running state
kubectl exec mypod -it -- printenv # shows MYSQL_ROOT_PASSWORD_FILE
# part 2 of lab
mkdir html
echo "Welcome to Lab 11.3 - Part 2" > html/index.html
kubectl create cm webcm --from-file=html/index.html
echo --- >> 11-3.yml
kubectl create deploy webserver --image=httpd --dry-run=client -oyaml > lab11-3.yml
nano lab11-3.yml # copy yaml format above and fix indentation
```

```yaml
kind: Deployment
spec:
  template:
    spec:
      volumes:
      - name: config-volume
        configMap:
          name: webcm
      containers:
      - name: httpd
        volumeMounts:
        - name: config-volume
          mountPath: /usr/local/apache2/htdocs
```

```sh
kubectl get deploy,po # note pod name and running status
kubectl exec $POD_NAME -it -- ls /usr/local/apache2/htdocs # index.html
kubectl port-forward pod/$POD_NAME 3000:80 & # bind port 3000 in background
curl localhost:3000 # Welcome to Lab 11.3 - Part 2
fg # bring job to fore-ground, then ctrl+c to terminate
kubectl delete -f lab11-3.yml
```
</details>

> Pay attention to the types of ConfigMaps, File vs Env-Var, and also note their YAML form differences

</div>


<div id="lab11-4">

### Lab 11.4. Decoding secrets

You may follow the [official _managing secrets using kubectl_ docs](https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-kubectl/)

1. Review the CoreDNS Pod in the `kube-system` namespace and determine its `serviceAccountName`
2. Review the ServiceAccount and determine the name of the `Secret` in use
3. View the contents of the `Secret` and decode the value of its keys: `ca.crt` `namespace` and `token`.

</div>

<div id="soln-lab11-4">

<details>
<summary>lab11.4 solution</summary>
  
```sh
kubectl -nkube-system get po # shows name of coredns pod
kubectl -nkube-system get po $COREDNS_POD_NAME -oyaml | grep serviceAccountName
kubectl -nkube-system get sa $SERVICE_ACCOUNT_NAME -oyaml # shows secret name
kubectl -nkube-system get secret $SECRET_NAME -ojsonpath="{.data}" | less # shows the secret keys
kubectl -nkube-system get secret $SECRET_NAME -ojsonpath="{.data.ca\.crt}" | base64 -d # decode ca.crt, BEGIN CERTIFICATE... long string
kubectl -nkube-system get secret $SECRET_NAME -ojsonpath="{.data.namespace}" | base64 -d # decode namespace, kube-system
kubectl -nkube-system get secret $SECRET_NAME -ojsonpath="{.data.token}" | base64 -d # decode token, ey... long string
```
</details>

</div>

<div id="lab11-5">

### Lab 11.5. Secrets as environment variables

Repeat [lab 11.2](#lab112-configmaps-as-environment-variables) with secrets

> See the [official _secrets as container env-vars_ docs](https://kubernetes.io/docs/concepts/configuration/secret/#use-case-as-container-environment-variables)

</div>

<div id="soln-lab11-5">

<details>
<summary>lab11.5 solution</summary>
  
```sh
# very similar to configmap solution, accepting pull-requests
```
</details>

</div>

<div id="lab11-6">

### Lab 11.6. Secrets as files

Repeat [lab 11.3](#lab113-mounting-configmaps) with secrets.

> See the [official _using secrets as files_ docs](https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-files-from-a-pod)

</div>

<div id="soln-lab11-6">

<details>
<summary>lab11.6 solution</summary>
  
```sh
# very similar to configmap solution, accepting pull-requests
```
</details>

</div>

<div id="lab11-7">

### Lab 11.7. Secrets as docker registry credentials

1. Create a secret with the details of your docker credentials
2. View more details of the resource created `kubectl describe`
3. View details of the secret in `yaml`
4. Decode the contents of the `.dockerconfigjson` key with `jsonpath`

> See the [official _create an `imagePullSecret`_ docs](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#create-an-imagepullsecret)

</div>

<div id="soln-lab11-7">

<details>
<summary>lab11.7 solution</summary>
  
```sh
# one-line command can be found in `kubectl create secret -h` examples, accepting pull-requests
```
</details>

</div>


<div id="lab12-1">

### Lab 12.1. Liveness probe

Many applications running for long periods of time eventually transition to broken states, and cannot recover except by being restarted. Kubernetes provides liveness probes to detect and remedy such situations. \
You may follow the [official _define a liveness command_ tutorial](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-a-liveness-command) to complete this lab.

```sh
# get events
kubectl get events
# get events of a specific resource, pod, deployment, etc
kubectl get events --field-selector=involvedObject.name=$RESOURCE_NAME
# watch events for updates
kubectl get events --watch
```

1. Using the official manifest file `pods/probe/exec-liveness.yaml` as base, create a Deployment `myapp` manifest file as follows:
   - busybox image
   - commandline arguments `mkdir /tmp/healthy; sleep 30; rm -d /tmp/healthy; sleep 60; mkdir /tmp/healthy; sleep 600;`
   - a _liveness probe_ that checks for the presence of `/tmp/healthy` directory
   - the Probe should be initiated 10secs after container starts
   - the Probe should perform the checks every 10secs
   > the container creates a directory `/tmp/healthy` on startup, deletes the directory 30secs later, recreates the directory 60secs later \
   > your goal is to monitor the Pod behaviour/statuses during these events, you can repeat this lab until you understand liveness probes
2. Apply the manifest file to create the Deployment
3. Review and monitor created Pod events for 3-5mins
4. Delete created resources

</div>

<div id="soln-lab12-1">

<details>
<summary>lab12.1 solution</summary>
  
```sh
kubectl create deploy myapp --image=busybox --dry-run=client -oyaml -- /bin/sh -c "touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 60; touch /tmp/healthy; sleep 600;" >lab12-1.yml
wget -qO- https://k8s.io/examples/pods/probe/exec-liveness.yaml | less # copy the liveness probe section
nano lab12-1.yml # paste, edit and fix indentation
```

```yaml
kind: Deployment
spec:
  template:
    spec:
      containers:
        livenessProbe:
          exec:
            command:
            - ls # `cat` for file
            - /tmp/healthy
          initialDelaySeconds: 10
          periodSeconds: 10
```

```sh
kubectl apply -f lab12-1.yml
kubectl get po # find pod name
kubectl get events --field-selector=involvedObject.name=$POD_NAME --watch
kubectl delete -f lab12-1.yml
```
</details>

</div>

<div id="lab12-2">

### Lab 12.2. Readiness probe

Sometimes, applications are temporarily unable to serve traffic, for example, a third party service become unavailable, etc. In such cases, you don't want to kill the application, but you don't want to send it requests either. Kubernetes provides readiness probes to detect and mitigate these situations. Both _readiness probe_ and _liveness probe_ use similar configuration.

1. Using the official manifest files `pods/probe/http-liveness.yaml` as base, create a Deployment `myapp` manifest file as follows:
   - `nginx:1.22-alpine` image
   - 2 replicas
   - a _readiness probe_ that uses an HTTP GET request to check that the root endpoint `/` on port 80 returns a success status code
   - the _readiness probe_ should be initiated 3secs after container starts, and perform the checks every 5secs
   - a _liveness probe_ that uses an HTTP GET request to check that the root endpoint `/` on port 80 returns a success status code
   - the _liveness probe_ should be initiated 8secs after the container is ready, and perform checks every 6secs
2. Apply the manifest file to create the Deployment
3. View more details of the Deployment and:
   - confirm how Probe configuration appear, note values for `delay | timeout | period | success | failure`, how do you set these values?
   - review Events for probe-related entries
   - note that no Events generated when Probes successful
4. List running Pods
5. View more details of one of the Pods and:
   - confirm both probes are configured
   - review Events for probe-related entries
6. Lets trigger a probe failure to confirm all works, edit the Deployment and change _readiness probe_ port to 8080
7. Review more details of the Deployment and individual Pods and determine how Probe failures are recorded, on Deployment or Pod?
8. Lets overload our Probe configuration, using official manifest `pods/probe/tcp-liveness-readiness.yaml` as example, edit the Deployment as follows:
   - replace the _readiness probe_ with one that uses TCP socket to check that port 80 is open
   - the _readiness probe_ should be initiated 5secs after container starts, and perform the checks every 10secs
   - replace the _liveness probe_ with one that uses TCP socket to check that port 80 is open
   - the _liveness probe_ should be initiated 15secs after the container starts, and perform checks every 10secs
   - note that these changes trigger a rolling update - chainging parameters within `deploy.spec.template`
9. Review more details of one of the Pod
10. Delete created resources

</div>

<div id="soln-lab12-2">

<details>
<summary>lab 12.2 solution</summary>

```sh
kubectl create deploy myapp --image=nginx:1.22-alpine --replicas=2 --dry-run=client -oyaml > lab12-2.yml
wget -qO- https://k8s.io/examples/pods/probe/http-liveness.yaml | less # copy probe section
nano lab12-2.yml # paste, fix indentation, edit correctly
```

```yaml
kind: Deployment
spec:
  template:
    spec:
      containers:
      - name: nginx
        readinessProbe:
          httpGet:
            path: /
            port: 80 # change this to 8080 in later steps
          initialDelaySeconds: 3
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 8
          periodSeconds: 6
# etc
```

```sh
kubectl apply -f lab12-2.yml
kubectl describe deploy myapp # review `Pod Template > Containers` and Events
kubectl get po
kubectl describe po $POD_NAME # review Containers and Events
KUBE_EDITOR=nano kubectl edit deploy myapp # change port to 8080 and save
kubectl describe deploy myapp # only shows rollout events
kubectl get po # get new pod names
kubectl describe po $NEW_POD_NAME # review Containers and Events
KUBE_EDITOR=nano kubectl edit deploy myapp # replace probes config with below
```

```yaml
kind: Deployment
spec:
  template:
    spec:
      containers:
      - name: nginx
        readinessProbe:
          tcpSocket:
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
        livenessProbe:
          tcpSocket:
            port: 80
          initialDelaySeconds: 15
          periodSeconds: 10
# etc
```

```sh
kubectl get po # get new pod names
kubectl describe po $ANOTHER_NEW_POD_NAME # review Containers and Events, no news is good news
kubectl delete -f lab12-2.yml
```
</details>

</div>

<div id="lab12-3">

### Lab 12.3. Protect slow starting containers with startup probe

Sometimes, you have to deal with legacy applications that might require additional startup time on first initialization. In such cases, it can be tricky to set up _liveness probe_ without compromising the fast response to deadlocks that motivated such a probe. The trick is to set up a _startup probe_ with the same command, HTTP or TCP check, with a `failureThreshold * periodSeconds` long enough to cover the worse case startup time.

1. Using the official manifest files `pods/probe/http-liveness.yaml` as base, create a Deployment `myapp` manifest file as follows:
   - `nginx:1.22-alpine` image
   - 2 replicas
   - a _readiness probe_ that uses an HTTP GET request to check that the root endpoint `/` on port 80 returns a success status code
   - the _readiness probe_ should be initiated 3secs after container starts, and perform the checks every 5secs
   - a _liveness probe_ that uses an HTTP GET request to check that the root endpoint `/` on port 80 returns a success status code
   - the _liveness probe_ should be initiated 8secs after the container is ready, and perform checks every 6secs
   - a _startup probe_ with the same command as the _liveness probe_ but checks every 5secs up to a maximum of 3mins
2. Apply the manifest file to create the Deployment
3. View more details of the Deployment and confirm how all Probe configuration appear
4. List running Pods
5. View more details of one of the Pods and confirm how Probe configuration appear
6. Delete created resources

</div>

<div id="soln-lab12-3">

<details>
<summary>lab 12.3 solution</summary>

```sh
kubectl create deploy myapp --image=nginx:1.22-alpine --replicas=2 --dry-run=client -oyaml > lab12-3.yml
nano lab12-3.yml # add probes
```

```yaml
kind: Deployment
spec:
  template:
    spec:
      containers:
      - name: nginx
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 3
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 8
          periodSeconds: 6
        startupProbe:
          httpGet:
            path: /
            port: 80
          periodSeconds: 5
          failureThreshold: 36 # 5secs x 36 = 3mins
# etc
```

```sh
kubectl apply -f lab12-3.yml
kubectl describe deploy myapp
kubectl get po
kubectl describe po $POD_NAME
kubectl delete -f lab12-2.yml
```
</details>

</div>

<div id="lab12-4">

### Lab 12.4. Blue/Green deployments

[Blue/green deployment](https://kubernetes.io/blog/2018/04/30/zero-downtime-deployment-kubernetes-jenkins/#blue-green-deployment) is a update-strategy used to accomplish zero-downtime deployments. The current version application is marked blue and the new version application is marked green. In Kubernetes, blue/green deployment can be easily implemented with Services.

<details>
  <summary>blue/green update strategy</summary>
  
  ![blue-green update strategy from https://engineering-skcc.github.io/performancetest/Cloud-%ED%99%98%EA%B2%BD-%EC%84%B1%EB%8A%A5%EB%B6%80%ED%95%98%ED%85%8C%EC%8A%A4%ED%8A%B8/](https://user-images.githubusercontent.com/17856665/185770858-83a088c2-4701-4fd6-943e-ebfb020aa498.gif)
</details>

1. Create a `blue` Deployment
   - three replicas
   - image `nginx:1.19-alpine`
   - on the cluster Node, create an HTML document `/mnt/data/index.html` with any content
   - mount the `index.html` file to the DocumentRoot as a _HostPath_ volume
2. Expose `blue` Deployment on port 80 with Service name `bg-svc`
3. Verify created resources and test access with `curl`
4. Create a new `green` Deployment using [1] as base
   - three replicas
   - use a newer version of the image `nginx:1.21-alpine`
   - on the cluster Node, create a new HTML document `/mnt/data2/index.html` with different content
   - mount the `index.html` file to the DocumentRoot as a _HostPath_ volume
5. Verify created resources and test access with `curl`
6. Edit `bg-svc` Service _Selector_ as `app=green` to redirect traffic to `green` Deployment
7. Confirm all working okay with `curl`
8. Delete created resources

</div>

<div id="soln-lab12-4">

<details>
<summary>lab 12.4 solution</summary>

```sh
# host terminal
minikube ssh
# node terminal
sudo mkdir /mnt/data /mnt/data2
echo "This is blue deployment" | sudo tee /mnt/data/index.html
echo "Green deployment" | sudo tee /mnt/data2/index.html
exit
# host terminal
kubectl create deploy blue --image=nginx:1.19-alpine --replicas=3 --dry-run=client -oyaml>12-4.yml
nano 12-4.yml # add hostpath volume and pod template label
```

```yaml
kind: Deployment
spec:
  template:
    spec:
      containers:
        volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: testvol
      volumes:
      - name: testvol
        hostPath:
          path: /mnt/data
```

```sh
kubectl apply -f lab12-4.yml
kubectl expose deploy blue --name=bg-svc --port=80
kubectl get all,ep -owide
kubectl run mypod --rm -it --image=nginx:alpine -- sh
# container terminal
curl bg-svc # This is blue deployment
exit
# host terminal
cp lab12-4.yml lab12-4b.yml
nano lab12-4b.yml # change `blue -> green`, hostpath `/mnt/data2`, image `nginx:1.21-alpine
kubectl apply -f lab12-4b.yml
kubectl edit svc bg-svc # change selector `blue -> green`
kubectl run mypod --rm -it --image=nginx:alpine -- sh
# container terminal
curl bg-svc # Green deployment
exit
kubectl delete -f lab12-4.yml,lab12-4b.yml
```

</details>

</div>

<div id="lab12-5">

### Lab 12.5. Canary deployments

Canary deployment is an update strategy where updates are deployed to a subset of users/servers (canary application) for testing prior to full deployment. This is a scenario where Labels are required to distinguish deployments by release or configuration.

<details>
  <summary>canary update strategy</summary>
  
  ![canary update strategy from https://life.wongnai.com/project-ceylon-iii-argorollouts-55ec70110f0a](https://user-images.githubusercontent.com/17856665/185770905-7c3901ec-f97a-4046-8411-7a722b0601a4.png)
</details>

1. Create a webserver application
   - three replicas
   - _Pod Template_ label `updateType=canary`
   - use image` nginx:1.19-alpine`
   - create an HTML document `index.html` with any content
   - mount the `index.html` file to the DocumentRoot as a _ConfigMap_ volume
2. Expose the Deployment on port 80 with Service name `canary-svc`
3. Verify created resources and test access with `curl`
4. Create a new application using [1] as base
   - one replica
   - _Pod Template_ label `updateType=canary`
   - use a newer version of the image `nginx:1.22-alpine`
   - create a new HTML document `index.html` with different content
   - mount the `index.html` file to the DocumentRoot as a _ConfigMap_ volume
5. Verify created resources and confirm the Service targets both webservers
6. Run multiple `curl` requests to the IP in [2] and confirm access to both webservers
7. Scale up the new webserver to three replicas and confirm all Pods running
8. Scale down the old webserver to zero and confirm no Pods running
9. Delete created resources

> Scaling down to zero instead of deleting provides an easy option to revert changes when there are issues

</div>

<div id="soln-lab12-5">

<details>
<summary>lab 12.5 solution</summary>

```sh
# host terminal
kubectl create cm cm-web1 --from-literal=index.html="This is current version"
kubectl create deploy web1 --image=nginx:1.19-alpine --replicas=3 --dry-run=client -oyaml>12-5.yml
nano 12-5.yml # add configmap volume and pod template label
```

```yaml
kind: Deployment
spec:
  selector:
    matchLabels:
      app: web1
      updateType: canary
  template:
    metadata:
      labels:
        app: web1
        updateType: canary
    spec:
      containers:
        volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: testvol
      volumes:
      - name: testvol
        configMap:
          name: cm-web1
```

```sh
kubectl apply -f lab12-5.yml
kubectl expose deploy web1 --name=canary-svc --port=80
kubectl get all,ep -owide
kubectl run mypod --rm -it --image=nginx:alpine -- sh
# container terminal
curl canary-svc # This is current version
exit
# host terminal
cp lab12-5.yml lab12-5b.yml
kubectl create cm cm-web2 --from-literal=index.html="New version"
nano lab12-5b.yml # change `web1 -> web2`, image `nginx:1.22-alpine`, replicas 1, add pod template label
kubectl apply -f lab12-5b.yml
kubectl get all,ep -owide # more ip addresses added to endpoint
kubectl run mypod --rm -it --image=nginx:alpine -- sh
# container terminal
watch "curl canary-svc" # both "New version" and "This is current version"
kubectl scale deploy web2 --replicas=3
kubectl get rs,po -owide
kubectl scale deploy web1 --replicas=0
kubectl get rs,po -owide
kubectl delete -f lab12-5.yml,lab12-5b.yml
```

</details>

</div>

<div id="lab13-1">

### Lab 13.1. Using kubectl proxy to access the API

Rather than run `kubectl` commands directly, we can use `kubectl` as a reverse proxy to provide the location and authenticate requests. See [access the API using kubectl proxy](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/#using-kubectl-proxy) for more details.

You may follow the [official _accessing the rest api_ docs](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/#directly-accessing-the-rest-api)

1. Expose the API with `kube-proxy`
2. Confirm k8s version information with `curl`
3. Explore the k8s API with curl
4. Create a deployment
5. List the pods created with `curl`
6. Get details of a specific pod with `curl`
7. Delete the pod with `curl`
8. Confirm pod deletion

</div>


<div id="lab13-2">

### Lab 13.2. Accessing the API without kubectl proxy

This requires using the token of the default ServiceAccount. The token can be read directly (see [lab 11.4 - decoding secrets](#lab-114-decoding-secrets)), but the recommended way to get the token is via the [TokenRequest API](https://kubernetes.io/docs/reference/kubernetes-api/authentication-resources/token-request-v1/).

You may follow the [official _access the API **without** kubectl proxy_ docs](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/#without-kubectl-proxy).

1. Request the ServiceAccount token by YAML. You can also request by [`kubectl create token $SERVICE_ACCOUNT_NAME`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-em-token-em-) on Kubernetes v1.24+.
2. Wait for the token controller to populate the Secret with a token
3. Use `curl` to access the API with the generated token as credentials

</div>

<div id="soln-lab13-2">

<details>
  <summary>lab 13.2 solution</summary>

```sh
# request token
kubectl apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: default-token
  annotations:
    kubernetes.io/service-account.name: default
type: kubernetes.io/service-account-token
EOF
# confirm token generated (optional)
kubectl get secret default-token -o yaml
# use token
APISERVER=$(kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}')
TOKEN=$(kubectl get secret default-token -o jsonpath='{.data.token}' | base64 --decode)
curl $APISERVER/api --header "Authorization: Bearer $TOKEN" --insecure
```

</details>

> Using `curl` with the `--insecure` option [skips TLS certificate validation](https://curl.se/docs/sslcerts.html)

</div>

<div id="lab13-3">

### Lab 13.3. Accessing the API from inside a Pod

You may follow the [official _access the API from within a Pod_ docs](https://kubernetes.io/docs/tasks/run-application/access-api-from-pod/#without-using-a-proxy).

> From within a Pod, the Kubernetes API is accessible via the `kubernetes.default.svc` hostname

1. Connect an interactive shell to a container in a running Pod (create one or use existing)
2. Use `curl` to access the API at `kubernetes.default.svc/api` with the automounted ServiceAccount credentials (`token` and `certificate`)
3. Can you access the Pods list at `kubernetes.default.svc/api/v1/namespaces/default/pods`?

</div>

<div id="soln-lab13-3">

<details>
  <summary>lab 13.3 solution</summary>

```sh
# connect an interactive shell to a container within the Pod
kubectl exec -it $POD_NAME -- /bin/sh
# use token stored within container to access API
SA=/var/run/secrets/kubernetes.io/serviceaccount
CERT_FILE=$($SA/ca.crt)
TOKEN=$(cat $SA/token)
curl --cacert $CERT_FILE --header "Authorization: Bearer $TOKEN" https://kubernetes.default.svc/api
```

</details>

</div>


<div id="lab13-4">

### Lab 13.4. Exploring RBAC

In [lab 13.3](#lab-123-accessing-the-api-from-a-pod-without-kubectl) we were unable to access the PodList API at `kubernetes.default.svc/api/v1/namespaces/default/pods`. Lets apply the required permissions to make this work.

1. Create a ServiceAccount and verify
2. Create a Role with permissions to list pods and verify
3. Create a RoleBinding that grants the Role permissions to the ServiceAccount, within the `default` namespace, and verify
4. Create a "naked" Pod bound to the ServiceAccount
5. Connect an interactive shell to the Pod and use `curl` to PodList API
6. Can you access the API to get a specific Pod like the one you're running? Hint: Role permissions
7. Can you use a deployment instead of a "naked" Pod?

</div>

<details>
  <summary>lab 13.4 solution</summary>

```sh
# create service account yaml
kubectl create serviceaccount test-sa --dry-run=client -o yaml > lab13-4.yaml
echo --- >> lab13-4.yaml
# create role yaml
kubectl create role test-role --resource=pods --verb=list --dry-run=client -o yaml >> lab13-4.yaml
echo --- >> lab13-4.yaml
# create rolebinding yaml
kubectl create rolebinding test-rolebinding --role=test-role --serviceaccount=default:test-sa --namespace=default --dry-run=client -o yaml >> lab13-4.yaml
echo --- >> lab13-4.yaml
# create configmap yaml
kubectl create configmap test-cm --from-literal="SA=/var/run/secrets/kubernetes.io/serviceaccount" --dry-run=client -o yaml >> lab13-4.yaml
echo --- >> lab13-4.yaml
# create pod yaml
kubectl run test-pod --image=nginx --dry-run=client -o yaml >> lab13-4.yaml
# review & edit yaml to add configmap and service account in pod spec, see `https://k8s.io/examples/pods/pod-single-configmap-env-variable.yaml`
nano lab13-4.yaml
# create all resources
kubectl apply -f lab13-4.yaml
# verify resources
kubectl get sa test-sa
kubectl describe sa test-sa | less
kubectl get role test-role
kubectl describe role test-role | less
kubectl get rolebinding test-rolebinding
kubectl describe rolebinding test-rolebinding | less
kubectl get configmap test-cm
kubectl describe configmap test-cm | less
kubectl get pod test-pod
kubectl describe pod test-pod | less
# access k8s API from within the pod
kubectl exec -it test-pod -- bash
TOKEN=$(cat $SA/token)
HEADER="Authorization: Bearer $TOKEN"
curl -H $HEADER https://kubernetes.default.svc/api --insecure
curl -H $HEADER https://kubernetes.default.svc/api/v1/namespaces/default/pods --insecure
curl -H $HEADER https://kubernetes.default.svc/api/v1/namespaces/default/pods/$POD_NAME --insecure
curl -H $HEADER https://kubernetes.default.svc/api/v1/namespaces/default/deployments --insecure
exit
# clean up
kubectl delete -f lab13-4.yaml
```

</details>

</div>

<div id="lab14-1">

### Lab 14.1. Explore Helm Charts

1. [Install Helm](https://github.com/helm/helm#install)
2. List installed Helm Charts
3. List installed Helm repos
4. Find and install a Chart from [ArtifactHUB](https://artifacthub.io/)
5. View resources created in your cluster by the Helm Chart
6. Update Helm repo
7. List installed Helm Charts
8. List installed Helm repos
9. View details of installed Chart
10. View status of installed Chart
11. Search for available Charts in added Helm repo

</div>


<div id="lab14-2">

### Lab 14.2. Customising Helm Charts

1. Download a Chart template and extract the content to a specified directory
2. Edit the template and change any value
3. Verify the edited template
4. Install the edited template
5. View resources created by the Helm Chart
6. List installed Charts
7. List installed Helm repos

</div>

<div id="lab14-4">

### Lab 14.4. Custom objects

You can follow the [official CRD tutorial](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/).

1. Create a custom resource from the snippet above
2. Confirm a new API resource added
3. Create a custom object of the custom resource
   ```sh
   apiVersion: "stable.example.com/v1"
   kind: CronTab
   metadata:
     name: my-new-cron-object
   spec:
     cronSpec: "* * * * */5"
     image: my-awesome-cron-image
   ```
4. Review all resources created and confirm the `shortName` works
5. Directly access the Kubernetes REST API and confirm endpoints for:
   - group `/apis/<group>`
   - version `/apis/<group>/<version>`
   - plural `/apis/<group>/<version>/<plural>`
6. Clean up by deleting with the manifest files

</div>

<div id="lab14-5">

### Lab 14.5. Operators

See the [official Calico install steps](https://projectcalico.docs.tigera.io/getting-started/kubernetes/minikube).

```sh
# 1. start a new cluster with network `192.168.0.0/16` or `10.10.0.0/16` whichever subnet is free in your network
minikube start --kubernetes-version=1.23.9 --network-plugin=cni --extra-config=kubeadm.pod-network-cidr=10.10.0.0/16
# 2. install tigera calico operator
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.0/manifests/tigera-operator.yaml
# 3. install custom resource definitions
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.0/manifests/custom-resources.yaml
# 4. view an installation resource
kubectl get installation -o yaml | less
# 5. verify calico installation
watch kubectl get pods -l k8s-app=calico-node -A
# you can use wget to view a file without saving
wget -O- https://url/to/file | less
wget -qO- https://url/to/file | less # quiet mode
```

1. Start a Minikube cluster with the `cni` network plugin and a suitable subnet
2. List existing namespaces
3. Install the Tigera Calico operator
4. Confirm resources added:
   - new API resources for `tigera`
   - new namespaces
   - resources in the new namespaces
5. Review the CRDs manifest file and ensure matching `cidr`, then install
6. Confirm resources added:
   - new `Installation` resource
   - new namespaces
   - resources in the new namespaces (Calico Pods take awhile to enter `Running` status)

</div>


<div id="lab14-6">

### Lab 14.6. Components of a StatefulSet

See the [example manifest](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#components)

1. Create a StatefulSet based on the example manifest
2. Verify resources created and compare to a regular deployment
3. Confirm persistent volume claims created

</div>
