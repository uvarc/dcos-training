# DCOS Training

Covers basic functionality for UVARC deployments

## Access / Logging In

1. Via a web browser https://128.143.245.170/. Authenticate using your UVA username/password.
2. Via the CLI. You must first log into the web UI and then select the "UVA-DCOS" drop-down in the upper-right corner, then select "Install CLI". Once installed, authenticate using LDAP and UVA credentials.

## Basic Container Deployment

DCOS was created to simplify container operations and fault-tolerance. In this exercise, you will run a basic (empty) container that echoes a bash stdout back to you.

1. Select the SERVICES navigation menu item.
2. If necessary, click into the service namespace assigned to you.
3. Select the `+ NEW` button in the upper-right, then "Run a Service."
4. Start by running a single container service.
5. Assign a unique service ID to your service. The service ID is prepended with your namespace, something like `/namespace/service-name`. This is how you will identify a specific service in later commands.
6. For container image, enter `busybox`, a general-purpose container with basic tools installed.
7. For the command, run the `date` command. However, that will execute so quickly that it will make it difficult to see the output. So set the command to `date; sleep 60;`. That will allow a minute to review the container output before the service refreshes.
8. Click `Review & Run`, then `Run Service` to confirm.
9. You will see your service appear in the list of running services. Click into the name of it.
10. Under the "Tasks" sub-menu for your service you will see all running containers and the resources allocated to it.
11. Finally, click the small page icon (to the right of "Health") to review container logs. Change to the "Output (stdout)" view. You should see something like this:

```
(AT BEGINNING OF FILE)
Mon Jan  6 14:41:49 UTC 2020
```

## Environment Variables and Secrets

1. In this exercise we will run the `busybox` container again but echo back variables entered at runtime. This would be useful if, for instance, there are usernames, passwords, keys, etc. that you need to run your service but which should not be "baked" into a container image. There are two main types of variables:
  a. Environment Variables
  b. Secrets
2. Create a new service with a new ID.
3. Find the "Environment" pane on the left of the service configuration wizard. Click `+ Add Environment Variable`, which is a simple key-value store that is made available to your running container(s). 
4. Enter a "key" of `LASTNAME` and a "value" of your own last name. This means that your last name is now available to the container as `$LASTNAME`.
5. Return to the "Service" pane of the wizard. For the command, enter `echo $LASTNAME; sleep 60;`.
6. Run your service and review the stdout log. You should see your name in the output.
7. Secrets are similar to environment variables in that they are made available specifically to your container(s) at runtime. However, they are encrypted, can be managed separately from your services, and can be consumed by your service as either an environment variable or as a file.

Two important details to remember about secrets:

- Secrets have their own namespace path (like `/namespace/secret_a`, etc.) that must match the namespace path of the service you wish to consume it with.
- Secrets should be used for runtime variables that you do not want exposed to other users, such as passwords, keys, or tokens, etc. Normal environment variables are useful for setting flags, command names, file paths, etc. that are not sensitive. But secrets are the more secure way to store and consume sensitive variables.

## Advanced Settings

Other advanced settings are useful for networked containers that need public visibility, or shared storage mounts, etc.

### Placement

As the name "distributed cloud operating system" suggests, DCOS is designed for multi-datacenter, multi-region, distributed computing. Placement rules
allow you to specify constraints like that the `hostname` the service should run on (by IP address), or the region, or whether it needs GPUs, or the max number of containers on any particular host, etc.

### Networking

Many containers need network connectivity in order to run. If the container only needs outbound connectivity, then no network settings are needed. If the container needs inbound connectivity, a few options are available:

**Network Types**

- Host - the IP and ports available on the host machine of the container itself.
- Bridge - a virtual network that spans across the cluster. This is similar to a Docker overlay network.
- Virtual Networks - a feature not yet implemented. This allows for the creation of separate, isolated networks for projects (CIDR blocks).

**Service Endpoints**
These endpoints allow you to map the network type and port to your container. The most common scenario is a Bridge network that is used to expose a container. 

For example, to run an NGINX web container, create a Bridge network. The container port 80 should be exposed to a host port automatically (it is mapped automatically to a port >10000 which can then be used for reverse proxying). 

More complicated scenarios are possible, where you declare a service endpoint name to be consumed by an N-tier application. A web layer could communicate with the service endpoint of an application layer, etc. 

### Volumes

Containers often need storage to run. DCOS allows for several options, but two are most useful:

- **Local Persisten Volume** - a local DCOS storageshare created on the host machine of the container. This is only persistent if new instances of the container run on the same host, which is made possible by using placement features.
- **Host Volume** - a local mountpoint of the host machine of the container. This can be a local directory or an NFS share. This is useful for attaching to Qumulo or GPFS shares.

### Health Checks

DCOS allows you to declare the "healthy" state of your container. This could be a successfull `HTTP 200` message, a ping reply, or the active availability of a TCP port.

## Command-Line

Services can be defined using standard JSON templates, and then run from the command-line. Here is a basic NGINX web service:

```
{
  "labels": {
    "HAPROXY_GROUP": "external",
    "HAPROXY_0_VHOST": "neal.uvadcos.io"
  },
  "id": "/neal/nginx",
  "backoffFactor": 1.15,
  "backoffSeconds": 1,
  "container": {
    "type": "MESOS",
    "volumes": [],
    "portMappings": [
      {
        "containerPort": 80,
        "hostPort": 0,
        "protocol": "tcp",
        "servicePort": 10026
      }
    ],
    "docker": {
      "image": "nginx",
      "forcePullImage": false,
      "parameters": []
    }
  },
  "cpus": 0.1,
  "disk": 0,
  "instances": 1,
  "maxLaunchDelaySeconds": 300,
  "mem": 128,
  "gpus": 0,
  "networks": [
    {
      "mode": "container/bridge"
    }
  ],
  "requirePorts": false,
  "upgradeStrategy": {
    "maximumOverCapacity": 1,
    "minimumHealthCapacity": 1
  },
  "killSelection": "YOUNGEST_FIRST",
  "unreachableStrategy": {
    "inactiveAfterSeconds": 0,
    "expungeAfterSeconds": 0
  },
  "role": "neal",
  "tasksStats": {
    "startedAfterLastScaling": {
      "stats": {
        "counts": {
          "staged": 0,
          "running": 1,
          "healthy": 0,
          "unhealthy": 0
        },
        "lifeTime": {
          "averageSeconds": 4670032.063,
          "medianSeconds": 4670032.063
        }
      }
    },
    "withLatestConfig": {
      "stats": {
        "counts": {
          "staged": 0,
          "running": 1,
          "healthy": 0,
          "unhealthy": 0
        },
        "lifeTime": {
          "averageSeconds": 4670032.063,
          "medianSeconds": 4670032.063
        }
      }
    },
    "totalSummary": {
      "stats": {
        "counts": {
          "staged": 0,
          "running": 1,
          "healthy": 0,
          "unhealthy": 0
        },
        "lifeTime": {
          "averageSeconds": 4670032.063,
          "medianSeconds": 4670032.063
        }
      }
    }
  },
  "healthChecks": [],
  "fetch": [],
  "constraints": []
}
```
