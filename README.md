# helm-test

![Helm Logo](images/helm-logo.png)

Getting familiar with Helm

Helm is way of managing pre-configured software installations - which are known as __charts__ in the Helm community.

Helm builds upon [Kubernetes](http://kubernetes.io/), which itself builds upon [Docker](http://www.docker.com/).

As of March, 2019 Helm is a [Cloud Native Computing Foundation (CNCF)](https://www.cncf.io/) incubation project
(Kubernetes was the first project to ___graduate___ from CNCF incubation, in March 2018).

## Motivation

Having thoroughly investigated [Docker](http://github.com/mramshaw/Docker) and [Kubernetes](http://github.com/mramshaw/Kubernetes),
of course I had heard of [Helm](http://helm.sh/).

So it seemed like it was finally time to try it out.

## Contents

The contents are as follows:

* [Prequisites](#prequisites)
* [Helm Components](#helm-components)
* [Installing Helm](#installing-helm)
* [Helm Charts](#helm-charts)
    * [Directory Structure](#directory-structure)
    * [Adding Bitnami Charts](#adding-bitnami-charts)
    * [Finding Helm Charts](#finding-helm-charts)
    * [Inspecting Helm Charts](#inspecting-helm-charts)
* [Development Life Cycle](#development-life-cycle)
    * [helm create](#helm-create)
    * [helm dep update](#helm-dep-update)
    * [helm lint](#helm-lint)
    * [helm test](#helm-test)
    * [helm package](#helm-package)
    * [helm ls](#helm-ls)
    * [helm delete](#helm-delete)
* [Deployment Life Cycle](#deployment-life-cycle)
    * [Helm Initialize](#helm-initialize)
    * [Helm Version](#helm-version)
    * [Helm Repo Update](#helm-repo-update)
    * [Helm Install](#helm-install)
    * [Helm List](#helm-list)
    * [Helm Uninstall](#helm-uninstall)
* [Preparation for Rancher](#preparation-for-rancher)
* [Versions](#versions)
* [Reference](#reference)
* [To Do](#to-do)

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

## Helm Components

Helm consists of two components:

1. `helm`
2. `tiller`

[Following the Kubernetes ("helmsman") metaphor, we have a helm and one or more tillers.]

The client, `helm`, is used to manage Helm charts and to manage one or more tillers.
The server, `tiller`, runs inside one or more Kubernetes clusters and manages the Helm releases.

## Installing Helm

Instructions for installing Helm may be found at:

    http://docs.helm.sh/using_helm/#installing-helm

In general it is only necessary to install `helm` as helm itself will deploy `tiller` as needed.

## Helm Charts

Helm Charts define Kubernetes resources that will be released to Kubernetes Clusters.

The `Chart.yaml` file should provide a high-level summary of the chart.

Important values (apart from secrets) should be stored in the `values.yaml` file.

Useful runtime information should go in the `templates/Notes.txt` file (these notes
will be displayed once the chart is installed).

#### Directory structure

Chart directories should have roughly the following structure:

	package-name/
	  templates/
	  .helmignore
	  Chart.yaml
	  OWNERS
	  LICENSE
	  README.md
	  requirements.lock
	  requirements.yaml
	  values.yaml

The `requirements.yaml` and `requirements.lock` files may or may not be needed.

Much like a `requirements.txt` file in a Python project, the `requirements.yaml`
file is used to list project dependencies (if there are any - which there may not be).

Running a <kbd>helm dep update mychart</kbd> command will generate a `requirements.lock`
file (which should be checked into any code repositories along with the `requirements.yaml`
file).

#### Adding Bitnami Charts

In order to be able to search the Bitnami charts, we will add their chart repo.

This should look as follows:

```bash
$ helm repo add bitnami https://charts.bitnami.com
"bitnami" has been added to your repositories
$
```

We can see the Bitnami charts as follows:

    $ helm search bitnami

And now our search results will include the Bitnami charts.

#### Finding Helm Charts

Find stable charts here:

    http://github.com/helm/charts/tree/master/stable

Or search the Bitnami charts (many of which have been upstreamed to helm/charts):

    https://github.com/bitnami/charts

Or at the Helm Hub:

    http://hub.helm.sh

Or as follows:

```bash
$ helm search redis
NAME                            	CHART VERSION	APP VERSION	DESCRIPTION
bitnami/redis                   	6.4.3        	4.0.14     	Open source, advanced key-value store. It is often referr...
stable/prometheus-redis-exporter	1.0.2        	0.28.0     	Prometheus exporter for Redis metrics
stable/redis                    	6.4.2        	4.0.14     	Open source, advanced key-value store. It is often referr...
stable/redis-ha                 	3.3.2        	5.0.3      	Highly available Kubernetes implementation of Redis
stable/sensu                    	0.2.3        	0.28       	Sensu monitoring framework backed by the Redis transport
$
```

[Note that the `bitnami/redis` chart seems to be more up-to-date than the `stable/redis` chart.]

#### Inspecting Helm Charts

Helm Chart values can be overridden at `helm install` time via the <kbd>--set</kbd> option.

Helm can be used to inspect Helm Charts via <kbd>helm inspect</kbd>.

[If the Helm chart is not available in the local cache, helm will fetch it and store it in the cache.]

This should look as follows:

```bash
$ helm inspect values stable/redis
## Global Docker image parameters
## Please, note that this will override the image parameters, including dependencies, configured to use the global value
## Current available global Docker image parameters: imageRegistry and imagePullSecrets
##
# global:
#   imageRegistry: myRegistryName
#   imagePullSecrets:
#     - myRegistryKeySecretName

<...>

$
```

## Development Life Cycle

Helm can be used to ___create___ Helm Charts:

1. <kbd>helm create mychart</kbd>
5. [Optional] <kbd>helm dep update mychart</kbd>
2. <kbd>helm lint mychart</kbd>
2. <kbd>helm test mychart</kbd>
3. <kbd>helm package mychart</kbd>
4. <kbd>helm install mychart</kbd>
5. [Optional] <kbd>helm update mychart</kbd>
6. [Optional] <kbd>helm delete mychart</kbd>
7. [Optional] <kbd>helm rollback mychart</kbd>
8. [Optional] <kbd>helm ls --deleted</kbd>
9. [Optional] <kbd>helm delete --purge mychart</kbd>

#### helm create

The __create__ command can be used to create a base set of Chart files.

This should look as follows:

```bash
$ helm create test-chart
Creating test-chart
$
```

The files that `helm create` will produce may be viewed at [/test-chart/](test-chart).

The __create__ command can take an optional <kbd>--starter</kbd> option for specifying a "starter chart".

Starter Charts are regular charts, but in template form - and must be stored in the `~/.helm/starters/` directory.

#### helm dep update

This step is only needed if there are any project dependencies. These may be specified
in a `requirements.yaml` file.

Running a <kbd>helm dep update mychart</kbd> command will then generate a `requirements.lock`
file (this file should be checked into any code repositories along with the `requirements.yaml`
file).

#### helm lint

The __lint__ command can be used to check for possible errors and/or omissions.

For our simple test chart, this looks as follows:

```bash
$ helm lint test-chart
==> Linting test-chart
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, no failures
$
```

[As our chart has no serious issues, we can "dry run" an install (requires a started `minikube`):

```bash
$ helm install --dry-run --debug test-chart
[debug] Created tunnel using local port: '42227'

[debug] SERVER: "127.0.0.1:42227"

[debug] Original chart version: ""
[debug] CHART PATH: /home/owner/Documents/Kubernetes/Helm/helm-test/test-chart

NAME:   nobby-parrot
REVISION: 1
RELEASED: Mon Mar 25 09:06:57 2019
CHART: test-chart-0.1.0
USER-SUPPLIED VALUES:

<...>

$
```

[As our chart will install, we can install and test it.]

    $ helm install --name mychart --debug test-chart

This should look as follows:

```bash
$ helm install --name mychart --debug test-chart
[debug] Created tunnel using local port: '46214'

[debug] SERVER: "127.0.0.1:46214"

[debug] Original chart version: ""
[debug] CHART PATH: /home/owner/Documents/Kubernetes/Helm/helm-test/test-chart

NAME:   mychart
REVISION: 1
RELEASED: Mon Mar 25 09:41:00 2019
CHART: test-chart-0.1.0

<...>

NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=test-chart,app.kubernetes.io/instance=mychart" -o jsonpath="{.items[0].metadata.name}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl port-forward $POD_NAME 8080:80

$
```

#### helm test

Once our chart is installed, we can test it:

    $ helm test my-chart

This should look as follows:

```bash
$ helm test mychart
RUNNING: mychart-test-chart-test-connection
PASSED: mychart-test-chart-test-connection
$
```

#### helm package

Once we have successfully tested our chart, we can bundle it up for community use.

The __package__ command creates a `.tgz` archive of the chart.

This should look as follows:

```bash
$ helm package test-chart
Successfully packaged chart and saved it to: /home/owner/Documents/Kubernetes/Helm/helm-test/test-chart-0.1.0.tgz
$
```

[The actual file location itself will depend upon where your code is located.]

This archive may be examined at [test-chart-0.1.0.tgz](test-chart-0.1.0.tgz).

Helm tracks revisions so that updates can be rolled-back.

#### helm ls

The __ls__ or __list__ command can take an optional <kbd>--deleted</kbd> option for viewing uninstalled charts.

#### helm delete

The __delete__ [Uninstall] command can take an optional <kbd>--purge</kbd> option for purging uninstalled charts.

## Deployment Life Cycle

Much like Docker containers, Helm Charts have a ___life cycle___.

Having chosen a chart to deploy, the life cycle is as follows:

1. [Helm Initialize](#helm-initialize)
2. [Helm Version](#helm-version) [Optional]
3. [Helm Repo Update](#helm-repo-update)
4. [Helm Install](#helm-install)
5. [Helm List](#helm-list)
6. [Helm Uninstall](#helm-uninstall)

Steps 2 and 3 can be repeated as necessary. Step 4 should always be preceded by Step 3.

#### Helm Initialize

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

The `--history-max` option is specified to prevent the helm history from growing too large.

Note that this propogates `tiller` to the Kubernetes Cluster.

We can query for the presence of a running `tiller` as follows (requires a started `minikube`):

```bash
$ kubectl get deployments --namespace kube-system
NAME                   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
coredns                2         2         2            2           3d
kubernetes-dashboard   1         1         1            1           3d
tiller-deploy          1         1         1            1           3d
$
```

And there we see a running __tiller-deploy__.

If `tiller` should __not__ be deployed, this may be specified as follows:

    $ helm init --client-only

Or:

    $ helm init -c

#### Helm Version

Use <kbd>helm version</kbd> to check the local version of Helm.

In order to verify that `helm` is installed and working, we will check it's version:

```bash
$ helm version
Client: &version.Version{SemVer:"v2.13.0", GitCommit:"79d07943b03aea2b76c12644b4b54733bc5958d6", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.13.0", GitCommit:"79d07943b03aea2b76c12644b4b54733bc5958d6", GitTreeState:"clean"}
$
```

[This step requires a started `minikube` - or else the Server will be unreachable.]

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

Also, the installation displays a number of useful comments (these vary depending upon the chart).

[It may take some time before everything is up and running.]

The __install__ command also has <kbd>--dry-run</kbd> and <kbd>--debug</kbd> options.

The __install__ command also allows for overriding default values via the <kbd>--set</kbd> option, such as:

    $ helm install testchart --set replicaCount=3

#### Helm List

Use <kbd>helm list</kbd> or <kbd>helm ls</kbd> (the short form) to see what has been released.

This should look as follows:

```bash
$ helm ls
NAME      	REVISION	UPDATED                 	STATUS  	CHART      	APP VERSION	NAMESPACE
crazy-kudu	1       	Thu Mar 21 12:20:56 2019	DEPLOYED	redis-6.4.2	4.0.14     	default
$
```

[Like Docker, Helm will randomly assign a name to the installation if one is not specified.]

The details for this redis chart may be found at:

    http://github.com/helm/charts/tree/master/stable/redis

For information purposes, the actual [Chart](Chart.yaml) from this repo is included here.

Use <kbd>helm ls --deleted</kbd> to see what has been uninstalled:

```bash
$ helm ls --deleted
NAME      	REVISION	UPDATED                 	STATUS 	CHART      	APP VERSION	NAMESPACE
crazy-kudu	1       	Thu Mar 21 12:20:56 2019	DELETED	redis-6.4.2	4.0.14     	default
$
```

#### Helm Uninstall

Use <kbd>helm delete</kbd> to uninstall a Helm Chart.

This should look something like the following (depending upon the chart):

```bash
$ helm delete crazy-kudu
release "crazy-kudu" deleted
$
```

Once the release has been uninstalled, it will show as `DELETED`.

[The release may still be rolled-back at this point.]

Use <kbd>helm delete --purge</kbd> to completely purge the uninstalled Helm Chart.

This should look as follows:

```bash
$ helm delete --purge crazy-kudu
release "crazy-kudu" deleted
$
```

[The release can no longer be rolled-back.]

## Preparation for Rancher

[Rancher](http://rancher.com/) offers a nice GUI for Multi-Cluster Kubernetes Management.

Under the covers Rancher actually uses `helm`, but sometimes a dashboard is handy.

To allow for the use of [Rancher](http://github.com/mramshaw/rancher-test) we will add their __stable__ charts:

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

## Reference

Installing Helm:

    http://docs.helm.sh/using_helm/#installing-helm

Helm Charts:

    http://github.com/helm/helm/blob/master/docs/charts.md

[This should be considered definitive.]

Creating Helm Charts:

    http://docs.bitnami.com/kubernetes/how-to/create-your-first-helm-chart/

Testing Helm Charts:

    http://github.com/helm/helm/blob/master/docs/chart_tests.md

[This should be considered definitive.]

Signing Helm Charts:

    http://github.com/helm/helm/blob/master/docs/provenance.md

[This should be considered definitive.]

## To Do

- [ ] Further testing
