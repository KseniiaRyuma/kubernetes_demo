# Getting Started with Minikube

## Goals

1. Setup all prerequisites for minikube (Docker and Virtualbox)
2. Setup and test Minikube
3. Setup and test kubectl

### Setup Prerequisities

#### Docker:

* Download links for Docker for Windows: https://www.docker.com/docker-windows
* Download links for Docker for Linux: https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu
* Download links for Docker for Mac: https://www.docker.com/docker-mac

Command to test: `docker version`

#### Virtualbox

* Download link: https://www.virtualbox.org/

Command to test: `virtualbox`

### Setup Kubectl:
* Download link: https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-binary-via-curl

Command to test: `kubectl version`


### Setup Minikube:
* General Download Instructions: https://kubernetes.io/docs/tasks/tools/install-minikube/
* Download link: https://github.com/kubernetes/minikube/releases

Command to test: `minikube version`

# Setup Instructions:

## Docker:

* Download links for Docker for Windows: https://www.docker.com/docker-windows
* Download links for Docker for Linux: https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu
* Download links for Docker for Mac: https://www.docker.com/docker-mac

Command to test: 
```
docker version
```

## Kubectl:
* Download link: https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-binary-via-curl

Command to test: 

```
kubectl version
```

## Virtualbox:
* Download link: https://www.virtualbox.org/

## Minikube:
* General Download Instructions: https://kubernetes.io/docs/tasks/tools/install-minikube/
* Download link: https://github.com/kubernetes/minikube/releases

Command to test: 
```
minikube version
```



# Running your first helloworld

## Goals

1. Starting up minikube
2. Set up your first Kubernetes helloworld application
3. Run your application in a minikube environment
4. Expose application via a load balancer and see it running

### Starting up minikube

First, get minikube up and running with the command `minikube start`. This command sets up a Kubernetes dev environment for you via VirtualBox.

The last statement in the output states that kubectl can talk to minikube. We can verify this by running the command `kubectl get nodes`

This will show you that minikube is ready to use.

### Set up your helloworld

