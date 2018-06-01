# Kubernetes Cheat Sheet

This cheat sheet is written while reading "*Kubernetes: Up and Running* book by Kelsey Hightower, Brendan Burns, and Joe Beda (O’Reilly). This is a great book for those who are starting to learn Kubernetes.

## Creating and Running Docker Containers

#### Sample App

Create a directory in development host (e.g. `$HOME/app`).

Create `app.py`:

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello():
    return "Flask inside Docker!!"


if __name__ == "__main__":
    app.run(debug=True,host='0.0.0.0')
```

Create `requirements.txt`:

```
flask
```

#### Sample Dockerfile

```dockerfile
FROM python:3.5-slim
COPY . /app
WORKDIR /app
RUN pip install -r requirements.txt
EXPOSE 5000
ENTRYPOINT ["python"]
CMD ["app.py"]
```

#### Creating Image

```bash
$ docker build -t hello-node:v1 .
```

#### Listing Images

```bash
$ docker image ls
```

#### Storing In Public Registry

First you need to login:
```bash
$ docker login
```

To Google Container Registry:

```bash
$ docker tag hello-node:v1 gcr.io/<username>/hello-node:v1
$ docker push gcr.io/<username>/hello-node:v1
```

To Docker registry:

```bash
$ docker tag hello-node:v1 <USER ID>/hello-node:v1
$ docker push <USER ID>/hello-node:v1
```

#### Running

To run a local image **locally** (meaning, make sure your terminal isn't "attached" to minikube or a docker-machine):

```bash
$ docker run -p 5555:5000 hello-node:v1
```

To run an image from public registry:

```bash
$ docker run -p 5555:5000 gcr.io/<USER ID>/hello-node:v1
```

To run in the background, add `-d`.

#### Listing Containers

To list running containers:

```bash
$ docker container ls
```

or 

```bash
$ docker ps
```

To list all containers:

```bash
$ docker container ls -a
```

#### Testing

To access the container running **locally** (i.e. not in Minikube nor Docker machine):

```bash
$ curl http://localhost:5555
```

If you built and run the containers while your terminal is attached to Minikube or a Docker machine, then replace *localhost* with the IP address of Minikube (`minikube ip` command) or Docker machine, e.g.:

```bash
$ curl http://192.168.99.100:5555
```


If you're on Windows, I think you need to replace *localhost* with the IP address of your host.
 

#### Stopping Container

```bash
$ docker stop <CONTAINER ID>
```
or

```bash
$ docker container stop <CONTAINER ID>
```

Or if the container is running interactively, just press Ctrl-C. On Windows, you still need to issue `docker container stop <CONTAINER ID>` after hitting Ctrl-C. 

#### Removing Container

```bash
$ docker rm <CONTAINER ID>
```
or 

```bash
$ docker container rm <CONTAINER ID>
```

#### Removing Image

```bash
$ docker rmi <IMAGE>
```

or

```bash
$ docker image rm <IMAGE>
```

where `<IMAGE>` can be their tag names or image IDs.

#### Pruning Unused Images

```bash
$ docker image prune
```

## Using Minikube

#### Starting Minikube

```bash
$ minikube start
```

#### Attaching Terminal to Minikube

If you're following a tutorial on Minikube, instead of pushing your Docker image to a registry, you can simply build the image using the same Docker host as the Minikube VM, so that the images are automatically present. To do so, make sure you are using the Minikube Docker daemon:

```bash
$ eval $(minikube docker-env)
```

To detach:

```bash
$ eval $(minikube docker-env -u)
```

#### Stopping Minikube

```bash
$ minikube stop
```

#### Deleting the Cluster

```bash
$ minikube delete
```

## Working with The Cluster

#### Check Cluster Version

```bash
$ kubectl version
```

#### Simple Diagnostics

```bash
$ kubectl get componentstatuses
```

#### Listing Kubernetes Worker Nodes

```bash
$ kubectl get nodes
```

#### More Information about A Node

```bash
$ kubectl describe nodes <NODE NAME>
```

#### Get Cluster Components Info

Kubernetes proxy:

```bash
$ kubectl get daemonSets --namespace=kube-system kube-proxy
```

DNS:

```bash
$ kubectl get deployments --namespace=kube-system kube-dns
```

Load balancing service for DNS:

```bash
$ kubectl get services --namespace=kube-system kube-dns
```

Kubernetes UI:

```bash
$ kubectl get deployments --namespace=kube-system kubernetes-dashboard
```

Load-balancing service for the dashboard:

```bash
$ kubectl get services --namespace=kube-system kubernetes-dashboard
```

#### Accessing Kubernetes UI

```bash
$ kubectl proxy
```


## Common kubectl Commands

#### Context

Context can be used to set default namespace, authentication, and other settings to be used by `kubectl`.

Create context with a different default namespace for your `kubectl` commands using:

```bash
$ kubectl config set-context my-context --namespace=mystuff
```

then activate it with:

```bash
$ kubectl config use-context my-context
```

#### Get and Describe

General syntax:

```bash
$ kubectl get <resource type> [<object-name>]
```

e.g.:

```bash
$ kubectl get pods [<pod-name>]
```

Options:

Option | Description
-------|------------
**`-o wide`** | More detail output
**`-o json`** | JSON output
**`-o yaml`** | YAML output
**`-o jsonpath`** | Get specific field, e.g. `-o jsonpath --template={.status.podIP}`

More detailed info:

```bash
$ kubectl describe <resource type> [<object-name>]
```

#### Creating, Updating, and Deleting Kubernetes Object

To create an object from object manifest in `obj.yaml`:

```bash
$ kubectl apply -f obj.yaml
```

Editing Kubernetes object interactively:

```bash
$ kubectl edit <resource type> <object-name>
```

Deleting:

```bash
$ kubectl delete -f obj.yaml
```

or

```bash
$ kubectl delete <resource type> <object-name>
```

#### Label and Annotation

Add label:

```bash
$ kubectl label pods bar color=red
```

Use **`--overwrite`** to change existing label.

Delete label `color`:

```bash
$ kubectl label pods bar -color
```

#### Logs

```bash
$ kubectl logs <pod-name>
```

Use **`-f`** to continuously stream the logs back to the terminal without exiting.

#### Accessing Container's Shell

```bash
$ kubectl exec -it <pod-name> -- bash
```

#### Copying Files to/from Container

```bash
$ kubectl cp <pod-name>:/path/to/remote/file /path/to/local/file
```

## Working with Pods

#### Running a Pod from Command Line

```bash
$ kubectl run hello-node --image=gcr.io/<username>/hello-node:v1
```

#### Getting Status

```bash
$ kubectl get pods
```

#### Getting Pod Details

```bash
$ kubectl describe pods hello-node
```

#### Deleting Pod

```bash
$ kubectl delete deployments/hello-node
```

#### Creating Pod Manifest

Create `hello-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello-node
spec:
  containers:
    - image: docker.io/bennylp/hello-node:v1
      name: hello-node
      ports:
        - containerPort: 5000
          name: http
          protocol: TCP
