### Task - Docker image

```Dockerfile
FROM nginx:1.22-alpine
EXPOSE 80
```

Using docker and the Dockerfile above, build an image with tag `bootcamp/nginx:v1` and tag `ckad/nginx:latest`. Once complete, export a tar file of the image to `/home/$USER/ckad-tasks/docker/nginx.tar`. \
Run a container named `web-test` from the image `bootcamp/nginx:v1` accessible on port 2000, and another container named `web-test2` from image `ckad/nginx:latest` accessible on port 2001. Leave both containers running.

What commands would you use to perform the above operations using `podman`? Specify these commands on separate lines in file `/home/$USER/ckad-tasks/docker/podman-commands`

</div>

<div id="hint-task2-1">

<details>
  <summary>hints</summary>

  <details>
  <summary>hint 1</summary>

  You can specify multiple tags when building an image `docker build -t tag1 -t tag2 /path//to/dockerfile-directory`
  </details>

  <details>
  <summary>hint 2</summary>

  Try to find the command for exporting a docker image with `docker image --help`
  </details>

  <details>
  <summary>hint 2</summary>

  Did you run the containers in detached mode?
  </details>

  <details>
  <summary>hint 3</summary>

  You can export a docker image to a tar file with `docker image save -o /path/to/output/file $IMAGE_NAME`
  </details>

  <details>
  <summary>hint 4</summary>

  Did you expose the required ports when creating the containers? You can use `docker run -p $HOST_PORT:$CONTAINER_PORT`
  </details>

  <details>
  <summary>hint 5</summary>

  Did you verify the containers running at exposed ports `curl localhost:2000` and `curl localhost:2001`?
  </details>

  <details>
  <summary>hint 6</summary>

  Docker and Podman have interchangeable commands, therefore, the only change is `docker -> podman`, For example, `docker run -> podman run`, `docker build -> podman build`, etc.
  </details>
</details>

</div>

### Task - Pods

Imagine a student in the CKAD Bootcamp training reached out to you for assistance to finish their homework. Their task was to create a `webserver` with a sidecar container for logging in the `cow` namespace. Find this Pod, which could be located in one of the Namespaces `ape`, `cow` or `fox`, and ensure it is configured as required.

At the end of your task, copy the log file used by the logging container to directory `/home/$USER/ckad-tasks/pods/`

- Command to setup environment:
  ```sh
  printf '\nlab: environment setup in progress...\n'; echo '{"apiVersion":"v1","items":[{"kind":"Namespace","apiVersion":"v1","metadata":{"name":"fox"}},{"kind":"Namespace","apiVersion":"v1","metadata":{"name":"ape"}},{"kind":"Namespace","apiVersion":"v1","metadata":{"name":"cow"}},{"apiVersion":"v1","kind":"Pod","metadata":{"labels":{"run":"box"},"name":"box","namespace":"ape"},"spec":{"containers":[{"args":["sleep","3600"],"image":"busybox","name":"box"}],"dnsPolicy":"ClusterFirst","restartPolicy":"Always"}},{"apiVersion":"v1","kind":"Pod","metadata":{"labels":{"run":"for-testing"},"name":"for-testing","namespace":"fox"},"spec":{"containers":[{"args":["sleep","3600"],"image":"busybox","name":"for-testing"}],"dnsPolicy":"ClusterFirst","restartPolicy":"Always"}},{"apiVersion":"v1","kind":"Pod","metadata":{"labels":{"run":"webserver"},"name":"webserver","namespace":"fox"},"spec":{"containers":[{"name":"server","image":"ngnx:1.20-alpine","volumeMounts":[{"name":"serverlog","mountPath":"/usr/share/nginx/html"}]},{"name":"logger","image":"busybox:1.28","args":["/bin/sh","-c","while true; do  echo $(date) >> /usr/share/nginx/html/1.log;\n  sleep 30;\ndone\n"],"volumeMounts":[{"name":"serverlog","mountPath":"/usr/share/nginx/html"}]}],"volumes":[{"name":"serverlog","emptyDir":{}}]}}],"metadata":{"resourceVersion":""},"kind":"List"}' | kubectl apply -f - >/dev/null; echo 'lab: environment setup complete!'
  ```
