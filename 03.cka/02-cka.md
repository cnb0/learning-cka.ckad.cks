# Certified Kubernetes Administrator (CKA) Exam Preparation

```
- Creating a cluster with kubeadm
       Provisioning compute resources
       Install Docker on the hosts
       Install kubeadm, kubelet and kubectl on the hosts
       Initialize the control plane node
       Join the workers
- Control Plane components
      Explore the Control Plane services
- Accessing the cluster
      Install kubectl on your dev machine
      Access the cluster from the dev machine
- Kubernetes Resources
     Namespaces
     Labels and selectors
     Annotations
- The workloads
    Pod specs
    Container Specs
    Pod Controllers
    ReplicaSet controller
    Deployment controller
    Update and Rollback
    Deployment Strategies
- Configuring applications
     Arguments to the command
     Environment Variables
     Configuration file from ConfigMap
     Configuration file from Secret
     Configuration file from Pod fields
     Configuration file from container resources fields
     Configuration file from different sources
- Scaling an application
    Manual scaling
    Auto-scaling
- Application Self-Healing
    Controller to the rescue
    Liveness probes
Resource limits and Quality of Service classes
- Scheduling Pods
    Using label selectors to schedule Pods on specific nodes
    DaemonSets
    Static Pods
    Resource requests
    Running multiple schedulers
- Discovery and Load-balancing
      Services
      Services Types
      Ingress
- Security
     Authentication
     Authorization
     Security Contexts
     Network policies
     Working with private Docker registries
- Storage
    Persistent Volumes
    Claiming a Persistent volume
    Using auto-provisioned persistent volumes
- Monitoring and Logging
    Basic logging
- Upgrading the cluster
      Upgrade the controller
      Upgrade the workers
      Upgrading the operating system
      Backup a cluster
      Restore a cluster
- kubectl
     Managing kubeconfig file
     Generic commands
     Creating applications resources
     Managing clusters
     Getting Documentation

```

## [CKA exam curriculum](https://www.cncf.io/certification/cka/)

 
### 25% - Installation, Configuration & Validation

- [ ] Manage role based access control (RBAC)
- [ ] Use Kubeadm to install a basic cluster
- [ ] Manage a highly-available Kubernetes cluster
- [ ] Provision underlying infrastructure to deploy a Kubernetes cluster
- [ ] Perform a version upgrade on a Kubernetes cluster using Kubeadm
- [ ] Implement etcd backup and restore


### 15%  Workloads & Scheduling

- [ ]  Understand deployments and how to perform rolling update and rollbacks
- [ ] Use ConfigMaps and Secrets to configure applications
- [ ] Know how to scale applications
- [ ] Understand the primitives used to create robust, self-healing, application deployments
- [ ] Understand how resource limits can affect Pod scheduling
- [ ] Awareness of manifest management and common templating tools


### 20%  Services & Networking

- [ ] Understand host networking configuration on the cluster nodes
- [ ] Understand connectivity between Pods
- [ ] Understand ClusterIP, NodePort, LoadBalancer service types and endpoints
- [ ] Know how to use Ingress controllers and Ingress resources
- [ ] Know how to configure and use CoreDNS
- [ ] Choose an appropriate container network interface plugin

### 30% - Troubleshooting

- [ ] Troubleshoot application failure.
- [ ] Troubleshoot control plane failure.
- [ ] Troubleshoot worker node failure.
- [ ] Troubleshoot networking.
- [ ] Evaluate cluster and node logging
- [ ] Understand how to monitor applications
- [ ] Manage container stdout & stderr logs
- [ ] Troubleshoot application failure
- [ ] Troubleshoot cluster component failure
- [ ] Troubleshoot networking

### 10% - Storage

- [ ] Understand storage classes, persistent volumes
- [ ] Understand volume mode, access modes and reclaim policies for volumes
- [ ] Understand persistent volume claims primitive
- [ ] Know how to configure applications with persistent storage


## Scheduling

