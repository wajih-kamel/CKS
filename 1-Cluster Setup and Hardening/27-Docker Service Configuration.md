> ## Documentation Index
> Fetch the complete documentation index at: https://notes.kodekloud.com/llms.txt
> Use this file to discover all available pages before exploring further.

# Docker Service Configuration

> This article explains how to configure the Docker daemon service on Linux using systemd, including starting, stopping, and managing the service.

In this lesson, we explore how to configure the Docker daemon service on Linux using systemd. We build on basic service management commands such as start, status, and stop. Note that procedures may vary depending on your operating system and Docker installation method.

## Checking the Docker Service Status

To verify that the Docker service is running, use the following command:

```bash  theme={null}
systemctl status docker
```

This command produces output similar to:

```plaintext  theme={null}
● docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2020-10-21 04:21:01 UTC; 3 days ago
     Docs: https://docs.docker.com
 Main PID: 4197 (dockerd)
    Tasks: 13
   Memory: 129.7M
      CPU: 9min 6.980s
   CGroup: /system.slice/docker.service
           └─4197 /usr/bin/dockerd -H fd:// -H tcp://0.0.0.0 --containerd=/run/containerd/containerd.sock
```

## Starting and Stopping the Docker Service

You can control the Docker service using the following commands:

<Callout icon="lightbulb" color="#1CB2FE">
  To start Docker, run:

  ```bash  theme={null}
  systemctl start docker
  ```

  To stop Docker, run:

  ```bash  theme={null}
  systemctl stop docker
  ```
</Callout>

Running Docker as a systemd service allows the daemon to run in the background and automatically start at boot. Alternatively, you can run the Docker daemon in the foreground using the `dockerd` command, which is particularly useful for troubleshooting.

## Running the Docker Daemon in the Foreground

When you run the daemon in the foreground, log messages are printed directly to the console. For example, you can start the daemon in debug mode with:

```bash  theme={null}
dockerd --debug
```

The output will include debug messages similar to the following:

```plaintext  theme={null}
INFO[2020-10-24T08:20:40.372653436Z] Starting up
INFO[2020-10-24T08:20:40.375298351Z] parsed scheme: "unix"
INFO[2020-10-24T08:20:40.375510773Z] scheme "unix" not registered, fallback to default scheme  module=grpc
INFO[2020-10-24T08:20:40.375657667Z] ccResolverWrapper: sending update to cc: [{unix:///run/containerd/containerd.sock 0 <nil>}] <nil> module=grpc
INFO[2020-10-24T08:20:40.375973480Z] ClientConn switching balancer to "pick_first"  module=grpc
INFO[2020-10-24T08:20:40.381198263Z] [graphdriver] using prior storage driver: overlay2
WARN[2020-10-24T08:20:40.572888603Z] Your kernel does not support swap memory limit
WARN[2020-10-24T08:20:40.573141192Z] Your kernel does not support cgroup rt period
WARN[2020-10-24T08:20:40.573408479Z] Your kernel does not support cgroup rt runtime
```

For even more detailed logs, the `--debug` flag will output additional information:

```bash  theme={null}
dockerd --debug
```

This produces verbose logs, including:

```plaintext  theme={null}
INFO[2020-10-24T08:29:00.331925176Z] Starting up
DEBU[2020-10-24T08:29:00.332463203Z] Listener created for HTTP on unix (/var/run/docker.sock)
INFO[2020-10-24T08:29:00.333116936Z] Golang's threads limit set to 6930
INFO[2020-10-24T08:29:00.333695956Z] parsed scheme: "unix"
INFO[2020-10-24T08:29:00.333705237Z] scheme "unix" not registered, fallback to default scheme  module=grpc
INFO[2020-10-24T08:29:00.333712042Z] ccResolverWrapper: sending update to cc: [{unix:///run/containerd/containerd.sock 0 <nil>}] <nil> module=grpc
INFO[2020-10-24T08:29:00.334889983Z] parsed scheme: "unix"
INFO[2020-10-24T08:29:00.334896126Z] scheme "unix" not registered, fallback to default scheme  module=grpc
INFO[2020-10-24T08:29:00.334913273Z] ccResolverWrapper: sending update to cc: [{unix:///run/containerd/containerd.sock 0 <nil>}] <nil> module=grpc
INFO[2020-10-24T08:29:00.335168292Z] Using default logging driver json-file
[graphdriver] priority list: [btrfs zfs overlay2 aufs overlay devicemapper vfs]
INFO[2020-10-24T08:29:00.335695827Z] processing event stream
DEBU[2020-10-24T08:29:00.337633123Z] backingFs=extfs, projectQuotaSupported=false, indexOff="" storage-driver=overlay2
[graphdriver] using prior storage driver: overlay2
WARN[2020-10-24T08:29:00.364679147Z] Initialized graph driver overlay2
WARN[2020-10-24T08:29:00.364679177Z] Your kernel does not support swap memory limit
WARN[2020-10-24T08:29:00.364679192Z] Your kernel does not support cgroup rt period
WARN[2020-10-24T08:29:00.364679207Z] Your kernel does not support cgroup rt runtime
```