- Command to destroy environment:
  ```sh
  kubectl delete ns ape cow fox
  ```

</div>

<div id="hint-task5-1">

<details>
  <summary>hints</summary>

  <details>
  <summary>hint 1</summary>

  Did you search for Pods in specific namespaces, e.g. `kubectl get pod -n ape`?
  </details>

  <details>
  <summary>hint 2</summary>

  Did you review the Pod error message under _STATUS_ column of `kubectl get po` command? You can reveal more information with `kubectl get -owide`.
  </details>

  <details>
  <summary>hint 3</summary>

  Did you review more details of the Pod, especially details under _Containers_ section of `kubectl describe po` command?
  </details>

  <details>
  <summary>hint 4</summary>

  Is the `webserver` Pod up and running in the `cow` Namespace? Remember this is the requirement, so migrate the Pod if not in correct Namespace. No other resources should be migrated.
  </details>

  <details>
  <summary>hint 5</summary>

  Did you delete the `webserver` Pod in wrong Namespace `fox`?
  </details>

  <details>
  <summary>hint 6</summary>

  You can use `kubectl cp --help` to copy files and directories to and from containers. See [_kubectl_ cheatsheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/#copy-files-and-directories-to-and-from-containers) for more details.
  </details>
</details>

</div>

<div id="task5-2">

### Task - Pods II

In the `rat` Namespace (create if required), create a Pod named `webapp` that runs `nginx:1.22-alpine` image and has env-var `NGINX_PORT=3005` which determines the port exposed by the container. The Pod container should be named `web` and should mount an `emptyDir` volume to `/etc/nginx/templates`. \
The Pod should have an _Init Container_ named `web-init`, running `busybox:1.28` image, that creates a file in the same `emptyDir` volume, mounted to `/tempdir`, with below command:

```sh
echo "server {\n\tlisten\t${NGINX_PORT};\n\n\tlocation / {\n\t\troot\t/usr/share/nginx/html;\n\t}\n}" > /tempdir/default.conf.template
```

</div>

<div id="hint-task5-2">

<details>
  <summary>hints</summary>

  <details>
  <summary>hint 1</summary>

  Did you create the Pod in Namespace `rat`?
  </details>

  <details>
  <summary>hint 2</summary>

  Did you set environment variable `NGINX_PORT=3005` in container `web`? See `kubectl run --help` for how to set an environment variable in a container. 
  </details>

  <details>
  <summary>hint 3</summary>

  Did you set Pod's `containerPort` parameter to be same value as env-var `NGINX_PORT`? Since the env-var `NGINX_PORT` determines the container port, you must change set the `containerPort` parameter to this value. See `kubectl run --help` for how to set port exposed by container.
  </details>

  <details>
  <summary>hint 4</summary>

  Did you specify an `emptyDir` volume and mounted it to `/etc/nginx/templates` in Pod container `web`? See [example pod manifest](#pod-manifest-file).
  </details>

  <details>
  <summary>hint 5</summary>

  Did you create `web-init` as an _Init Container_ under `pod.spec.initContainers`? See [_lab 5.3 - init containers_](#lab-53-init-containers).
  </details>

  <details>
  <summary>hint 6</summary>

  Did you run appropriate command in _Init Container_? You can use _list-form_, or _array-form_ with single quotes.

  ```yaml
  # list form
  command:
  - /bin/sh
  - -c
  - echo "..." > /temp...
  # array form with single quotes
  command: ["/bin/sh", "-c", "echo '...' > /temp..."]
  ```
  </details>

  <details>
  <summary>hint 7</summary>

  Did you specify an `emptyDir` volume, mounted to `/tempdir` in _Init Container_ `web-init`? See [example pod manifest](#pod-manifest-file).
  </details>

  <details>
  <summary>hint 8</summary>

  Did you confirm that a webpage is being served by container `web` on specified port? Connect a shell to the container and run `curl localhost:3005`.
  </details>
</details>

</div>

### Task - CronJobs

In the `boa` Namespace, create a Pod that runs the shell command `date`, in a busybox container, once every hour, regardless success or failure. Job should terminate after 20s even if command still running. Jobs should be automatically deleted after 12 hours. A record of 5 successful Jobs and 5 failed Jobs should be kept. All resources should be named `bootcamp`, including the container. You may create a new Namespace if required.

At the end of your task, to avoid waiting an hour to confirm all works, manually run the Job from the Cronjob and verify expected outcome.

</div>

<div id="hint-task6-1">

<details>
  <summary>hints</summary>

  <details>
  <summary>hint 1</summary>

  Did you create the Cronjob in the `boa` Namespace? You can generate YAML with Namespace specified, see [lab 5.6](#lab-56-namespaces)
  </details>

  <details>
  <summary>hint 2</summary>

  You can generate YAML for Cronjob schedule and command, see [_lab 6.5 - working with cronjobs_](#lab-65-working-with-cronjobs)
  </details>

  <details>
  <summary>hint 3</summary>

  See `kubectl explain job.spec` for terminating and auto-deleting Jobs after specified time.
  </details>

  <details>
  <summary>hint 4</summary>

  See `kubectl explain cronjob.spec` for keeping successful/failed Jobs.
  </details>

  <details>
  <summary>hint 5</summary>

  You can create a Job to manually run a Cronjob, see `kubectl create job --help`
  </details>

  <details>
  <summary>hint 6</summary>

  Did you create the Job in the `boa` Namespace?
  </details>

  <details>
  <summary>hint 7</summary>

  Did you specify `cronjob.spec.jobTemplate.spec.activeDeadlineSeconds` and `cronjob.spec.jobTemplate.spec.ttlSecondsAfterFinished`?
  </details>

  <details>
  <summary>hint 8</summary>

  Did you specify `cronjob.spec.failedJobsHistoryLimit` and `cronjob.spec.successfulJobsHistoryLimit`?
  </details>

  <details>
  <summary>hint 9</summary>

  After Cronjob creation, did you verify configured parameters in `kubectl describe`?
  </details>

  <details>
  <summary>hint 10</summary>

  After manual Job creation, did you verify Job successfully triggered?
  </details>
</details>

</div>

<div id="task6-2">

### Task - Resources and Security Context

A client requires a Pod running the `nginx:1.21-alpine` image with name `webapp` in the `dog` Namespace. The Pod should start with 0.25 CPU and 128Mi memory, but shouldn't exceed 0.5 CPU and half of the Node's memory. All processes in Pod containers should run with user ID 1002 and group ID 1003. Containers mustn't run in `privileged` mode and privilege escalation should be disabled. You may create a new Namespace if required.

When you are finished with the task, the client would also like to know the Pod with the highest memory consumption in the `default` Namespace. Save the name the Pod in the format `<namespace>/<pod-name>` to a file `/home/$USER/ckad-tasks/resources/pod-with-highest-memory`

</div>

<div id="hint-task6-2">

<details>
  <summary>hints</summary>

  <details>
  <summary>hint 1</summary>

  Did you create the resource in the `dog` Namespace? You can generate YAML with Namespace specified, see [lab 5.6](#lab-56-namespaces)
  </details>

  <details>
  <summary>hint 2</summary>

  You can separately generate YAML for the `pod.spec.containers.resources` section, see [_lab 6.7 - resource allocation and usage_](#lab-67-resource-allocation-and-usage)
  </details>

  <details>
  <summary>hint 3</summary>

  See [lab 6.3](#lab-63-set-pod-security-context) for security context. You will need to add four separate rules for user ID, group ID, privileged and privilege escalation.
  </details>

  <details>
  <summary>hint 4</summary>

  You can use a combination of the output-name and sorting format `kubectl -oname --sort-by=json-path-to-field`. The JSON path can be derived from viewing the resource with output-json `-ojson`. See [_kubectl_ cheatsheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/#viewing-finding-resources) for more details
  </details>
</details>

</div>

### Task - Deployment

Some bootcamp students have been messing with the `webapp` Deployment for the test environment's webpage in the `default` Namespace, leaving it broken. Please rollback the Deployment to the last fully functional version. Once on the fully functional version, update the Deployment to have a total of 10 Pods, and ensure that the total number of old and new Pods, during a rolling update, do not exceed 13 or go below 7.

Update the Deployment to `nginx:1.22-alpine` to confirm the Pod count stays within these thresholds. Then rollback the Deployment to the fully functional version. Before you leave, set the Replicas to 4, and just to be safe, Annotate all the Pods with `description="Bootcamp Test Env - Please Do Not Change Image!"`.

- Command to setup environment:
  ```sh
  printf '\nlab: environment setup in progress...\n'; echo '{"apiVersion":"apps/v1","kind":"Deployment","metadata":{"labels":{"appid":"webapp"},"name":"webapp"},"spec":{"replicas":2,"revisionHistoryLimit":15,"selector":{"matchLabels":{"appid":"webapp"}},"template":{"metadata":{"labels":{"appid":"webapp"}},"spec":{"volumes":[{"name":"varlog","emptyDir":{}}],"containers":[{"image":"nginx:1.12-alpine","name":"nginx","volumeMounts":[{"name":"varlog","mountPath":"/var/logs"}]}]}}}}' > k8s-task-6.yml; kubectl apply -f k8s-task-6.yml >/dev/null; cp k8s-task-6.yml k8s-task-6-bak.yml; sed -i -e 's/nginx:1.12-alpine/nginx:1.13alpine/g' k8s-task-6.yml 2>/dev/null; sed -i '' 's/nginx:1.12-alpine/nginx:1.13alpine/g' k8s-task-6.yml 2>/dev/null; kubectl apply -f k8s-task-6.yml >/dev/null; sleep 1; sed -i -e 's/nginx:1.13alpine/nginx:1.14-alpine/g' k8s-task-6.yml 2>/dev/null; sed -i '' 's/nginx:1.13alpine/nginx:1.14-alpine/g' k8s-task-6.yml 2>/dev/null; kubectl apply -f k8s-task-6.yml >/dev/null; sleep 4; sed -i -e 's/nginx:1.14-alpine/nginx:1.15-alpine/g' k8s-task-6.yml 2>/dev/null; sed -i -e 's/\/var\/logs/\/usr\/share\/nginx\/html/g' k8s-task-6.yml 2>/dev/null; sed -i '' 's/nginx:1.14-alpine/nginx:1.15-alpine/g' k8s-task-6.yml 2>/dev/null; sed -i '' 's/\/var\/logs/\/usr\/share\/nginx\/html/g' k8s-task-6.yml 2>/dev/null; kubectl apply -f k8s-task-6.yml >/dev/null; sleep 2; sed -i -e 's/nginx:1.15-alpine/ngnx:1.16-alpine/g' k8s-task-6.yml 2>/dev/null; sed -i -e 's/\/var\/logs/\/usr\/share\/nginx\/html/g' k8s-task-6.yml 2>/dev/null; sed -i '' 's/nginx:1.15-alpine/ngnx:1.16-alpine/g' k8s-task-6.yml 2>/dev/null; sed -i '' 's/\/var\/logs/\/usr\/share\/nginx\/html/g' k8s-task-6.yml 2>/dev/null; kubectl apply -f k8s-task-6.yml >/dev/null; sleep 4; kubectl apply -f k8s-task-6-bak.yml >/dev/null; sleep 4; kubectl rollout undo deploy webapp --to-revision=5 >/dev/null; kubectl delete $(kubectl get rs --sort-by=".spec.replicas" -oname | tail -n1) >/dev/null; rm k8s-task-6.yml k8s-task-6-bak.yml; echo 'lab: environment setup complete!'
  ```
- Command to destroy environment:
  ```sh
  kubectl delete deploy webapp
  ```

</div>

<div id="hint-task7-1">

<details>
  <summary>hints</summary>

  <details>
  <summary>hint 1</summary>

  ReplicaSets store the Pod configuration used by a Deployment.
  </details>

  <details>
  <summary>hint 2</summary>

  You can reveal more resource details with `kubectl get -owide`. You might be able to find defective Pods/ReplicaSets quicker this way. 
  </details>

  <details>
  <summary>hint 3</summary>

  You will need to review the Deployment's rollout history, see [lab 7.4 - rolling updates](https://github.com/piouson/ckad-bootcamp#lab-74-rolling-updates)
  </details>

  <details>
  <summary>hint 4</summary>

  You can view more details of a rollout revision with `kubectl rollout history --revision=$REVISION_NUMBER`
  </details>

  <details>
  <summary>hint 5</summary>

  Did you test that the Pods are serving an actual webpage? This task isn't complete without testing the webpage - Pods in _Running_ state doesn't mean _fully functional_ version.
  </details>

  <details>
  <summary>hint 6</summary>

  You can test a Pod with `kubectl port-forward`, by creating a temporary Pod `kubectl run --rm -it --image=nginx:alpine -- sh` and running `curl $POD_IP`, etc.
  </details>

  <details>
  <summary>hint 7</summary>

  Always remember `kubectl explain` when you encounter new requirements. Use this to figure out what rolling update parameters are required.  
  </details>

  <details>
  <summary>hint 8</summary>

  You can update a Deployment's image quickly with `kubectl set image --help`. You're not required to count Pods during rolling update, all should be fine long as you have `maxSurge` and `maxUnavailable` set correctly. 
  </details>

  <details>
  <summary>hint 9</summary>

  Any change that triggers a rollout (changing anything under `deploy.spec.template`) will create a new ReplicaSet which becomes visible with `kubectl rollout history`. \
  Be sure to perform updates one after the other, without batching, as an exam question dictates, especially if the changes trigger a rollout. For example, apply replicas and update strategy changes before applying image changes.
  </details>

  <details>
  <summary>hint 10</summary>

  You can set replicas quickly with `kubectl scale --help`.  
  </details>

  <details>
  <summary>hint 11</summary>

  You can Annotate all 4 Pods in a single command, see `kubectl annotate --help`.
  </details>
</details>

</div>

### Task - Service

Create a Pod named `webapp` in the `pig` Namespace (create new if required), running `nginx:1.20-alpine` image. The Pod should have a Annotation `motd="Welcome to Piouson's CKAD Bootcamp"`. Expose the Pod on port 8080.

</div>

<div id="hint-task8-1">

<details>
  <summary>hints</summary>

  <details>
  <summary>hint 1</summary>

  Did you create the Pod in the `pig` Namespace? You should create the Namespace if it doesn't exist.
  </details>

  <details>
  <summary>hint 2</summary>

  You can set Annotation when creating a Pod, see `kubectl run --help`
  </details>

  <details>
  <summary>hint 3</summary>

  Actually, besides creating the Namespace, you can complete the rest of the task in a single command. Completing this task any other way is not but time wasting. Have a deeper look at `kubectl run --help`.
  </details>

  <details>
  <summary>hint 4</summary>

  Did you test you are able to access the app via the Service? This task is not complete until you confirm the application is accessible via the Service.
  </details>

  <details>
  <summary>hint 5</summary>

  You can test the Service by connecting a shell to a temporary Pod `kubectl run -it --rm --image=nginx:alpine -n $NAMESPACE -- sh` and run `curl $SERVICE_NAME:$PORT`. If you did not create the temporary Pod in the same Namespace, you will need to add the Namespace to the hostname `curl $SERVICE_NAME.$NAMESPACE:$PORT`. \
  Testing this way, with Service hostname, is also a way to confirm DNS is working in the cluster.
  </details>
</details>

</div>

<div id="task8-2">

### Task - Service II

A bootcamp student is stuck on a _simple task_ and would appreciate your expertise. Their goal is to create a `webapp` Deployment running `gcr.io/google-samples/node-hello:1.0` image in the `bat` Namespace, exposed on port 80 and _NodePort_ 32500. The student claims _everything_ was setup as explained in class but still unable to access the application via the Service. Swoop down like a superhero and save the day by picking up where the student left off.

- Command to setup environment:
  ```sh
  printf '\nlab: environment setup in progress...\n'; echo '{"apiVersion":"v1","kind":"List","items":[{"apiVersion":"v1","kind":"Namespace","metadata":{"name":"bat"}},{"apiVersion":"apps/v1","kind":"Deployment","metadata":{"labels":{"appid":"webapp"},"name":"webapp","namespace":"bat"},"spec":{"replicas":2,"selector":{"matchLabels":{"appid":"webapp"}},"template":{"metadata":{"labels":{"appid":"webapp"}},"spec":{"containers":[{"image":"gcr.io/google-samples/node-hello:1.0","name":"nginx"}]}}}},{"apiVersion":"v1","kind":"Service","metadata":{"labels":{"appid":"webapp"},"name":"webapp","namespace":"bat"},"spec":{"ports":[{"port":80,"protocol":"TCP","targetPort":80}],"selector":{"app":"webapp"}}}]}' | kubectl apply -f - >/dev/null; echo 'lab: environment setup complete!'
  ```
- Command to destroy environment
  ```sh
  kubectl delete ns bat
  ```

</div>

<div id="hint-task8-2">

<details>
  <summary>hints</summary>

  <details>
  <summary>hint 1</summary>

  Did you check for the relationship between the Service, Endpoint and Pods? When a Service with a _Selector_ is created, an Endpoint with the same name is automatically created. See [_lab 8.1 - connecting applications with services_](#lab-81-connecting-applications-with-services).
  </details>

  <details>
  <summary>hint 2</summary>

  Did you confirm that the Service configuration matches the requirements with `kubectl describe svc`? You should also run some tests, see [_discovering services_](#discovering-services) and [_lab 8.1 - connecting applications with services_](#lab-81-connecting-applications-with-services).
  </details>

  <details>
  <summary>hint 3</summary>

  If you're still unable to access the app but Endpoints have correct IP addresses, you might want to check if there is a working application to begin with. See [_lab 5.1 - creating pods_](#lab-51-creating-pods)
  </details>

  <details>
  <summary>hint 4</summary>

  Now you have the container port? Is the Service configured to use this container port? Is the Pod configured to use this container port? 💡
  </details>

  <details>
  <summary>hint 5</summary>

  Remember a Service can specify three types of ports: `port | targetPort | nodePort`. Which is the container port?
  </details>

  <details>
  <summary>hint 6</summary>

  For a Service, you can quickly verify the configured container port by reviewing the IP addresses of the Service Endpoint, they should be of the form `$POD_IP:CONTAINER_PORT` \
  Once resolved, you should be able to access the application via the Service with `curl`.
  </details>

  <details>
  <summary>hint 7</summary>

  For a Pod, you can quickly verify the configured container port by reviewing the ReplicaSet config with `kubectl describe rs`. \
  Once resolved, you should be able to access the application via the Service with `curl`.
  </details>
</details>

</div>

### Task - Ingress

The application is meant to be accessible at `ckad-bootcamp.local`. Please debug and resolve the issue without creating any new resource.

- Command to setup environment:
  ```sh
  printf '\nlab: environment setup in progress...\n'; echo '{"apiVersion":"v1","kind":"List","items":[{"apiVersion":"v1","kind":"Namespace","metadata":{"name":"bat"}},{"apiVersion":"apps/v1","kind":"Deployment","metadata":{"labels":{"appid":"webapp"},"name":"webapp","namespace":"bat"},"spec":{"replicas":2,"selector":{"matchLabels":{"appid":"webapp"}},"template":{"metadata":{"labels":{"appid":"webapp"}},"spec":{"containers":[{"image":"gcr.io/google-samples/node-hello:1.0","name":"nginx"}]}}}},{"apiVersion":"v1","kind":"Service","metadata":{"labels":{"appid":"webapp"},"name":"webapp","namespace":"bat"},"spec":{"ports":[{"port":80,"protocol":"TCP","targetPort":80}],"selector":{"app":"webapp"}}},{"kind":"Ingress","apiVersion":"networking.k8s.io/v1","metadata":{"name":"webapp","namespace":"bat"},"spec":{"ingressClassName":"ngnx","rules":[{"http":{"paths":[{"path":"/","pathType":"Prefix","backend":{"service":{"name":"webapp","port":{"number":80}}}}]}}]}}]}' | kubectl apply -f - >/dev/null; echo 'lab: environment setup complete!'
  ```
- Command to destroy environment:
  ```sh
  kubectl delete ns dog
  ```

</div>

<div id="task9-2">

### Task - Network policy

Given several Pods in Namespaces `pup` and `cat`, create network policies as follows:
  - Pods in the same Namespace can communicate together
  - `webapp` Pod in the `pup` Namespace can communicate with `microservice` Pod in the `cat` Namespace
  - DNS resolution on UDP/TCP port 53 is allowed for all Pods in all Namespaces

- Command to setup environment:
  ```sh
  printf '\nlab: environment setup in progress...\n'; echo '{"apiVersion":"v1","kind":"List","items":[{"apiVersion":"v1","kind":"Namespace","metadata":{"name":"pup"}},{"apiVersion":"v1","kind":"Namespace","metadata":{"name":"cat"}},{"apiVersion":"v1","kind":"Pod","metadata":{"labels":{"server":"frontend"},"name":"webapp","namespace":"pup"},"spec":{"containers":[{"image":"nginx:1.22-alpine","name":"nginx"}],"dnsPolicy":"ClusterFirst","restartPolicy":"Always"}},{"apiVersion":"v1","kind":"Pod","metadata":{"labels":{"server":"backend"},"name":"microservice","namespace":"cat"},"spec":{"containers":[{"image":"node:16-alpine","name":"nodejs","args":["sleep","7200"]}],"dnsPolicy":"ClusterFirst","restartPolicy":"Always"}}]}' | kubectl apply -f - >/dev/null; echo 'lab: environment setup complete!'
  ```
- Command to destroy environment:
  ```sh
  kubectl delete ns cat pup
  ```

</div>

### Task - Persistent volumes

In the `kid` Namespace (create if required), create a Deployment `webapp` with two replicas, running the `nginx:1.22-alpine` image, that serves an `index.html` HTML document (see below) from the Cluster Node's `/mnt/data` directory. The HTML document should be made available via a Persistent Volume with 5Gi storage and no class name specified. The Deployment should use Persistent Volume claim with 2Gi storage.

<div>
```html
<!-- index.html -->
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=Edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>K8s Bootcamp (CKAD)</title>
  </head>
  <body>
    <h1>Welcome to K8s Bootcamp!</h1>
  </body>
</html>
```

</div>


### Task - ConfigMap and Secrets

The latest Bootcamp cohort have requested a new database in the `rig` Namespace. This should be created as a single replica Deployment named `db` running the `mysql:8.0.22` image with container named `mysql`. The container should start with 128Mi memory and 0.25 CPU but should not exceed 512Mi memory and 1 CPU.

The Resource limit values should be available in the containers as env-vars `MY_CPU_LIMIT` and `MY_MEM_LIMIT` for the values of the cpu limit and memory limit respectively. The Pod IP address should also be available as env-var `MY_POD_IP` in the container.

A Secret named `db-secret` should be created with variables `MYSQL_DATABASE=bootcamp` and `MYSQL_ROOT_PASSWORD="shhhh!"` to be used by the Deployment as the database credentials. A ConfigMap named `db-config` should be used to load the `.env` file (see below) and provide environment variable `DEPLOY_ENVIRONMENT` to the Deployment.

```sh
# .env
DEPLOY_CITY=manchester
DEPLOY_REGION=north-west
DEPLOY_ENVIRONMENT=staging
```

</div>


### Task - Probes

You have a legacy application `legacy` running in the `dam` Namespace that has a long startup time. Once startup is complete, the `/healthz:8080` endpoint returns 200 status. If this application is down at anytime or starting up, this endpoint will return a 500 status. The container port for this application often changes and will not always be `8080`.

Create a probe for the existing Deployment that checks the endpoint every 10secs, for a maximum of 5mins, to ensure that the application does not receive traffic until startup is complete. 20 secs after startup, a probe should continue to check, every 30secs, that the application is up and running, otherwise, the Pod should be killed and restarted anytime the application is down.

You do not need to test that the probes work, you only need to configure them. Another test engineer will perform all tests.

- Command to setup environment:
  ```sh
  printf '\nlab: lab environment setup in progress...\n'; echo '{"apiVersion":"v1","kind":"List","items":[{"apiVersion":"v1","kind":"Namespace","metadata":{"name":"dam"}},{"apiVersion":"apps/v1","kind":"Deployment","metadata":{"labels":{"app":"legacy"},"name":"legacy","namespace":"dam"},"spec":{"replicas":1,"selector":{"matchLabels":{"app":"legacy"}},"template":{"metadata":{"labels":{"app":"legacy"}},"spec":{"containers":[{"args":["/server"],"image":"registry.k8s.io/liveness","name":"probes","ports":[{"containerPort":8080}]}],"restartPolicy":"OnFailure"}}}}]}' | kubectl apply -f - >/dev/null; echo 'lab: environment setup complete!'
  ```
- Command to destroy environment:
  ```sh
  kubectl delete ns dam
  ```

</div>

<div id="task12-2">

### Task - Zero Downtime Updates

In the `hog` Namespace, you will find a Deployment named `high-app`, and a Service named `high-svc`. It is currently unknown if these resources are working together as expected. Make sure the Service is a _NodePort_ type exposed on TCP port 8080 and that you're able to reach the application via the _NodePort_.

Create a single replica Deployment named `high-appv2` based on `high-app.json` file running `nginx:1.18-alpine`.

- Update `high-appv2` Deployment such that 20% of all traffic going to existing `high-svc` Service is routed to `high-appv2`. The total Pods between `high-app` and `high-appv2` should be 5.
- Next, update `high-app` and `high-appv2` Deployments such that 100% of all traffic going to `high-svc` Service is routed to `high-appv2`. The total Pods between `high-app` and `high-appv2` should be 5.

Finally, create a new Deployment named `high-appv3` based on `high-app.json` file running `nginx:1.20-alpine` with 5 replicas and _Pod Template_ label `box: high-app-new`.

- Update `high-svc` Service such that 100% of all incoming traffic is routed to `high-appv3`.
- Since `high-appv2` Deployment will no longer be used, perform a cleanup to delete all Pods related to `high-appv2` only keeping the Deployment and ReplicaSet.

- Command to setup environment (also creates `high-app.json` file):
  ```sh
  printf '\nlab: environment setup in progress...\n'; echo '{"apiVersion":"v1","kind":"List","items":[{"apiVersion":"v1","kind":"Namespace","metadata":{"name":"hog"}},{"apiVersion":"v1","kind":"Service","metadata":{"labels":{"kit":"high-app"},"name":"high-svc","namespace":"hog"},"spec":{"ports":[{"port":8080,"protocol":"TCP","targetPort":8080}],"selector":{"box":"high-svc-child"}}}]}' | kubectl apply -f - >/dev/null; echo '{"apiVersion":"apps/v1","kind":"Deployment","metadata":{"labels":{"kit":"high-app"},"name":"high-app","namespace":"hog"},"spec":{"replicas":4,"selector":{"matchLabels":{"box":"high-app-child"}},"template":{"metadata":{"labels":{"box":"high-app-child"}},"spec":{"containers":[{"image":"nginx:1.15-alpine","name":"nginx","ports":[{"containerPort":80}]}]}}}}' > high-app.json; kubectl apply -f high-app.json  >/dev/null; echo 'lab: environment setup complete!';
  ```
- Command to destroy environment:
  ```sh
  kubectl delete ns hog
  ```

