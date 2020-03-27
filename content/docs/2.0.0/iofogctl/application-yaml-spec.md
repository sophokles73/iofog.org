# Iofogctl Application YAML specification

`iofogctl` allows users to deploy sets of Microservices to Edge Compute Networks ('ECNs'). The various components of Microservices are specified within YAML files for `iofogctl` to consume.

### Note for those upgrading from a previous version

We have updated the yml spec significantly, and while these are small changes in practice, your current yml files will fail to deploy on 2.0.0

To translate your yml files to the new spec, please see these [quick changes](./translating.html)

## Application

An application is a set of Microservices working together to achieve one specific purpose.

An application is defined by a YAML file. This file is passed as a parameter to the deploy command: `iofogctl deploy -f <path-to-yaml>`

An application YAML file definition can be retrieved with the describe command: `iofogctl describe application <NAME> [-o <path-to-yaml>]`

Don't panic if this seems like a lot to digest, the [microservice yaml definition](#microservices) is explained in more details further down.
The main take away is that an application is defined by: a `name`, a set of `microservices` and a set of `routes`.

```yaml
apiVersion: iofog.org/v2
kind: Application # What are we deploying
metadata:
  name: Healthcare Wearable # Application name
  namespace: default # (Optional) iofogctl namespace to use

# Specifications of the application
spec:
  # List of microservices composing your application
  microservices:
    #  # It uses the microservice YAML schema described below
    - name: 'heart-rate-monitor'
      agent:
        name: 'horse-1'
      images:
        arm: 'edgeworx/healthcare-heart-rate:arm-v1'
        x86: 'edgeworx/healthcare-heart-rate:x86-v1'
        # registry: remote
      container:
        rootHostAccess: false
        ports: []
      config:
        test_mode: true
        data_label: 'Anonymous Person'
        nested_object:
          key: 42
          deep_nested:
            foo: bar
    # Simple JSON viewer for the heart rate output
    - name: 'heart-rate-viewer'
      agent:
        name: 'horse-1'
      images:
        arm: 'edgeworx/healthcare-heart-rate-ui:arm'
        x86: 'edgeworx/healthcare-heart-rate-ui:x86'
        registry: remote
      container:
        rootHostAccess: false
        ports:
          # The ui will be listening on port 80 (internal).
          - external: 5000 # You will be able to access the ui on <AGENT_IP>:5000
            internal: 80 # The ui is listening on port 80.
            publicMode: false
        env:
          - key: 'BASE_URL'
            value: 'http://localhost:8080/data'
      config:
        test: 54
  routes:
    # Use this section to configure route between microservices
    # Use microservice name
    - from: 'heart-rate-monitor'
      to: 'heart-rate-viewer'
```

| Field         | Description                                                                                                                                                                                                                 |
| ------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| name          | User-defined unique identifier of an Application within an ioFog Controller. Must start and end with lowercase alphanumeric character. Can include '-' character.                                                           |
| microservices | List of Microservices. See Microservice section for more details.                                                                                                                                                           |
| routes        | List of ioFog Routes. `From` and `To` use microservice name as identifiers. The microservices specified must be part of the application. When deploying an application, prefer this method to the microservice route field. |

## Microservices

Microservices configuration and set up are defined using YAML files.

Those YAML definitions can be used inside an application YAML file, or by themselves when deploying a microservice to an existing application: `iofogctl deploy microservice -f <path-to-microservice.yaml>`

A microservice YAML definition file can be retrieved using the describe command: `iofogctl describe microservice <NAME> [-o microservice.yaml]`

