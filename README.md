# Kubernetes Cheat Sheet

This cheat sheet is written while reading "*Kubernetes: Up and Running* book by Kelsey Hightower, Brendan Burns, and Joe Beda (O’Reilly). This is a great book for those who are starting to learn Kubernetes.

#### Table of Contents

* [Kubernetes Cheat Sheet](#kubernetes-cheat-sheet)
* [Creating and Running Docker Containers](#creating-and-running-docker-containers)
* [Using Minikube](#using-minikube)
* [Working with The Cluster](#working-with-the-cluster)
* [Common kubectl Commands](#common-kubectl-commands)
* [Working with Pods](#working-with-pods)
* [Labels and Annotations](#labels-and-annotations)
* [Service](#service)
* [ReplicaSets](#replicasets)
* [Daemon Set](#daemon-set)
* [Jobs](#jobs)
* [ConfigMaps](#configmaps)
* [Secrets](#secrets)
* [Deployment](#deployment)


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
$ kubectl get [output-options] (TYPE [NAME | -l label] | TYPE/NAME ...) [flags] [options]
```

e.g.:

```bash
$ kubectl get pods [<pod-name>]
```

Output options can be specified with **`-o`** or **`--output`**. Syntax:

`[(-o|--output=)json|yaml|wide|custom-columns=...|custom-columns-file=...|go-template=...|go-template-file=...|jsonpath=...|jsonpath-file=...]`


Option | Description
-------|------------
**`-o wide`** | More detail output
**`-o json`** | JSON output
**`-o yaml`** | YAML output
**`-o jsonpath`** | Get specific field, e.g. `-o jsonpath --template={.status.podIP}`
**`-o template`** | Get specific field, e.g. `-o jsonpath --template={.status.podIP}`

More detailed info:

```bash
$ kubectl describe <resource-type> [<object-name>]
```

Valid resource types include: 
* all  
* certificatesigningrequests (aka 'csr')  
* clusterrolebindings  
* clusterroles  
* componentstatuses (aka 'cs')  
* configmaps (aka 'cm')  
* controllerrevisions  
* cronjobs  
* customresourcedefinition (aka 'crd')  
* daemonsets (aka 'ds')  
* deployments (aka 'deploy')  
* endpoints (aka 'ep')  
* events (aka 'ev')  
* horizontalpodautoscalers (aka 'hpa')  
* ingresses (aka 'ing')  
* jobs  
* limitranges (aka 'limits')  
* namespaces (aka 'ns')  
* networkpolicies (aka 'netpol')  
* nodes (aka 'no')  
* persistentvolumeclaims (aka 'pvc')  
* persistentvolumes (aka 'pv')  
* poddisruptionbudgets (aka 'pdb')  
* podpreset  
* pods (aka 'po')  
* podsecuritypolicies (aka 'psp')  
* podtemplates  
* replicasets (aka 'rs')  
* replicationcontrollers (aka 'rc')  
* resourcequotas (aka 'quota')  
* rolebindings  
* roles  
* secrets  
* serviceaccounts (aka 'sa')  
* services (aka 'svc')  
* statefulsets (aka 'sts')  
* storageclasses (aka 'sc')

#### Creating, Updating, and Deleting Kubernetes Object

To create an object from object manifest in `obj.yaml`:

```bash
$ kubectl apply -f obj.yaml
```

Editing Kubernetes object interactively:

```bash
$ kubectl edit <resource-type> <object-name>
```

Deleting:

```bash
$ kubectl delete -f obj.yaml
```

or

```bash
$ kubectl delete <resource-type> <object-name>
```

#### Label and Annotation

Add label:

```bash
$ kubectl label pods bar color=red
```

Use **`--overwrite`** to change existing label.

Delete label `color` by applying dash suffix:

```bash
$ kubectl label pods bar "color-"
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
  labels:
    app: hello-node
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

## Labels and Annotations

Both labels and annotations are key/value pairs that can be attached to objects. Labels are identifying information, while annotations are not.

Label name can be prefixed by a DNS subdomain, e.g. `acme.com/app-version`.

#### Applying Label

Applying when creating deployment (note: this label is applied to the Deployment object and not the Pods):

```bash
$ kubectl run alpaca-test ... --labels="ver=1,color=red,env=prod"
```

Adding to already running object:

```bash
$ kubectl label pods bar color=red
```

#### Specifying Labels in Manifest

```yaml
...
metadata:
  labels:
    app: wordpress
...
```

#### Showing Labels

```bash
$ kubectl get pods --show-labels
```

#### Showing Label as Column

```bash
$ kubectl get deployments -L color
```

#### Label Selector

Query if a label is set at all:

```bash
$ kubectl get deployments --selector="canary"
```

Query for a particular value:

```bash
$ kubectl get pods --selector="ver=2"
```

Use comma to separate labels:

```bash
$ kubectl get pods --selector="app=bandicoot,ver=2"
```

Query for multiple values:

```bash
$ kubectl get pods --selector="app in (alpaca,bandicoot)"
```

Operators:

  Operator  | Description
------------|------------
`key = value` | Equality
`key != value` | Inequality
`key in (value1, value2)` | Having any of these values
`key notin (value1, value2)` | Not having any of these values
`key` | This key is set
`!key` | This key is not set

#### Defining Annotations

In object definition:

```yaml
...
metadata:
  annotations:
    example.com/icon-url: "https://example.com/icon.png"
...
```

## Service

#### Exposing a Deployment as Service

```bash
$ kubectl expose deployment hello-node
```

#### Defining Service in YAML

```yaml
kind: Service
apiVersion: v1
metadata:
  name: hello-svc
spec:
  type: NodePort
  selector:
    app: hello-node
  ports:
  - protocol: TCP
    port: 5555
    targetPort: 5000
```

When defined without *selector*, service can be used to target other kind of backends (such as database servers).

#### Service Types

Type           | Description
---------------|------------
`ClusterIP`    | (default) Exposes the service on a cluster-internal IP. Choosing this value makes the service only reachable from within the cluster. This is the default ServiceType. 
`NodePort`     |  Exposes the service on each Node’s IP at a static port (the NodePort). A ClusterIP service, to which the NodePort service will route, is automatically created. You’ll be able to contact the NodePort service, from outside the cluster, by requesting <NodeIP>:<NodePort>. 
`LoadBalancer` | builds on NodePort and creates an external load-balancer (if supported in the current cloud) which routes to the clusterIP.
`ExternalName` | Maps the service to the contents of the externalName field (e.g. `foo.bar.example.com`), by returning a CNAME record with its value. No proxying of any kind is set up.


#### Get Service Details

```bash
$ kubectl describe service hello-svc
```

#### Service DNS

Because the cluster IP address of a service is virtual, it is stable and you can map the service address to a DNS entry (how??). For example the service above can be named as `hello-svc.default.svc.cluster.local`, where:

* `hello-svc`: the service name
* `default`: namespace
* `svc`: indicates that this is a service
* `cluster.local`: default base domain

#### Endpoint

No idea what this is..


#### Service without Selector

```yaml
kind: Service
apiVersion: v1
metadata:
  name: external-database
spec:
  type: ExternalName
  externalName: "database.company.com"
```

## ReplicaSets

The doc says in many cases it is recommended to create a Deployment instead of ReplicaSet.

#### Example

```yaml
apiVersion: extensions/v1beta1
kind: ReplicaSet
metadata:
  # Unique key of the ReplicaSet instance
  name: replicaset-example
spec:
  # 3 Pods should exist at all times.
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      # Run the nginx image
      - name: nginx
        image: nginx:1.10
```

#### Spec

The label in template's spec is the specification for pods which are managed by this ReplicaSet.

#### Creating

Use `kubectl apply`.

#### Inspecting

Use `kubectl describe`.

#### Finding a ReplicaSet from a Pod

```bash
$ kubectl get pods <pod-name> -o yaml
```

Check the `kubernetes.io/created-by` annotation.

#### Finding a Set of Pods for a ReplicaSet

Use the `--selector` or `-l` flag:

```bash
$ kubectl get pods -l app=nginx
```

#### Scaling (Imperatively)

```bash
$ kubectl scale replicaset-example --replicas=4
```

#### Scaling (The Right Way)

Change the `replicas` field in the manifest and do `kubectl apply`.

#### Auto Scaling

For example based on CPU usage:

```bash
$ kubectl autoscale rs replicaset-example --min=2 --max=5 --cpu-percent=80
```

#### Deleting

```bash
$ kubectl delete rs replicaset-example
```

Add `--cascade=false` to prevent pod deletion.


## Daemon Set

`DaemonSet` is used to specify pods to be run on each node.

#### Manifest

```yaml
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  # Unique key of the DaemonSet instance
  name: daemonset-example
spec:
  template:
    metadata:
      labels:
        app: daemonset-example
    spec:
      containers:
      # This container is run once on each Node in the cluster
      - name: daemonset-example
        image: ubuntu:trusty
        command:
        - /bin/sh
        args:
        - -c
        # This script is run through `sh -c <script>`
        - >-
          while [ true ]; do
          echo "DaemonSet running on $(hostname)" ;
          sleep 10 ;
          done
```

#### Running

Use `kubectl apply`.

#### Limiting Placement to Certain Nodes

Set the appropriate label on the nodes and use `spec.template.spec.nodeSelector` to select them:

```yaml
spec:
  template:
    metadata:
      labels:
        app: nginx
        ssd: "true"
    spec:
      nodeSelector:
        ssd: "true"
```

#### Updating a DaemonSet by Deleting Individual Pods

```bash
PODS=$(kubectl get pods -o jsonpath -template='{.items[*].metadata.name}'
for x in $PODS; do
  kubectl delete pods ${x}
  sleep 60
done
```

#### Rolling Update of a DaemonSet

Configure the update strategy by setting the `spec.updateStrategy.type` field to `RollingUpdate`. With this, any change to the `spec.template` field (or subfields) in the `DaemonSet` will initiate a rolling update.

#### Deleting a DaemonSet

```bash
$ kubectl delete -f daemonset-example.yaml
```

Add `--cascade=false` to prevent pod deletion.


## Jobs

Jobs are short-lived pods to execute some tasks.

### Running from Command Line

Running a one shot job interactively (the `-i` option):

```bash
$ kubectl run -i example-job --image=.. --restart=OnFailure -- arg1 arg2 ..
```

#### Manifest

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  # Unique key of the Job instance
  name: example-job
spec:
  parallelism: 5
  completions: 10
  template:
    metadata:
      name: example-job
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl"]
        args: ["-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: OnFailure
```

#### Getting Job Info

```bash
$ kubectl describe jobs example-job
```

#### Listing Pods for Running/Have Run Some Jobs

```bash
$ kubectl get pod -l job-name=example-job -a
```

#### Cleaning Jobs

```bash
$ kubectl delete jobs example-job
```

## ConfigMaps

#### Creating from Command Line

Suppose we have a config file `my-config.txt` with the following content:

```
# This is a sample config file that I might use to configure an application
parameter1 = value1
parameter2 = value2
```

Here's to create a ConfigMap:

```bash
$ kubectl create configmap my-config \
   --from-file=my-config.txt \
   --from-literal=extra-param=extra-value \
   --from-literal=another-param=another-value
```

#### Manifest

```bash
$ kubectl get configmaps my-config -o yaml

apiVersion: v1
data:
  another-param: another-value
  extra-param: extra-value
  my-config.txt: |
    # This is a sample config file that I might use to configure an application
    parameter1 = value1
    parameter2 = value2
kind: ConfigMap
metadata:
  creationTimestamp: 2018-06-03T01:13:31Z
  name: my-config
  namespace: default
  resourceVersion: "211863"
  selfLink: /api/v1/namespaces/default/configmaps/my-config
  uid: 5458f599-66cb-11e8-b3a2-08002738b79a
```

#### Listing

```bash
$ kubectl get configmaps
```

#### Using ConfigMap

Three ways:
* mount as file system
* environment variable
* command-line argument

The first two are shown in example below:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-example
spec:
  containers:
    - name: test-container
      image: gcr.io/kuar-demo/kuard-amd64:1
      imagePullPolicy: Always
      command:
        - "/kuard"
        - "$(EXTRA_PARAM)"
      env:
        - name: ANOTHER_PARAM
          valueFrom:
            configMapKeyRef:
              name: my-config
              key: another-param
        - name: EXTRA_PARAM
          valueFrom:
            configMapKeyRef:
              name: my-config
              key: extra-param
      volumeMounts:
        - name: config-volume
          mountPath: /config
volumes:
  - name: config-volume
    configMap:
      name: my-config
restartPolicy: Never
```

#### Updating

If you have the YAML:

```bash
$ kubectl replace -f <filename>
```

If previously applied using `kubectl apply`:

```bash
$ kubectl apply -f <filename>
```

#### Updating Interactively

```bash
$ kubectl edit configmap my-config
```

## Secrets

#### Creating

Putting `example.crt` and `example.key` in the secret:

```bash
$ kubectl create secret generic example-secret --from-file=example.crt --from-file=example.key
```

#### Updating

```bash
$ kubectl create secret generic example-secret \
  --from-file=example.crt --from-file=example.key \
  --dry-run -o yaml | \
  kubectl replace -f -
```

#### Listing

```bash
$ kubectl get secrets
```

#### Getting the Info

```
$ kubectl describe secrets example-secret

Name:        example-tls
Namespace:   default
Labels:      <none>
Annotations: <none>

Type:        Opaque

Data
====
example.crt: 1679 bytes
example.key: 1050 bytes
```

#### Using Secret

Mount as volume:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
    - name: example-pod
      image: ...
      imagePullPolicy: Always
      volumeMounts:
      - name: tls-certs
        mountPath: "/tls"
        readOnly: true
volumes:
  - name: tls-certs
    secret:
      secretName: example-secret
```

#### Saving Private Docker Registry Credential

```bash
$ kubectl create secret docker-registry my-docker-credential \
  --docker-username=<username> \
  --docker-password=<password> \
  --docker-email=<email-address>
```

### Using Private Docker Registry Credential

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example.pod
spec:
  containers:
    - name: example-pod
      image: docker...
      imagePullPolicy: Always
  imagePullSecrets:
    - name: my-image-pull-secret

```

## Deployment

#### Creating from Command Line

```bash
$ kubectl run nginx --image=nginx:1.7.12
```

#### Getting Info

```bash
$ kubectl get deployments nginx
```

#### Getting the ReplicaSet

Getting the ReplicaSet that the deployment manages:

```bash
$ kubectl get replicasets --selector=run=nginx
```

#### Scaling

```bash
$ kubectl scale deployments nginx --replicas=2
```

#### Manifest

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  # Unique key of the Deployment instance
  name: deployment-example
spec:
  # 3 Pods should exist at all times.
  replicas: 3
  # Keep record of 2 revisions for rollback
  revisionHistoryLimit: 10
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: "20%"
  template:
    metadata:
      labels:
        # Apply this label to pods and default
        # the Deployment label selector to this value
        app: nginx
      annotations:
        kubernetes.io/change-cause: "Update nginx to 1.10"
    spec:
      containers:
      - name: nginx
        # Run this image
        image: nginx:1.10
```

#### Getting the Info

Use `kubectl describe`.

#### Updating

Change the YAML and use `kubectl apply -f <filename>`.

You'd better change the annotation to indicate the reason of the update.

#### Rollout Status

```bash
$ kubectl rollout status deployments nginx
```

#### Pausing and Resuming a Rollout

You can pause the rollout if something weird happens:

```bash
$ kubectl rollout pause deployments nginx
```

And to resume it:

```bash
$ kubectl rollout resume deployments nginx
```

#### Rollout History

```bash
$ kubectl rollout history deployment nginx
```

Detailed info for a particular revision:

```bash
$ kubectl rollout history deployment nginx --revision=2
```

#### Rollback Deployment

```bash
$ kubectl rollout undo deployments nginx
```

But this is probably bad idea. Better to change the YAML and apply it.

#### Controlling Rollout

Use:
* `maxUnavailable`
* `minReadySeconds`
* `progressDeadlineSeconds`