```

#### Running Pod

```bash
$ kubectl apply -f hello-pod.yaml
```


#### Deleting Pod (2)

By name:

```bash
$ kubectl delete pods/hello-node
```

or using the manifest:

```bash
$ kubectl delete -f hello-node.yaml
```

#### Port Forwarding to Access the Pod

```bash
$ kubectl port-forward hello-node 5555:5000
```

You can then access the pod's port 5000 from http://localhost:5555.


#### Liveness Probe

Liveness probe is a health check to see if the container is alive. If the liveness probe fails, the container will be restarted.

Say we create `hello-pod-health.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello-node
spec:
  containers:
    - image: docker.io/bennylp/hello-node:v1
      name: hello-node
      livenessProbe:
        httpGet:
          path: /healthy
          port: 5000
        initialDelaySeconds: 5
        timeoutSeconds: 1
        periodSeconds: 10
        failureThreshold: 3
      ports:
        - containerPort: 5000
          name: http
          protocol: TCP
```

Create the pod with:

```bash
$ kubectl apply -f hello-node-health.yaml
```

Note: you need to handle `/health` HTTP URI path in the application.

#### Readiness Probe

Readiness probe indicates that the container is ready to serve requests. If it fails, the load balancer will not give new request to the container.

Same syntax as liveness probe, but the name is `readinessProbe` instead of `livenessProbe`.


#### Requesting Minimum Required Resource

Add in container's spec in the YAML:

```yaml
...
spec:
  containers:
    - image: ...
      name: ...
      resources:
        requests:
          cpu: "500m"
          memory: "128Mi"
...
```

#### Limiting Resource Usage

```yaml
...
spec:
  containers:
    - image: ...
      name: ...
      resources:
        requests:
          cpu: "500m"
          memory: "128Mi"
        limits:
          cpu: "1000m"
          memory: "256Mi"
...
```

#### Using Volumes

```yaml
spec:
  volumes:
    - name: "my-data"
      hostPath:
        path: "/var/lib/hello-node"
  containers:
    - image: ..
      name: ..
      volumeMounts:
        - mountPath: "/data"
          name: "my-data"
```
