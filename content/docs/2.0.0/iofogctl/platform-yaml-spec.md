# Iofogctl Platform YAML Specification

`iofogctl` allows users to deploy Edge Compute Networks ('ECNs'). The various resources which constitute an ECN are specified within YAML files for iofogctl to consume.

### Note for those upgrading from a previous version

We have updated the yml spec significantly, and while these are small changes in practice, your current yml files will fail to deploy on 2.0

To translate your yml files to the new spec, please see these [quick changes](./translating.html)

## Control Plane

The Control Plane is the overarching component containing specifications for [Controllers](#controller) and associated credentials.

```yaml
apiVersion: iofog.org/v2
kind: ControlPlane # What are we deploying
metadata:
  name: buffalo
  namespace: default
spec:
  iofogUser:
    name: Foo
    surname: Bar
    email: user@domain.com
    password: g9hr823rhuoi
  controllers:
    - name: vanilla
      host: 30.40.50.3
      ssh:
        user: foo
        keyFile: ~/.ssh/id_rsa
        port: 22
    - name: kubernetes
      kube:
        config: ~/.kube/config
        replicas: 2
```

| Field       | Description                                                            |
| ----------- | ---------------------------------------------------------------------- |
| iofogUser   | Credentials registered against ioFog Controller REST API.              |
| controllers | List of Controller instances. See Controller section for more details. |

## Controller

Controllers are components of an ECN responsible for executing the workload of the Control Plane. You can define individuals Controllers for the purposes of expanding your Control Plane. Most of the time, however, you will deploy your Controllers through the Control Plane spec.

```yaml
apiVersion: iofog.org/v2
kind: Controller # What are we deploying
metadata:
  name: alpaca
  namespace: default # Optional, iofogctl namespace to use
spec:
  # Only required for non-K8s deployment
  host: 30.40.50.5
  ssh:
    user: foo
    keyFile: ~/.ssh/id_rsa
    port: 22 # Optional, defaults to 22
  # Only required for K8s deployment
  kube:
    config: ~/.kube/config
    staticIp: 34.23.14.6 # Optional
    replicas: 1 # Optional, defaults to 1
    serviceType: LoadBalancer # Optional, defaults to "LoadBalancer"
```

| Field            | Description                                                                                                                                                              |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| name             | User-defined unique identifier of Controller instance within an iofogctl namespace. Must start and end with lowercase alphanumeric character. Can include '-' character. |
| host             | Hostname of remote host that iofogctl must SSH into to install Controller service.                                                                                       |
| ssh.user         | Username of remote host that iofogctl must SSH into to install Controller service.                                                                                       |
| ssh.keyFile      | Path to private SSH key that iofogctl must use to SSH into remote host to install Controller service.                                                                    |
| ssh.port         | Port to use with SSH                                                                                                                                                     |
| kube.config      | Path to Kubernetes configuration file that iofogctl uses to install Controller service to Kubernetes cluster.                                                            |
| kube.staticIp    | Pre-existing static IP address for Kuberneretes Load Balancer service to use.                                                                                            |
| kube.replicas    | Number of Controller Pods to deploy on Kubernetes cluster.                                                                                                               |
| kube.serviceType | Kubernetes service type for Controller (one of `LoadBalancer`, `NodePort` or `ClusterIP`)                                                                                |

## Agent

Agents are components of an ECN which run on edge nodes. They communicate with Connectors and Controllers to allow your edge nodes to host Microservices.

```yaml
apiVersion: iofog.org/v2
kind: Agent # What are we deploying
metadata:
  name: meerkat
  namespace: default # Optional, iofogctl namespace to use
spec:
  host: 30.40.50.6
  ssh:
    user: foo
    keyFile: ~/.ssh/id_rsa
    port: 22 # Optional, defaults to 22
```

| Field       | Description                                                                                                                                                         |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| name        | User-defined unique identifier of Agent instance within an iofogctl namespace. Must start and end with lowercase alphanumeric character. Can include '-' character. |
| host        | Hostname of remote host that iofogctl must SSH into to install Agent service.                                                                                       |
| ssh.user    | Username of remote host that iofogctl must SSH into to install Agent service.                                                                                       |
| ssh.keyFile | Path to private SSH key that iofogctl must use to SSH into remote host to install Agent service.                                                                    |
| ssh.port    | Port to use with SSH                                                                                                                                                |

## Edge Compute Network

An entire ECN can be specified within a single YAML file.

Multiple resources can be incorporated into a single YAML file using `---` as a separator.

```yaml
---
apiVersion: iofog.org/v2
kind: ControlPlane
metadata:
  name: buffalo
  namespace: default
spec:
  iofogUser:
    name: John
    surname: Doe
    email: user@example.com
    password: mysecretpw
  controllers:
    - name: alpaca-1
      host: 30.40.50.3
      ssh:
        user: john
        keyFile: ~/.ssh/id_rsa
    - name: alpaca-2
      host: 30.40.50.4
      ssh:
        user: john
        keyFile: ~/.ssh/id_rsa
---
apiVersion: iofog.org/v2
kind: Agent
metadata:
  name: hippo-1
  namespace: default
spec:
  host: 30.40.50.6
  ssh:
    user: john
    keyFile: ~/.ssh/id_rsa
---
apiVersion: iofog.org/v2
kind: Agent
metadata:
  name: hippo-2
  namespace: default
spec:
  host: 30.40.50.7
  ssh:
    user: john
    keyFile: ~/.ssh/id_rsa
```

You can also use this approach to deploy a subset of the ECN by omitting any of the Control Plane, Connectors, or Agents sections.

<aside class="notifications note">
  <b>See anything wrong with the document? Help us improve it!</b>
  <a href="https://github.com/eclipse-iofog/iofog.org/edit/develop/content/docs/2.0.0/iofogctl/platform-yaml-spec.md"
    target="_blank">
    <p style="text-align:left">Edit on Github <img src="/images/icos/ico-github.svg" alt=""></p>
  </a>
</aside>