## Docker CLI Communication via Unix Socket

When the Docker daemon starts, it listens on an internal Unix socket at `/var/run/docker.sock`. This Inter-Process Communication (IPC) mechanism allows local processes, especially the Docker CLI, to interact with the daemon.

## Remote Access to the Docker Daemon

If you need to manage Docker containers on a remote host—for instance, from your laptop targeting a server—you must configure the Docker daemon to listen on a TCP interface. By default, Docker listens only on the Unix socket, so it is necessary to explicitly specify a TCP host.

For example, if your Docker host has an IP address of `192.168.1.10` and you want to listen on port `2375`, start the daemon as follows:

```bash  theme={null}
dockerd --debug \
  --host=tcp://192.168.1.10:2375
```

Then, on your laptop, set the environment variable to interact with the remote Docker host:

```bash  theme={null}
export DOCKER_HOST="tcp://192.168.1.10:2375"
docker ps
```

<Callout icon="triangle-alert" color="#FF6B6B">
  Exposing the Docker API over TCP without encryption or authentication represents a significant security risk. Unauthorized users who can access this interface may run containers on your host for malicious purposes.
</Callout>

## Enabling TLS for Secure Communication

To secure the TCP interface, enable TLS encryption. First, generate TLS certificates and then start the Docker daemon with the appropriate TLS flags. TLS-secured Docker daemons typically listen on port `2376`.

Set the `DOCKER_HOST` environment variable accordingly:

```bash  theme={null}
export DOCKER_HOST="tcp://192.168.1.10:2376"
docker ps
```

Start the Docker daemon with TLS enabled:

```bash  theme={null}
dockerd --debug \
  --host=tcp://192.168.1.10:2376 \
  --tls=true \
  --tlscert=/var/docker/server.pem \
  --tlskey=/var/docker/serverkey.pem
```

## Using a Configuration File

Instead of setting all the options on the command line, you can use a configuration file located at `/etc/docker/daemon.json`. This JSON file is not created by default, so you must create it manually. Here is an example configuration:

```json  theme={null}
{
  "debug": true,
  "hosts": ["tcp://192.168.1.10:2376"],
  "tls": true,
  "tlscert": "/var/docker/server.pem",
  "tlskey": "/var/docker/serverkey.pem"
}
```

The `hosts` property is an array that can include multiple listeners. Once this configuration file is in place, you no longer need to pass these options on the command line when starting Docker.

<Callout icon="triangle-alert" color="#FF6B6B">
  If a parameter is specified both in the `daemon.json` file and via command-line flags, Docker will report a conflict and fail to start. For example, conflicting `debug` settings will cause an error.
</Callout>

After updating the configuration file, restart the Docker service with:

```bash  theme={null}
systemctl start docker
```

This configuration file is also used when starting Docker as a systemd service.

***

That concludes our lesson on Docker service configuration. For further details on Docker functionalities, please refer to the [Docker Documentation](https://docs.docker.com).

<CardGroup>
  <Card title="Watch Video" icon="video" cta="Learn more" href="https://learn.kodekloud.com/user/courses/certified-kubernetes-security-specialist-cks/module/eac6dac8-4481-4138-96ef-a2135f20e05e/lesson/ba404fff-7bc0-42ff-9430-ba10009a2e47" />
</CardGroup>


Built with [Mintlify](https://mintlify.com).