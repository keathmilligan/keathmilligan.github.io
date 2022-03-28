---
layout: post
title: Using Kubernetes for internal infrastructure
date: 2022-03-18 08:30:00 -0500
categories: [dev, devops, testing, automation]
tags: [kubernetes, k8s, docker, containers, microservices]
image: /assets/images/kubernetes-logo.png
---

![K8S dashboard](/assets/images/k8s-dashboard.png)

Tame the chaos of build, SCM, test and other internal development infrastructure with Kubernetes.

<!--more-->

If you work for a medium- to large-sized company (or maybe even a not-so large company), there's a good chance you have a fair amount of infrastructure hosted on internal servers, VMs, or maybe running on a PC sitting under someone's desk (OK hopefully not, but this isn't that uncommon). We can't all be 100% cloud native. This infrastructure could be a part of your software development and/or test pipeline, an internal application or some other process important to your business that either can't be hosted in the cloud or just isn't something that can be easily migrated to the cloud right now. And many organizations have significant legacy systems that aren't likely to make it to the cloud any time soon. So can deploying an on-premises Kubernetes cluster be a step in the right direction?

* TOC
{:toc}

# Internal Infrastructure Challenges

Because this kind of infrastructure tends to evolve over time, there often isn't a deliberate "design" or deployment plan - a need arises, a new server or VM gets stood up, a new problem comes along and another VM is spun up and another and another. Before long, you can end up with a patchwork of servers and VMs cobbled together into a system that is probably working fine until something goes wrong or you need to make a significant change.

## Hand-provisioned servers

One of the biggest challenges with managing internal infrastructure is dealing with individual servers deployed for some purpose or another - like hosting a database, an internal web site or some other application - that were installed and configured by hand. It got the job done at the time, but if the server fails, becomes corrupted or needs to be upgraded, you've got a lot of work on your hands that likely could have been avoided.

And the server running under some guy's desk example is not that unusual. Even if the server isn't literally under a desk, these snowflake servers are still common - hand-configured systems, probably running software that is years out of date, providing some important service that you hope you never have to touch because the person who originally set it up is long gone.

## VM sprawl

Fortunately, most internal infrastructure runs on VMs these days and physical servers are much less of an issue. But virtualization doesn't address many of the challenges of managing internal infrastructure and in fact, introduces a couple of new issues, not the least of which is [VM sprawl](https://whatis.techtarget.com/definition/virtualization-sprawl-virtual-server-sprawl): it's trivially easy to spin up VMs, but much harder to remember to tear them down and relinquish the CPU/memory when they are no longer needed - this leads to large numbers of unused or underutilized virtual machines accumulating and consuming costly resources.

## Un-managed containers

These challenges have many teams to begin experimenting with using containers for internal infrastructure. This is definitely a step in the right direction, but again, it raises a set of new problems:

- How are your containers started?
- What happens if they fail?
- How do you roll out updates?
- How do you manage persistent storage?

When you only have a couple of containers to manage, custom scripts, possibly incorporated into the host server's `init` might be fine for a while and [Docker Compose](https://docs.docker.com/compose/) provides a simple solution for managing multi-container applications. However, as your infrastructure grows beyond what can fit on a single server, these solutions become more difficult to maintain and scale.

## Is Kubernetes right for your infrastructure?

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Deployed my blog on Kubernetes <a href="https://t.co/XHXWLrmYO4">pic.twitter.com/XHXWLrmYO4</a></p>&mdash; For DevOps Eyes Only (@dexhorthy) <a href="https://twitter.com/dexhorthy/status/856639005462417409?ref_src=twsrc%5Etfw">April 24, 2017</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

### On-premises Kubernetes might be overkill for you if:

- Your solution or application consists of only one or two servers.
- Your containers run comfortably on a single server or VM (in this case Docker Compose is probably a better choice).
- Your solution runs entirely (or mostly) inside a CI tool like Jenkins, Gitlab CI or Bamboo.
- Your solution is mostly cloud-based and you have very little on-premises infrastructure.

### Where K8S can help:

- Your infrastructure includes multiple components like web servers, databases, messaging queues, dashboards, custom servers, etc.
- You already have multiple VMs involved in your infrastructure.
- You have multiple containers running all or parts of your solution.
- You are already using Docker Compose or a custom container management solution and are outgrowing it.

# Running an on-premises Kubernetes cluster

While Kubernetes is closely associated with the cloud, it is certainly not limited to the cloud and can be a valuable tool for managing internal infrastructure efficiently and cost effectively.

## Infrastructure-as-Code

Breaking your infrastructure down into separate containers, each providing a part of the solution has a number of advantages including reliability, improved maintainability, separation of concerns and more. One of the secondary benefits is that it naturally leads you to begin deploying and managing your infrastructure from code stored in a revision control system. This provides the benefit of reliable, repeatable deployments of your infrastructure components when you need to add new elements or recover from a failure.

## Efficient resource management

Kubernetes allows you to right-size your deployment - you can freely and easily distribute container workloads across your nodes to eliminate bottlenecks and maximize utilization of each node. Using Kubernetes' built-in resource monitoring tools, you can decide when to add or drop nodes as needed. No more VM sprawl.

## Automated service management

This is of course where Kubernetes really shines:

- Your services are launched automatically and recover if the server or VM needs to be restarted.
- Services are not tied to a particular server or VM. If a server fails, it can be replaced without limited impact to critical services.
- Services can be monitored and can be restarted automatically if they fail.
- Upgrades can be deployed in a controlled manner that is less disruptive.

# Deploying an on-premises K8S cluster

In the past couple of years, several options have matured for deploying Kubernetes locally, including:

- [Minikube](https://minikube.sigs.k8s.io/docs/)
- [Microk8s](https://microk8s.io/)
- [K3S](https://k3s.io/)
- [K0S](https://k0sproject.io/)

All of these are easy to set up and configure and each has its own set of advantages and disadvantages. For this discussion, I will be using [Microk8s](https://microk8s.io/) from Canonical. Microk8s is a full-featured Kubernetes that supports multi-node high-availability and yet remains easy to configure and maintain.

> You may elect to use another Kubernetes distribution, but note that some of the configuration/setup steps below will be different.
{: .warning}

## Caveats

Before getting started, please note a couple of things:

- This post describes deploying a *internal* K8S cluster, if you want to deploy to the cloud, there are a number of other considerations you will need to take into account and you will want to leverage the Kubernetes engine that is supported by the cloud provider (AWS EKS, Google GKS, Azure AKS, etc.).
- Take care that your K8S is not exposed to the internet. The examples here are not intended for public access.

## What you will need

- VMs or physical servers - for this example, we'll create a 3-node cluster. See below for modelling a cluster on your workstation.
- [Ubuntu Server LTS installation media](https://ubuntu.com/download/server)
- [kubectl](https://kubernetes.io/docs/tasks/tools/) installed locally on your workstation.
- [helm](https://helm.sh/) installed locally on your workstation.

## Modelling a cluster on your workstation

If your workstation has a decent about of memory (32GB+), you can easily model a 3 or 4 node cluster with minimal workloads using virtual machines. Several options for hosting VMs include:

- [Microsoft Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/about/) on Windows (built-in to Pro and Server editions)
- [VirtualBox](https://www.virtualbox.org/)
- [Multipass](https://multipass.run/)

# Installation

## Create your virtual machines or servers

- Provision each server with enough memory/CPU for your workloads.
- Each node should be identical unless you plan to create different classes of nodes and limit where workloads run.
- For the simple examples here, a minimal profile should be sufficient for each node (e.g. 2-4GB RAM, 2-4 CPUs).

## OS installation & configuration

- Install OpenSSH and configure as needed for remote access.
- Configure a *static or reserved* IP address for each node. Node IPs should never change under normal operation.

Also, on each node, execute the following command to enable the `iscid` service:

```
sudo systemctl enable iscsid
```

This service is required for persistent storage (see below).

> This might be a good time to use your VM software's "snapshot" or "checkpoint" feature to take a snapshot of each node in case you ever want to roll back to a clean install.
{: .tip}

## Install microk8s

On each node, [install micok8s](https://microk8s.io/docs/getting-started) with:

```
sudo snap install microk8s --classic
```

See the Microk8s [getting started](https://microk8s.io/docs/getting-started) guide for more information on installation.

When the install is completed, check the status with:

```
sudo microk8s status
```

You should see something like:

```
microk8s is running
high-availability: no
  datastore master nodes: 127.0.0.1:19001
  datastore standby nodes: none
addons:
  enabled:
    ha-cluster           # Configure high availability on the current node
  disabled:
    ambassador           # Ambassador API Gateway and Ingress
    ...
```

Next, you can check the status of the workloads in the cluster by running:

```
sudo microk8s.kubectl get all --all-namespaces
```

You should see something like:

```
NAMESPACE     NAME                                           READY   STATUS    RESTARTS   AGE
kube-system   pod/calico-node-rjmjj                          1/1     Running   0          2m26s
kube-system   pod/calico-kube-controllers-65c7fcd667-xllgs   1/1     Running   0          2m27s

NAMESPACE   NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
default     service/kubernetes   ClusterIP   10.152.183.1   <none>        443/TCP   2m33s

NAMESPACE     NAME                         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   daemonset.apps/calico-node   1         1         1       1            1           kubernetes.io/os=linux   2m32s

NAMESPACE     NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/calico-kube-controllers   1/1     1            1           2m32s

NAMESPACE     NAME                                                 DESIRED   CURRENT   READY   AGE
kube-system   replicaset.apps/calico-kube-controllers-65c7fcd667   1         1         1       2m27s
```

If any of the pods fail to start, check the [troubleshooting](https://microk8s.io/docs/troubleshooting) guide.

## Join nodes together

Now we will join the nodes together to form a [highly-available cluster](https://microk8s.io/docs/high-availability).

Designate one of your nodes as the "lead". I have three nodes - `moe`, `larry` and `curly`. You can guess which one is the lead.

On the lead node, run:

```
sudo microk8s add-node
```

You will be given a command to execute on another node to join it to the cluster. If your server has multiple IP addresses, be sure to select the `join` command with the ***static*** IP address. For example, in my case, the `add-node` command returns something like:

```
From the node you wish to join to this cluster, run the following:
microk8s join 172.23.254.108:25000/aa3b7566b89c55f907cca1f814bf6c19/9aabde8eb9a6

Use the '--worker' flag to join a node as a worker not running the control plane, eg:
microk8s join 172.23.254.108:25000/aa3b7566b89c55f907cca1f814bf6c19/9aabde8eb9a6 --worker

If the node you are adding is not reachable through the default interface you can use one of the following:
microk8s join 172.23.254.108:25000/aa3b7566b89c55f907cca1f814bf6c19/9aabde8eb9a6
microk8s join 192.168.2.30:25000/aa3b7566b89c55f907cca1f814bf6c19/9aabde8eb9a6
```

The 172.x.x.x address is not static and not the one I want, so I copy the `join` command that specifies the 192.x.x.x address.

Copy the correct `join` command and run it on one of the other nodes (you'll need to add `sudo`). Wait for the node to join the cluster.

To join the next node, you must go back to the lead node and run the `add-node` command again. Notice that you get a different token in the jon URL. Copy and run the `join` command on the next node to join and wait for it to join the cluster. Repeat this process for each node that you wish to join to the cluster.

On the lead node, run the `sudo microk8s status` command again. You should see the other nodes and that the cluster now has high-availability:

```
microk8s is running
high-availability: yes
  datastore master nodes: 192.168.2.30:19001 192.168.2.31:19001 192.168.2.32:19001
  datastore standby nodes: none
addons:
  enabled:
    ha-cluster           # Configure high availability on the current node
  disabled:
    ambassador           # Ambassador API Gateway and Ingress
    ...
```

See [Create a Microk8s cluster](https://microk8s.io/docs/clustering) for more information on joining nodes.

> This is a good time to use your VM software's "snapshot" or "checkpoint" feature to take a snapshot of each node in case you ever want to roll back to the current state.
{: .tip}

## Enable microk8s add-ons

On the lead node, run:

```
sudo microk8s enable dns ingress metrics-server dashboard openebs
```

This will enable the following microk8s add-ons:

- `dns` - core DNS services
- `ingress` - allows access to services from outside the cluster
- `metrics-server` - provides access to metrics data
- `dashboard` - adds the [Kubernetes dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)

Run the `sudo microk8s.kubectl get all --all-namespaces` command again on the lead node and you will now see quite a few more items:

```
NAMESPACE     NAME                                                  READY   STATUS    RESTARTS      AGE
kube-system   pod/calico-kube-controllers-65c7fcd667-xllgs          1/1     Running   1 (23m ago)   55m
kube-system   pod/calico-node-ptzc5                                 1/1     Running   1 (23m ago)   33m
kube-system   pod/calico-node-2qqjp                                 1/1     Running   2 (22m ago)   36m
kube-system   pod/coredns-64c6478b6c-t6lv9                          1/1     Running   0             6m20s
kube-system   pod/calico-node-5qqh7                                 1/1     Running   2 (22m ago)   36m
ingress       pod/nginx-ingress-microk8s-controller-97q25           1/1     Running   0             5m3s
ingress       pod/nginx-ingress-microk8s-controller-5kk9q           1/1     Running   0             5m3s
ingress       pod/nginx-ingress-microk8s-controller-dwntx           1/1     Running   0             5m3s
kube-system   pod/metrics-server-679c5f986d-s8n7j                   1/1     Running   0             5m3s
kube-system   pod/dashboard-metrics-scraper-69d9497b54-s2djw        1/1     Running   0             3m54s
kube-system   pod/kubernetes-dashboard-585bdb5648-sfr97             1/1     Running   0             3m54s
openebs       pod/openebs-cstor-admission-server-5754659f4b-r7p22   1/1     Running   0             53s
openebs       pod/openebs-ndm-6cljd                                 1/1     Running   0             53s
openebs       pod/openebs-ndm-9kkh9                                 1/1     Running   0             53s
openebs       pod/openebs-jiva-operator-564964cb67-nc25m            1/1     Running   0             53s
openebs       pod/openebs-cstor-cspc-operator-55f9cc6858-lwqdm      1/1     Running   0             53s
openebs       pod/openebs-localpv-provisioner-658895c6c9-hfkfs      1/1     Running   0             53s
openebs       pod/openebs-cstor-cvc-operator-754f9cb6b7-wvrhl       1/1     Running   0             53s
openebs       pod/openebs-ndm-57rph                                 1/1     Running   0             53s
openebs       pod/openebs-cstor-csi-node-xwm5p                      2/2     Running   0             52s
openebs       pod/openebs-jiva-csi-node-gbtjx                       3/3     Running   0             53s
openebs       pod/openebs-ndm-operator-7bd6898d96-hbz6w             1/1     Running   0             53s
openebs       pod/openebs-cstor-csi-node-bb5dc                      2/2     Running   0             52s
openebs       pod/openebs-cstor-csi-controller-0                    6/6     Running   0             53s
openebs       pod/openebs-jiva-csi-node-qnsrc                       3/3     Running   0             53s
openebs       pod/openebs-cstor-csi-node-ttfq8                      2/2     Running   0             52s
openebs       pod/openebs-jiva-csi-controller-0                     5/5     Running   0             53s
openebs       pod/openebs-jiva-csi-node-cjmcs                       3/3     Running   0             53s

NAMESPACE     NAME                                     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes                       ClusterIP   10.152.183.1     <none>        443/TCP                  55m
kube-system   service/kube-dns                         ClusterIP   10.152.183.10    <none>        53/UDP,53/TCP,9153/TCP   6m21s
kube-system   service/metrics-server                   ClusterIP   10.152.183.245   <none>        443/TCP                  5m36s
kube-system   service/kubernetes-dashboard             ClusterIP   10.152.183.47    <none>        443/TCP                  4m59s
kube-system   service/dashboard-metrics-scraper        ClusterIP   10.152.183.93    <none>        8000/TCP                 4m59s
openebs       service/openebs-cstor-cvc-operator-svc   ClusterIP   10.152.183.155   <none>        5757/TCP                 53s
openebs       service/openebs-cstor-admission-server   ClusterIP   10.152.183.43    <none>        443/TCP                  45s

NAMESPACE     NAME                                               DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   daemonset.apps/calico-node                         3         3         3       3            3           kubernetes.io/os=linux   55m
ingress       daemonset.apps/nginx-ingress-microk8s-controller   3         3         3       3            3           <none>                   5m37s
openebs       daemonset.apps/openebs-ndm                         3         3         3       3            3           <none>                   53s
openebs       daemonset.apps/openebs-cstor-csi-node              3         3         3       3            3           <none>                   53s
openebs       daemonset.apps/openebs-jiva-csi-node               3         3         3       3            3           <none>                   53s

NAMESPACE     NAME                                             READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/calico-kube-controllers          1/1     1            1           55m
kube-system   deployment.apps/coredns                          1/1     1            1           6m21s
kube-system   deployment.apps/metrics-server                   1/1     1            1           5m36s
kube-system   deployment.apps/dashboard-metrics-scraper        1/1     1            1           4m59s
kube-system   deployment.apps/kubernetes-dashboard             1/1     1            1           4m59s
openebs       deployment.apps/openebs-cstor-admission-server   1/1     1            1           53s
openebs       deployment.apps/openebs-jiva-operator            1/1     1            1           53s
openebs       deployment.apps/openebs-cstor-cspc-operator      1/1     1            1           53s
openebs       deployment.apps/openebs-localpv-provisioner      1/1     1            1           53s
openebs       deployment.apps/openebs-cstor-cvc-operator       1/1     1            1           53s
openebs       deployment.apps/openebs-ndm-operator             1/1     1            1           53s

NAMESPACE     NAME                                                        DESIRED   CURRENT   READY   AGE
kube-system   replicaset.apps/calico-kube-controllers-65c7fcd667          1         1         1       55m
kube-system   replicaset.apps/coredns-64c6478b6c                          1         1         1       6m20s
kube-system   replicaset.apps/metrics-server-679c5f986d                   1         1         1       5m3s
kube-system   replicaset.apps/dashboard-metrics-scraper-69d9497b54        1         1         1       3m54s
kube-system   replicaset.apps/kubernetes-dashboard-585bdb5648             1         1         1       3m54s
openebs       replicaset.apps/openebs-cstor-admission-server-5754659f4b   1         1         1       53s
openebs       replicaset.apps/openebs-jiva-operator-564964cb67            1         1         1       53s
openebs       replicaset.apps/openebs-cstor-cspc-operator-55f9cc6858      1         1         1       53s
openebs       replicaset.apps/openebs-localpv-provisioner-658895c6c9      1         1         1       53s
openebs       replicaset.apps/openebs-cstor-cvc-operator-754f9cb6b7       1         1         1       53s
openebs       replicaset.apps/openebs-ndm-operator-7bd6898d96             1         1         1       53s

NAMESPACE   NAME                                            READY   AGE
openebs     statefulset.apps/openebs-cstor-csi-controller   1/1     53s
openebs     statefulset.apps/openebs-jiva-csi-controller    1/1     53s
```

## Enable remote access to your cluster

Finally, on the lead node, run `sudo microk8s config` and copy this output to `~/.kubectl/config` on your workstation. If your lead node has more than one IP address, you may need to change the `server` value to reflect the correct static IP.

Now verify that have remote access to your cluster by running the following command on *your workstation*:

```
kubectl get all --all-namespaces
```

You should see the same list as above.

> Keep this configuration data secure. It provides complete control over the cluster and everything running in it.
{: .danger }

# Cluster configuration

## Persistent storage

Containers running in a K8S cluster have no persistent storage by default. When the container stops or is restarted, any data stored in its filesystem is lost. To add persistent storage, you must utilize a storage provider and specify persistent volume information. There are many options for persistent storage for microk8s including:

- The `storage` add-on - this is a simple storage provider that uses the node's local filesystem. It does not replicate volumes across nodes.
- The `openebs` add-on - supports distributed persistent volumes using [OpenEBS](https://openebs.io/).
- [NFS](https://microk8s.io/docs/nfs) - mount NFS volumes for persistent storage.

Other storage options are also possible.

The `openebs` add-on for microk8s is a good option for internal infrastructure. Here is an example of deploying a [mysql](https://hub.docker.com/_/mysql) instance to the cluster using and OpenEBS volume:

**mysql.yaml**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - image: mysql:8.0.26
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: root
        ports:
        - containerPort: 3306
          name: mysql
        resources:
          limits:
            memory: "100M"
            cpu: "0.1"
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: mysql-pv-claim
spec:
  storageClassName: openebs-jiva-csi-default
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5G
---
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  ports:
  - port: 3306
  selector:
    app: mysql
  clusterIP: None
```

Deploy this yaml to the cluster with:
```
kubectl apply -f mysql.yaml
```

OpenEBS will create a persistent volume for our database that is replicated across all nodes. The key is to specify `openebs-jiva-csi-default` in the `storageClassName` field in the persistent volume claim.

## External access to services

You have a number of different options for accessing services running in a cluster - simple port forwarding, using a load-balancer or any of a variety of ways of creating service ingresses.

Create a simple HTTP service:

**echo.yaml**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: echoserver-deployment
  name: echoserver
spec:
  replicas: 1
  selector:
    matchLabels:
      app: echoserver
  template:
    metadata:
      labels:
        app: echoserver
    spec:
      containers:
      - image: k8s.gcr.io/echoserver:1.4
        name: echoserver
        resources:
          limits:
            cpu: "0.1"
            memory: "50M"
---
apiVersion: v1
kind: Service
metadata:
  name: echoserver
spec:
  type: ClusterIP
  selector:
    app: echoserver
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 8080
```

Deploy it with:

```
kubectl apply -f echo.yaml
```

### Port forwarding

[Port forwarding](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/) is a simple way to temporarily gain external access to a service running in a K8S cluster. For example, to access the echo service above:

```
kubectl port-forward deployment/echoserver 18080:8080
```

This will forward local port 18080 to port 8080 of the service. Notice that the command continues to run - it will forward the port until you press **Ctrl-C**.

Open a browser (on the same machine) to `http://localhost:18080` and you should see some information about the request echoed back. Hit **Ctrl-C** to stop forwarding.

Port-forwarding is handy for debugging and temporarily accessing a service but it isn't a practical method of exposing services for regular use.

### Service ingress

An "ingress" is a way of routing requests from outside the cluster to a specific service and port. Using ingresses, you can map your services to a specific external URL and port regardless of what port the service uses internally.

Using the `ingress` add-on for microk8s, create an ingress for the echo service:

**echo-ingress.yaml**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: echo-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "false"
spec:
  rules:
    - http:
        paths:
        - path: /echo(/|$)(.*)
          pathType: Prefix
          backend:
            service:
              name: echoserver
              port:
                number: 80
```

Apply it with `kubectl apply -f echo-ingress.yaml`.

Now you should be able to access the echo service at `http://nodeip/echo`. Where "nodeip" is the address of any of your cluster's nodes. Ingress will take care of routing the request to the correct node regardless of which node the echo service container is actually running on (neat!).

> The simplicity of creating ingresses like this make it a good option for internal infrastructure services where load balancing is not needed.
{: .tip }

You can list ingresses with `kubectl get ingress`. To delete the ingress we just created, use:

```
kubectl delete ingress echo-ingress
```

### Using a load balancer

When a K8S cluster is deployed in a cloud environment, it is most often paired with a load balancer service such as Elastic Load Balancing for AWS (or equivalent offerings from Google, Azure, etc.) and this is the standard method by which services are exposed to the outside. For an on-premises cluster, we will need to deploy an alternative solution.

#### Do you really need a load balancer?

Load balancing is necessary for large scale applications that serve a high rate of requests. Unless you are serving an especially large number of users or requests, most internal dev/test/build infrastructure applications probably do not need a load balancer and a simpler solution like `ingress` above will suffice, avoiding the additional complication of supporting a load balancer.

#### MetalLB

[MetalLB](https://metallb.universe.tf/) is included as a microk8s add-on. To use it, you need to reserve a range of IP addresses that it will control. For example, on the lead cluster node, enter:

```
sudo microk8s enable metallb:192.168.2.100-192.168.2.110
```

This will enable `metallb` with a range of IPs from 192.168.2.100 to 192.168.2.110. See [Metallb microk8s] add-on page for more information on enabling the add-on and the full [MetalLB configuration documentation] for more details on configuration.

Next (back on your workstation) create an ingress service:

**loadbalancer-ingress.yaml**:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: ingress
  namespace: ingress
spec:
  selector:
    name: nginx-ingress-microk8s
  type: LoadBalancer
  # loadBalancerIP is optional. MetalLB will automatically allocate an IP 
  # from its pool if not specified. You can also specify one manually.
  # loadBalancerIP: x.y.z.a
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 80
    - name: https
      protocol: TCP
      port: 443
      targetPort: 443
```

And apply it with `kubectl apply -f loadbalancer-ingress.yaml`.

Now you should be able to access the echo service on the first IP of the specified range (in my case `http://192.168.2.100/echo`).

## Enable access to the dashboard

The [Kubernetes dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/) is a powerful tool for monitoring and managing your K8S cluster. We've enabled it above when we set up the cluster, but by default, external access to it is not provided.

Like any other service, you can expose the dashboard for external access in a variety of ways - port-forwarding, ingress, etc. To enable a permanent ingress for the dashboard, use `kubectl apply` to apply the following:

**dashboard-ingress.yaml**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: dashboard
  namespace: kube-system
  annotations:
    kubernetes.io/ingress.class: public
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
    nginx.ingress.kubernetes.io/rewrite-target: /$2
    nginx.ingress.kubernetes.io/configuration-snippet: |
      rewrite ^(/dashboard)$ $1/ redirect;
spec:
  rules:
  - http:
      paths:
      - path: /dashboard(/|$)(.*)
        pathType: ImplementationSpecific
        backend:
          service:
            name: kubernetes-dashboard
            port:
              number: 443
```

Now you can access the dashboard at `https://clusterip/dashboard`. Where "clusterip" is either your lead node's IP address or the first address in the range passed to MetalLB if you are using the load balancer.

The dashboard will prompt for a token or a kubeconfig file (`~/.kube/config`) for authentication.

![K8S dashboard](/assets/images/k8s-dashboard.png)

> The dashboard gives you the ability to not only monitor your cluster, but also to fully administer it - add/remove/start/stop pods or services, etc. Be sure to take appropriate steps to secure access to it.
{: .warning}

## Enable access to non-HTTP/S services

If you need to provide access to a service that is not HTTP/S based, you can configure the `ingress` service to map TCP/UDP ports. For example, to provide access to port 3306 for MySQL and port 7379 for Redis:

Create and apply the following config map:

**nginx-ingress-tcp-microk8s-conf.yaml:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-ingress-tcp-microk8s-conf
  namespace: ingress
data:
  3306: "default/mysql:3306"
  6379: "default/redis:6379"
```

Next, create a patch file:

**ngnx-ingress-microk8s-controller.patch.yaml:**
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    microk8s-application: nginx-ingress-microk8s
  name: nginx-ingress-microk8s-controller
  namespace: ingress
spec:
  template:
    spec:
      containers:
      - name: nginx-ingress-microk8s
        ports:
        - name: mysql
          containerPort: 3306
          hostPort: 3306
          protocol: TCP
        - name: redis
          containerPort: 6379
          hostPort: 6379
          protocol: TCP
```

This is an update to the `nginx-ingress-microk8s-controller` Daemonset that is already running in your cluster. To apply this patch, run:

```
kubectl patch daemonset nginx-ingress-microk8s-controller --namespace ingress --patch-file nginx-ingress-microk8s-controller.patch.yaml
```

Now you can use a tool like [MySQL Workbench](https://www.mysql.com/products/workbench/) to administer your database from your workstation at `nodeip:3306` (where "nodeip" is the IP address of one of your nodes - note that this will not work from a load balancer address). Obviously, be mindful of the security implications of exposing arbitrary service ports.

# Next steps

Now that you have a functional cluster, you can start containerizing and deploying your services into it. Some things to consider:

- Create [namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) for your applications to make finding and administering them easy. This becomes important when you have many services.
- Use [RBAC authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) to control access to various elements of your cluster - especially if multiple people will be involved in administering it.
- Deploy additional services such as [Grafana](https://grafana.com/) for reporting and dashboards, [Prometheous](https://prometheus.io/) for metrics and/or [Loki](https://grafana.com/oss/loki/) for accessing and querying log data.
- Handle [DockerHub rate limits](https://microk8s.io/docs/dockerhub-limits) in your cluster.
