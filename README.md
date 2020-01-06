# DCOS Training

Covers basic functionality for UVARC deployments

### Access / Logging In

1. Via a web browser https://128.143.245.170/. Authenticate using your UVA username/password.
2. Via the CLI. You must first log into the web UI and then select the "UVA-DCOS" drop-down in the upper-right corner, then select "Install CLI". Once installed, authenticate using LDAP and UVA credentials.

### Basic Container Deployment

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

### Environment Variables and Secrets

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

### Advanced Settings

