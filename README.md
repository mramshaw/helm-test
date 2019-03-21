# helm-test

Getting familiar with Helm

Helm is way of managing pre-configured software installations - which are known as __charts__ in the Helm community.

Helm builds upon [Kubernetes](http://kubernetes.io/), which itself builds upon [Docker](http://www.docker.com/).

## Motivation

Having thoroughly investigated [Docker](http://github.com/mramshaw/Docker) and [Kubernetes](http://github.com/mramshaw/Kubernetes),
of course I had heard of [Helm](http://helm.sh/).

So it seemed like it was finally time to try it out.

## Prequisites

* Docker
* kubectl
* minikube

Minikube requires a [Hypervisor](http://en.wikipedia.org/wiki/Hypervisor).

In order to be able to run locally, we will use `minikube` for our Kubernetes environment:

```bash
$ kubectl config current-context
minikube
$
```

[This step requires a started `minikube`.]

## Helm Charts

Find stable charts here:

    http://github.com/helm/charts/tree/master/stable

Or at the Helm Hub:

    http://hub.helm.sh

Or as follows:

```bash
$ helm search redis
NAME                            	CHART VERSION	APP VERSION	DESCRIPTION
stable/prometheus-redis-exporter	1.0.2        	0.28.0     	Prometheus exporter for Redis metrics
stable/redis                    	6.4.2        	4.0.14     	Open source, advanced key-value store. It is often referr...
stable/redis-ha                 	3.3.2        	5.0.3      	Highly available Kubernetes implementation of Redis
stable/sensu                    	0.2.3        	0.28       	Sensu monitoring framework backed by the Redis transport
$
```

## Life Cycle

Having chosen a chart to deploy, the life cycle is as follows:

1. [Helm Init](#helm-init)
2. [Helm Version](#helm-version) [Optional]
3. [Helm Repo Update](#helm-repo-update)
4. [Helm Install](#helm-install)
5. [Helm List](#helm-list)
6. [Helm Uninstall](#helm-uninstall)

Steps 2 and 3 can be repeated as necessary. Step 4 should always be preceded by Step 3.

#### Helm Init

This is a one-time requirement and serves to create a local Helm cache.

It should look as follows:

```bash
$ helm init --history-max 200
Creating /home/owner/.helm
Creating /home/owner/.helm/repository
Creating /home/owner/.helm/repository/cache
Creating /home/owner/.helm/repository/local
Creating /home/owner/.helm/plugins
Creating /home/owner/.helm/starters
Creating /home/owner/.helm/cache/archive
Creating /home/owner/.helm/repository/repositories.yaml
Adding stable repo with URL: https://kubernetes-charts.storage.googleapis.com
Adding local repo with URL: http://127.0.0.1:8879/charts
$HELM_HOME has been configured at /home/owner/.helm.

Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.

Please note: by default, Tiller is deployed with an insecure 'allow unauthenticated users' policy.
To prevent this, run `helm init` with the --tiller-tls-verify flag.
For more information on securing your installation see: https://docs.helm.sh/using_helm/#securing-your-helm-installation
Happy Helming!
$
```

#### Helm Version

Use <kbd>helm version</kbd> to check the local version of Helm.

In order to verify that `helm` is installed and working, we will check it's version:

```bash
$ helm version
Client: &version.Version{SemVer:"v2.13.0", GitCommit:"79d07943b03aea2b76c12644b4b54733bc5958d6", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.13.0", GitCommit:"79d07943b03aea2b76c12644b4b54733bc5958d6", GitTreeState:"clean"}
$
```

#### Helm Repo Update

Just to be on the safe side, lets update our list of charts:

```bash
$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Skip local chart repository
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈ Happy Helming!⎈
$
```

[This should be carried out from time to time.]

#### Helm Install

Use <kbd>helm install</kbd> to release a Helm Chart.

This should look something like the following (depending upon the chart):

```bash
$ helm install stable/redis
NAME:   crazy-kudu
LAST DEPLOYED: Thu Mar 21 12:20:56 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME                     DATA  AGE
crazy-kudu-redis         3     0s
crazy-kudu-redis-health  3     0s

==> v1/Pod(related)
NAME                                     READY  STATUS             RESTARTS  AGE
crazy-kudu-redis-master-0                0/1    ContainerCreating  0         0s
crazy-kudu-redis-slave-767ccd5df4-bsm9v  0/1    ContainerCreating  0         0s

==> v1/Secret
NAME              TYPE    DATA  AGE
crazy-kudu-redis  Opaque  1     0s

==> v1/Service
NAME                     TYPE       CLUSTER-IP      EXTERNAL-IP  PORT(S)   AGE
crazy-kudu-redis-master  ClusterIP  10.104.133.165  <none>       6379/TCP  0s
crazy-kudu-redis-slave   ClusterIP  10.110.217.7    <none>       6379/TCP  0s

==> v1beta1/Deployment
NAME                    READY  UP-TO-DATE  AVAILABLE  AGE
crazy-kudu-redis-slave  0/1    1           0          0s

==> v1beta2/StatefulSet
NAME                     READY  AGE
crazy-kudu-redis-master  0/1    0s


NOTES:
** Please be patient while the chart is being deployed **
Redis can be accessed via port 6379 on the following DNS names from within your cluster:

crazy-kudu-redis-master.default.svc.cluster.local for read/write operations
crazy-kudu-redis-slave.default.svc.cluster.local for read-only operations


To get your password run:

    export REDIS_PASSWORD=$(kubectl get secret --namespace default crazy-kudu-redis -o jsonpath="{.data.redis-password}" | base64 --decode)

To connect to your Redis server:

1. Run a Redis pod that you can use as a client:

   kubectl run --namespace default crazy-kudu-redis-client --rm --tty -i --restart='Never' \
    --env REDIS_PASSWORD=$REDIS_PASSWORD \
   --image docker.io/bitnami/redis:4.0.14 -- bash

2. Connect using the Redis CLI:
   redis-cli -h crazy-kudu-redis-master -a $REDIS_PASSWORD
   redis-cli -h crazy-kudu-redis-slave -a $REDIS_PASSWORD

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace default svc/crazy-kudu-redis 6379:6379 &
    redis-cli -h 127.0.0.1 -p 6379 -a $REDIS_PASSWORD

$
```

Note that the installation is called `crazy-kudu` and that our ___pods___ are starting.

Also, the installation displays a number of useful comments (these vary depending upon the
 chart, and may not always be present).

[It may take some time before everything is up and running.]

#### Helm List

Use <kbd>helm ls</kbd> to see what has been released.

This should look as follows:

```bash
$ helm ls
NAME      	REVISION	UPDATED                 	STATUS  	CHART      	APP VERSION	NAMESPACE
crazy-kudu	1       	Thu Mar 21 12:20:56 2019	DEPLOYED	redis-6.4.2	4.0.14     	default
$
```

[Like Docker, Helm will randomly assign a name to the installation if one is not specified.]

#### Helm Uninstall

Use <kbd>helm delete</kbd> to uninstall a Helm Chart.

This should look something like the following (depending upon the chart):

```bash
$ helm delete crazy-kudu
release "crazy-kudu" deleted
$
```

## Preparation for Rancher

[Rancher](http://rancher.com/) offers a nice GUI for Multi-Cluster Kubernetes Management.

Under the covers Rancher actually uses `helm`, but sometimes a dashboard is handy.

To allow for the possible use of [Rancher](http://rancher.com/) we will add their __stable__ charts:

    $ helm repo add rancher-stable https://releases.rancher.com/server-charts/stable

This should look as follows:

```bash
$ helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
"rancher-stable" has been added to your repositories
$
```

## Versions

* Docker (Client and Server) - __18.09.3__
* helm __v2.13.0__
* kubectl __v1.10.7__
* Kubernetes __v1.13.4__
* minikube __v0.35.0__
* virtualbox __5.1.38__

## To Do

- [ ] Further testing