- [x] [Use label selectors to schedule Pods.](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/)
  * [`nodeSelector`](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector)
    
    > It specifies a map of key-value pairs. For the pod to be eligible to run on a node, the node must have each of the indicated key-value pairs as labels (it can have additional labels as well).

    > ```bash
    > kubectl label nodes <node-name> <label-key>=<label-value>
    > ```

    > ```yaml
    > apiVersion: v1
    > kind: Pod
    > metadata:
    >   name: nginx
    >   labels:
    >     env: test
    > spec:
    >   containers:
    >   - name: nginx
    >     image: nginx
    >     imagePullPolicy: IfNotPresent
    >   nodeSelector:
    >     disktype: ssd
    > ```

    > ##### Interlude: built-in node labels
    > 
    > * `kubernetes.io/hostname`
    > * `failure-domain.beta.kubernetes.io/zone`
    > * `failure-domain.beta.kubernetes.io/region`
    > * `beta.kubernetes.io/instance-type`
    > * `beta.kubernetes.io/os`
    > * `beta.kubernetes.io/arch`

  * [Affinity and anit-affinity](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity)

    > The key enhancements are
    > 1. the language is more expressive (not just “AND of exact match”)
    > 2. you can indicate that the rule is “soft”/”preference” rather than a hard requirement, so if the scheduler can’t satisfy it, the pod will still be scheduled
    > 3. you can constrain against labels on other pods running on the node (or other topological domain), rather than against labels on the node itself, which allows rules about which pods can and cannot be co-located

    * [Node affinity](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#node-affinity-beta-feature)

      > Node affinity is conceptually similar to `nodeSelector` – it allows you to constrain which nodes your pod is eligible to schedule on, based on labels on the node.
      > There are currently two types of node affinity, called 
      > * `requiredDuringSchedulingIgnoredDuringExecution`
      > * `preferredDuringSchedulingIgnoredDuringExecution`
      
      > The “IgnoredDuringExecution” part of the names means that, similar to how `nodeSelector` works, if labels on a node change at runtime such that the affinity rules on a pod are no longer met, the pod will still continue to run on the node.

      > ```yaml
      > spec:
      >   affinity:
      >     nodeAffinity:
      >       requiredDuringSchedulingIgnoredDuringExecution:
      >         nodeSelectorTerms:
      >         - matchExpressions:
      >           - key: kubernetes.io/e2e-az-name
      >             operator: In
      >             values:
      >             - e2e-az1
      >             - e2e-az2
      >       preferredDuringSchedulingIgnoredDuringExecution:
      >       - weight: 1
      >         preference:
      >           matchExpressions:
      >           - key: another-node-label-key
      >             operator: In
      >             values:
      >             - another-node-label-value
      > ```

      > The new node affinity syntax supports the following operators:
      > * `In`,
      > * `NotIn`,
      > * `Exists`,
      > * `DoesNotExist`,
      > * `Gt`,
      > * `Lt`.

      > If you specify both `nodeSelector` and `nodeAffinity`, _both_ must be satisfied for the pod to be scheduled onto a candidate node.

      > If you specify multiple `nodeSelectorTerms` associated with `nodeAffinity` types, then the pod can be scheduled onto a node **if one of** the `nodeSelectorTerms` is satisfied.

      > If you specify multiple `matchExpressions` associated with `nodeSelectorTerms`, then the pod can be scheduled onto a node **only if all** `matchExpressions` can be satisfied.

    * [Inter-pod affinity and anti-affinity (beta feature)](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#inter-pod-affinity-and-anti-affinity-beta-feature)
      
      > Inter-pod affinity and anti-affinity allow you to constrain which nodes your pod is eligible to be scheduled *based on labels on pods that are already running on the node* rather than based on labels on nodes.

      > * `requiredDuringSchedulingIgnoredDuringExecution`
      > * `preferredDuringSchedulingIgnoredDuringExecution`

      > You express it using a `topologyKey` which is the key for the node label that the system uses to denote such a topology domain

      > ```yaml
      > spec:
      >   affinity:
      >     podAffinity:
      >       requiredDuringSchedulingIgnoredDuringExecution:
      >       - labelSelector:
      >           matchExpressions:
      >           - key: security
      >             operator: In
      >             values:
      >             - S1
      >         topologyKey: failure-domain.beta.kubernetes.io/zone
      >     podAntiAffinity:
      >       preferredDuringSchedulingIgnoredDuringExecution:
      >       - weight: 100
      >         podAffinityTerm:
      >           labelSelector:
      >             matchExpressions:
      >             - key: security
      >               operator: In
      >               values:
      >               - S2
      >           topologyKey: kubernetes.io/hostname
      > ```

      > The legal operators for pod affinity and anti-affinity are
      > * `In`,
      > * `NotIn`,
      > * `Exists`,
      > * `DoesNotExist`
- [x] [Understand the role of DaemonSets.](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)
  * [What is a DaemonSet?](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/#what-is-a-daemonset)

    > A *DaemonSet* ensures that all (or some) Nodes run a copy of a Pod. As nodes are added to the cluster, Pods are added to them. As nodes are removed from the cluster, those Pods are garbage collected.

    > Some typical uses of a DaemonSet are:
    > * running a cluster storage daemon, such as `glusterd`, `ceph`, on each node.
    > * running a logs collection daemon on every node, such as `fluentd` or `logstash`.
    > * running a node monitoring daemon on every node, such as Prometheus Node Exporter, `collectd`, Datadog agent, New Relic agent, or Ganglia `gmond`.

  * [Writing a DaemonSet Spec](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/#writing-a-daemonset-spec)

    > ```yaml
    > apiVersion: apps/v1 
    > kind: DaemonSet
    > ```

    > A Pod Template in a DaemonSet must have a `RestartPolicy` equal to `Always`, or be unspecified, which defaults to `Always`.

  * [How Daemon Pods are Scheduled](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/#how-daemon-pods-are-scheduled)

    > If you specify a `.spec.template.spec.nodeSelector`, then the DaemonSet controller will create Pods on nodes which match that node selector. Likewise if you specify a `.spec.template.spec.affinity`, then DaemonSet controller will create Pods on nodes which match that node affinity.
    > * The `unschedulable` field of a node is not respected by the DaemonSet controller.
    > * The DaemonSet controller can make Pods even when the scheduler has not been started, which can help cluster bootstrap.

    > Daemon Pods do respect taints and tolerations, but they are created with `NoExecute` tolerations for the following taints with no `tolerationSeconds`:
    > * `node.kubernetes.io/not-ready`
    > * `node.alpha.kubernetes.io/unreachable`

  * [Updating a DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/#updating-a-daemonset)

    > If node labels are changed, the DaemonSet will promptly add Pods to newly matching nodes and delete Pods from newly not-matching nodes.

    > In Kubernetes version 1.6 and later, you can perform a rolling update on a DaemonSet.

  * [Alternatives to DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/#alternatives-to-daemonset)

    * Init Scripts
    * Bare Pods
    * Static Pods
    * Deployments
- [ ] Understand how resource limits can affect Pod scheduling.
- [ ] Understand how to run multiple schedulers and how to configure Pods to use them.
- [ ] Manually schedule a pod without a scheduler.
- [ ] Display scheduler events.
- [ ] Know how to configure the Kubernetes scheduler.