We will run one of the most common Docker helloworld applications out there- [https://hub.docker.com/r/karthequian/helloworld/]

To run this, type:

```
kubectl run hw --image=karthequian/helloworld --port=80
```

This command runs a deployment called "hw", pulling from the image karthequian/helloworld, and exposes port 80 of the container to the pod.

Running this command will give you this output, stating that the deployment "hw" was created.

```
MacbookHome:~ karthik$ kubectl run hw --image=karthequian/helloworld --port=80
deployment "hw" created
```

We can run the command `kubectl get all` to see all our resources running, as shown in the output below.

```
MacbookHome:~ karthik$ kubectl get all
NAME                     READY     STATUS    RESTARTS   AGE
po/hw-4212494168-zkv1f   1/1       Running   0          24m

NAME             CLUSTER-IP   EXTERNAL-IP   PORT(S)          AGE
svc/hw           10.0.0.14    <pending>     8080:30218/TCP   7d
svc/kubernetes   10.0.0.1     <none>        443/TCP          7d

NAME        DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deploy/hw   1         1         1            1           24m

NAME               DESIRED   CURRENT   READY     AGE
rs/hw-4212494168   1         1         1         24m
MacbookHome:~ karthik$
```

### Setting up a load balancer
You'll notice that in the `kubectl get all` command, the service has a port mapping defined; however, when you try to hit that port via your web browser, you won't be able to reach the service.

This is because by default, the pod is only accessible by its internal IP address within the cluster. To make the helloworld container accessible from outside the Kubernetes virtual network, you have to expose the pod as a Kubernetes service.

To do this, we can expose the pod to the public internet using the kubectl expose command 
`kubectl expose deployment hw --type=NodePort`

The --type=LoadBalancer flag exposes the deployment outside of the cluster. On cloud providers that support load balancers, an external IP address would be provisioned to access the service.

kubectl expose deployment hello-minikube --type=NodePort

To do this in the minikube environment, the nodeport or loadbalancer type makes the service accessible through the minikube service command.

`minikube service hw1`

This will open your web browser to your application that is running in Kubernetes!

# Scaling the helloworld app

## Goal
1. Learn how to scale the helloworld application

### Learn how to scale the helloworld application

When we run the `kubectl get deployments` command, we notice that there's a single instance of the helloworld deployment running, as well as a single pod associated with the deployment.

There are many scenarios in which this is undesired. If for some reason the pod crashes or ends up in a crash loop, we will not have any instances of the application running, which will cause downtime until the pod can restart successfully.

Fortunately, we can use the inbuilt property of Kubernetes, called replica sets, to solve this issue for deployments. 

To run a replica set for our helloworld deployment, run the command `kubectl scale --replicas=3 deploy/helloworld-deployment`. This will add three replicas for our deployment, which effectively means three pods running for a single deployment.

```
MacbookHome:03_03 Breaking down the helloworld app karthik$ kubectl scale --replicas=3 deploy/helloworld-deployment
deployment "helloworld-deployment" scaled
```

Now, if we run `kubectl get all`, we'll see three pods instead of one.

```
MacbookHome:03_03 Breaking down the helloworld app karthik$ kubectl get all
NAME                                        READY     STATUS    RESTARTS   AGE
po/helloworld-deployment-2148054017-6fc7f   1/1       Running   0          6h
po/helloworld-deployment-2148054017-88nd8   1/1       Running   0          53m
po/helloworld-deployment-2148054017-dvg47   1/1       Running   0          53m

NAME                     CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
svc/helloworld-service   10.0.0.244   <pending>     80:32138/TCP   6h
svc/kubernetes           10.0.0.1     <none>        443/TCP        9d

NAME                           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deploy/helloworld-deployment   3         3         3            3           6h

NAME                                  DESIRED   CURRENT   READY     AGE
rs/helloworld-deployment-2148054017   3         3         3         6h
```



# Adding labels to the application

## Goal
1. Adding labels during build time
2. Viewing labels
2. Adding labels to running pods
3. Deleting a label
4. Searching by labels
5. Extending the label concept to deployments/services

### Adding labels during build time

You can add labels to pods, services and deployments either at build time or at run time. If you're adding labels at build time, you can add a label section in the metadata portion of the YAML as shown below:

```
```

You can deploy the code above by using the command `kubectl create -f helloworld-pod-with-labels.yml`


### Viewing labels

You have a pod with labels. Super! But how do you see them? You can add the `--show-labels` option to your kubectl get command as shown here: `kubectl get pods --show-labels`.

```
```

### Adding labels to running pods
To add labels to a running pod, you can use the `kubectl label` command as follows: `kubectl label po/helloworld app=helloworld`. This adds the label `app` with the value `helloworld` to the pod.

To update the value of a label, use the `--overwrite` flag in the command as follows: `kubectl label po/helloworld app=helloworldapp --overwrite` 

### Deleting a label
To remove an existing label, just add a `-` to the end of the label key as follows: `kubectl label po/helloworld app-`. This will remove the app label from the helloworld pod.

```
```

### Searching by labels
Creating, getting and deleting labels is nice, but the ability to search using labels helps us identify what's going on in our infrastructure better. Let's take a look. First, we're going to deploy a few pods that will constitute what a small org might have. 

`kubectl create -f sample-infrastructure-with-labels.yml`

```
```

Looking at these applications running from a high level makes it hard to see what's going on with the infrastructure.

`kubectl get pods`

`kubectl get pods --show-labels`

```
```

You can search for labels with the flag `--selector` (or `-l`). If you want to search for all the pods that are running in production, you can run `kubectl get pods --selector env=production` as shown below:

`kubectl get pods --selector env=production`


```
```

Similarly, to get all pods by dev lead Karthik, you'd add `dev-lead=karthik` to the selector as shown below.

`kubectl get pods --selector dev-lead=karthik`

```
```

You can also do more complicated searches, like finding any pods owned by Karthik in the development tier, by the following query `dev-lead=karthik,env=staging`:

`kubectl get pods -l dev-lead=karthik,env=staging`

```
```

Or, any apps not owned by Karthik in staging (using the ! construct):

`kubectl get pods -l dev-lead!=karthik,env=staging`

```
```

Querying also supports the `in` keyword

`kubectl get pods -l 'release-version in (1.0,2.0)'`

```
```

Or a more complicated example:

`kubectl get pods -l "release-version in (1.0,2.0),team in (marketing,ecommerce)"`

```
```

The opposite of "in" is "notin". Surprise. "Notin" is supported as well, as shown in this example:

`kubectl get pods -l 'release-version notin (1.0,2.0)'`

```
```

Finally, sometimes your label might not have a value assigned to it, but you can still search by label name.

`kubectl get pods -l 'release-version'`

```
```

### Extending the label concept to deployments/services

As a bonus, labels will also work with `kubectl get services/deployments/all --show-labels` and will return labels for your services, deployments or all objects.

```
```

And, you can delete pods, services or deployments by label as well! For example `kubectl delete pods -l dev-lead=karthik` will delete all pods who's dev-lead was Karthik. 

```
```

To summarize, labels in Kubernetes is a powerful concept! Use the labeling feature to your advantage to build your infrastructure in an organized fashion.
# kubernetes_demo