```yaml
apiVersion: iofog.org/v2
kind: Microservice # What are we deploying
metadata:
  name: heart-rate-monitor # Microservice name
  namespace: default # (Optional) iofogctl namespace to use

# Specifications of the microservice
spec:
  # Agent on which to deploy the microservice
  agent:
    # Agent name
    name: zebra-1
    # Optional agent configuration
    config:
      # All fields are optional
      dockerUrl: unix:///var/run/docker.sock
      diskLimit: 50
      diskDirectory: /var/lib/iofog-agent/
      memoryLimit: 4096
      cpuLimit: 80
      logLimit: 10
      logDirectory: /var/log/iofog-agent/
      logFileCount: 10
      statusFrequency: 10
      changeFrequency: 10
      deviceScanFrequency: 60
      bluetoothEnabled: true
      watchdogEnabled: false
      abstractedHardwareEnabled: false
      upstreamRouters: ['default-router']
      networkRouter: ''
      host: horse-1
      routerConfig:
        routerMode: edge
        messagingPort: 5672
        edgeRouterPort: 56721
        interRouterPort: 56722
      dockerPruningFrequency: 1
      logLevel: INFO
      availableDiskThreshold: 90

  # Information about the container images to be used
  images:
    x86: edgeworx/healthcare-heart-rate:x86-v1 # Image to be used on x86 type agents
    arm: edgeworx/healthcare-heart-rate:arm-v1 # Image to be used on arm type agents
    registry: remote # Either 'remote' or 'local' or the registry ID - Remote will pull the image from Dockerhub, local will use the local cache of the agent
    # Optional catalog item id (See Catalog items in the advanced section)
    catalogId: 0 # 0 is equivalent to not providing the field

  # Microservice container configuration
  container:
    # Requires root host access on the agent ?
    rootHostAccess: false
    # Microservice container volume mapping list on the agent
    volumes:
      # This will create a volume mapping between the agent '/tmp/msvc' volume and the microservice container volume '/data'
      - hostDestination: '/tmp/msvc'
        containerDestination: '/data'
        accessMode: 'rw' # ReadWrite access to the mounted volume

    # Microservice container environment variable list on the agent
    env:
      # This will create an environment variable inside the microservice container with the key 'BASE_URL' and the value 'http://localhost:8080/data'
      - key: BASE_URL
        value: http://localhost:8080/data
    # Microservice container port mapping list on the agent
    ports:
      # This will create a mapping between the port 80 of the microservice container and the port 5000 of the agent
      - internal: 80
        external: 5000
    commands:
      # This will result in the container being started as `docker run <image> <options> dbhost localhost:27017`
      - 'dbhost'
      - 'localhost:27017'

  # Microservice configuration
  config:
    # Arbitrary key, value YAML object
    data_label: test_mode=false_cross_agent_microservice_routing_aug_27
    test_mode: true

  # List of microservice names to which a route needs to be created
  routes:
    - heart-rate-viewer # This will create a route from 'heart-rate-monitor' to 'heart-rate-viewer'
  # When deploying application, the application level 'routes' field is preferred to this field

  # Mandatory application name inside which to deploy the microservice
  application: Healthcare Wearable

  # Optional. Boolean instructing agent to rebuild the microservice container
  rebuild: false
```

| Field                    | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| name                     | User-defined unique identifier of an Microservice within an ioFog Controller. Must start and end with lowercase alphanumeric character. Can include '-' character.                                                                                                                                                                                                                                                                                                      |
| agent                    | Object describing the name and the required configuration of the ioFog agent the microservice is to be deployed on. All configuration fields are optional only the specified values will be updated.                                                                                                                                                                                                                                                                    |
| images                   | Description of the images to be used by the container running the microservice. `x86` is the image to be used on x86 ioFog Agents. `arm` is the image to be used on ARM ioFog Agents. `Registry` is either `local`, `remote`, or `registryID`. Remote will pull the image from Dockerhub, local will use the local cache of the ioFog Agent. RegistryID will use the specified registry. A catalog ID can be provided in lieu and place of the images and the registry. |
| config                   | User-defined arbitrary object to be passed to the microservice runtime as its configuration                                                                                                                                                                                                                                                                                                                                                                             |
| container.rootHostAccess | Does the container running the microservice requires root access to the host                                                                                                                                                                                                                                                                                                                                                                                            |
| container.ports          | List of port mapping to be provided to the container running the microservice                                                                                                                                                                                                                                                                                                                                                                                           |
| container.volumes        | List of volume mapping to be provided to the container running the microservice                                                                                                                                                                                                                                                                                                                                                                                         |
| container.env            | List of environment variables to be provided to the container running the microservice                                                                                                                                                                                                                                                                                                                                                                                  |
| container.commands       | List of arguments passed as CMD to the container runtime                                                                                                                                                                                                                                                                                                                                                                                                                |
| routes                   | List of ioFog Routes destination. Use microservice name as identifiers. The microservice specified must be part of the application. Only use this field when updating a microservice in isolation.                                                                                                                                                                                                                                                                      |
| application              | Unique identifier of the Application the microservice is part of                                                                                                                                                                                                                                                                                                                                                                                                        |
| rebuild                  | Boolean instructing the ioFog Agent to rebuild the microservice container after update. Use this flag if you updated the content of the docker image but didn't change the image name and/or tag.                                                                                                                                                                                                                                                                       |
|                          |

<aside class="notifications note">
  <b>See anything wrong with the document? Help us improve it!</b>
  <a href="https://github.com/eclipse-iofog/iofog.org/edit/develop/content/docs/2.0.0/iofogctl/application-yaml-spec.md"
    target="_blank">
    <p style="text-align:left">Edit on Github <img src="/images/icos/ico-github.svg" alt=""></p>
  </a>
</aside>