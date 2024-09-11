# Deploying Applications Into Kubernetes Cluster

In this project, you will build upon your knowledge of Kubernetes architecture, and begin to deploy applications on a K8s cluster. Kubernetes has a lot of moving parts; it operates with several layers of abstraction between your application and host machines where it runs. So many terms, and capabilities that is not realistic to learn it all at once. Hence, you will be introduced to as many concepts as possible, but gradually.

Within this project we are going to learn and see in action the following:

1. Deployment of software applications using [YAML](https://en.wikipedia.org/wiki/YAML) manifest files with following K8s objects:

- Pods
- ReplicaSets
- Deployments
- StatefulSets
- Services (ClusterIP, NodeIP, Loadbalancer)
- Configmaps
- Volumes
- PersistentVolumes
- PersistentVolumeClaims
...and many more

2. Difference between __stateful__ and __stateless__ applications

- Deploy MySQL as a StatefulSet and explain why

3. Limitations of using manifests directly to deploy on K8s

- Working with [Helm](https://helm.sh/) templates, its components and the most important parts - semantic versioning
Converting all the `.yaml` templates into a helm chart

4. Deploying more tools with Helm charts on AWS Elastic [Kubernetes Service (EKS)](https://aws.amazon.com/eks/)

- Jenkins -MySQL -Ingress Controllers (Nginx)
- Cert-Manager
  - Ingress for Jenkins
  - Ingress for the actual application

5. Deploy Monitoring Tools

- Prometheus
- Grafana

6. Hybrid CI/CD by combining different tools such as: [Gitlab CICD](https://docs.gitlab.com/ee/ci/), Jenkins. And, you will also be introduced to concepts around [GitOps](https://gitops.weave.works/) using [Weaveworks Flux](https://www.weave.works/oss/flux/).

## Choosing the right Kubernetes cluster set up

When it comes to using a Kubernetes cluster, there is a number of options available depending on the ultimate use of it. For example, if you just need a cluster for development or learning, you can use lightweight tools like [Minikube](https://minikube.sigs.k8s.io/docs/start/?arch=%2Fmacos%2Farm64%2Fstable%2Fbinary+download), or [k3s](https://k3s.io/). These tools can run on your workstation without heavy system requirements. Obviously, there is limit to the amount of workload you can deploy there for testing purposes, but it works exactly like any other Kubernetes cluster.

On the other hand, if you need something more robust, suitable for a production workload and with more advanced capabilities such as horizontal scaling of the worker nodes, then you can consider building own Kubernetes cluster from scratch just as you did in the last Project 21. If you have been able to automate the entire bootstrap using Ansible, you can easily spin up your nodes with Terraform, and configure the cluster with your Ansible configuration scripts.

It is a great combination of tools responsible for different parts of your applications:

- Terraform for infrastructure provisioning
- Ansible for cluster master and worker nodes configuration
- Kubernetes for deploying your containerized application and orchestrating the deployment

![](./images/tools-image.png)

Other options will be to leverage a [Managed Service](https://www.wavenet.co.uk/adept) Kubernetes cluster from public cloud providers such as: [AWS EKS](https://aws.amazon.com/eks/), [Microsoft AKS](https://azure.microsoft.com/en-gb/products/kubernetes-service/), or [Google Cloud Platform GKE](https://cloud.google.com/kubernetes-engine). There are so many more options out there. Regardless of whichever one you choose, the experience is usually very similar.

Most organisations choose Managed Service options for obvious reasons such as:

1. Less administrative overheads
2. Reduced cost of ownership
3. Improved Security
4. Seamless support
5. Periodical updates to a stable and well-tested version
6. Faster cluster spin up
... and many more

However, there is usually strong reasons why organisations with very strict compliance and security concerns choose to build their own Kubernetes clusters. Most of the companies that go this route will mostly use on-premises data centres. When there is need to store data privately due to its sensitive nature, companies will rather not use a public cloud provider. Because, if they do, they have no idea of the physical location of the data centre in which their data is being persisted. Banks and Governments are typical examples of this.

Some setup options can combine both public and private cloud together. For example, the master nodes, etcd clusters, and some worker nodes that run [stateful](https://www.techtarget.com/searchsoftwarequality/definition/application) applications can be configured in private datacentres, while worker nodes that require heavy computations and [stateless](https://www.redhat.com/en/topics/cloud-native-apps/stateful-vs-stateless) applications can run in public clouds. This kind of hybrid architecture is ideal to satisfy compliance, while also benefiting from other public cloud capabilities.

![](./images/k8s-hybrid-cloud.png)

## Deploying the Tooling app using Kubernetes objects

In this section, you will begin to write configuration files for Kubernetes objects (they are usually referred as `manifests`) in the form of files with `yaml` syntax and deploy them using `kubectl` console. But first, let us understand what a Kubernetes object is.

__Kubernetes objects__ are persistent entities in the Kubernetes system. Kubernetes uses these entities to represent the state of your cluster. Specifically, they can describe:

- What containerized applications are running (and on which nodes)
- The resources available to those applications
- The policies around how those applications behave, such as restart policies, upgrades, and fault-tolerance

These objects are "record of intent" - once you create the object, the Kubernetes system will constantly work to ensure that the object exists. By creating an object, you are effectively telling the Kubernetes system what you want your cluster's workload to look like; this is your cluster's desired state.

These objects are ___"record of intent"___ - once you create the object, the Kubernetes system will constantly work to ensure that the object exists. By creating an object, you are effectively telling the Kubernetes system what you want your cluster's workload to look like; this is your cluster's desired state.

To work with Kubernetes objects - whether to create, modify, or delete them - you will need to use the Kubernetes API. When you use the `kubectl` command-line interface, for example, the CLI makes the necessary Kubernetes API calls for you. It is also possible to use curl to directly interact with the Kubernetes API, or it can be as part of developing a program in different programming languages. That will require some advance knowledge. You can [read more about client libraries](https://kubernetes.io/docs/reference/using-api/client-libraries/) to get an idea on how that works.

__Common Kubernetes objects__

- Pod
- Namespace
- ResplicaSet (Manages Pods)
- DeploymentController (Manages Pods)
- StatefulSet
- DaemonSet
- Service
- ConfigMap
- Volume
- Job/Cronjob

The very first concept to understand is the difference between how __Docker__ and __Kubernetes__ run containers - with Docker, every `docker run` command will run an image (representing an application) as a container. The running container is a Docker's smallest entity, it is the most basic deployable object. Kubernetes on the other hand operates with `pods` instead of containers, a pods encapsulates a container. Kubernetes uses `pods` as its smallest, and most basic deployable object with a unique feature that allows it to run multiple containers within a single Pod. It is not the most common pattern - to have more than one container in a Pod, but there are cases when this capability comes in handy.

In the world of `docker`, or `docker compose`, to run the __Tooling app__, you must deploy separate containers for the application and the database. But in the world of Kubernetes, you can run both: application and database containers in the same Pod. When multiple containers run within the same Pod, they can directly communicate with each other as if they were running on the same `localhost`. Although running both the application and database in the same Pod is __NOT__ a recommended approach.

A Pod that contains one container is called `single container pod` and it is the most common Kubernetes use case. A Pod that contains multiple co-related containers is called `multi-container pod`. There are few patterns for multi-container Pods; one of them is the [sidecar](https://medium.com/bb-tutorials-and-thoughts/kubernetes-learn-sidecar-container-pattern-6d8c21f873d) container pattern - it means that in the same Pod there is a main container and an auxiliary one that extends and enhances the functionality of the main one without changing it.

There are other patterns, such as: [init container](https://medium.com/bb-tutorials-and-thoughts/kubernetes-learn-init-container-pattern-7a757742de6b), [adapter container](https://medium.com/bb-tutorials-and-thoughts/kubernetes-learn-adaptor-container-pattern-97674285983c), [ambassador container](https://medium.com/bb-tutorials-and-thoughts/kubernetes-learn-ambassador-container-pattern-bc2e1331bd3a). These are more advanced topics that you can study on your own, let us continue with the other objects.

We will not go into the theoretical details of all the objects, rather we will begin to experience them in action.

## Understanding the common YAML fields for every Kubernetes object

Every Kubernetes object includes object fields that govern the object's configuration:

__kind__: Represents the type of kubernetes object created. It can be a `Pod`, `DaemonSet`, `Deployments` or `Service`.
__version:__ Kubernetes api version used to create the resource, it can be `v1`, `v1beta` and `v2`. Some of the kubernetes features can be released under beta and available for general public usage.
__metadata:__ provides information about the resource like name of the Pod, namespace under which the Pod will be running, labels and annotations.
__spec:__ consists of the core information about Pod. Here we will tell kubernetes what would be the expected state of resource, Like container image, number of replicas, environment variables and volumes.
__status:__ consists of information about the running object, status of each container. Status field is supplied and updated by Kubernetes after creation. This is not something you will have to put in the YAML manifest.









