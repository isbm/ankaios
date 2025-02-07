# Quickstart

If you have not installed Ankaios or build from source already, please follow the instructions [here](installation.md). 

Ankaios needs a startup configuration that contains all the workloads and their
configuration which should be started when Ankaios starts up.

Let's create a simple config and store that in `state.yaml`

```yaml
workloads:
  nginx:
    runtime: podman
    agent: agent_A
    restart: true
    updateStrategy: AT_MOST_ONCE
    accessRights: # (1)
      allow: []
      deny: []
    tags:
      - key: owner
        value: Ankaios team
    runtimeConfig: |
      image: docker.io/nginx:latest
      ports:
      - containerPort: 80
        hostPort: 8081
```

1.  Note that access rights are currently not implemented.


Before we start Ankaios we need to make sure that Podman is listening on a
socket that can be used by Ankaios.

```shell
systemctl --user start podman.socket
```

Then we can start the Ankaios server:

```shell
ank-server --startup-config state.yaml
```

The Ankaios server will read the config but detect that no agent with the name
`agent_A` is available that could start the workload.

In a new terminal let's start an agent:

```shell
ank-agent --name agent_A
```

This Ankaios agent will run the workload that has been assigned to it. We can
use the Ankaios CLI to check the current state (again in an other terminal):

```shell
ank get state
```

Ankaios also provides adding and removing workloads dynamically.
To add another workload call:

```shell
ank run workload \
--runtime podman \
--agent agent_A \
--config 'image: docker.io/busybox:1.36
env:
  MESSAGE: Hello World!
command:
- sh
- -c
- echo "$MESSAGE"
' helloworld
```

We can check the state again with `ank get state` and see, that the workload
`helloworld` has been added to `currentState.workloads` and the execution
state is available in `workloadStates`.

As the workload had a one time job its state is `ExecSucceeded` and we can 
delete it from the state again with:

```shell
ank delete workload helloworld
```
