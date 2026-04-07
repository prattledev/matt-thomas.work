---
date: 2026-04-07T09:00:00Z
draft: false
categories:
  - Broadcast Technology
tags:
  - Docker
  - Linux
  - Networking
  - Software
  - NMOS
  - IP
title: "Docker Basics for Broadcast Engineers"
description: "Docker packages software and all its dependencies into portable containers that run the same way on any machine. For broadcast engineers deploying IP infrastructure tools, it removes the configuration friction and gets complex systems running quickly."
cover:
  image: docker_logo.png
  alt: "Docker"
---

Broadcast IP infrastructure increasingly relies on software-defined components - NMOS registries, stream monitors, timing tools, test utilities. Getting these running on a server traditionally means installing dependencies, resolving version conflicts, and documenting the exact steps so someone can reproduce the environment six months later. Docker changes that. It packages an application and everything it needs into a single portable unit that runs the same way regardless of what is already installed on the host.

If you have worked through the [Git post](../git-for-audio-engineers) or the [SSH post](../ssh-basics-audio-engineers), Docker follows the same pattern: a tool from the software world that has clear practical value for engineers working with modern broadcast infrastructure.

## What Docker Is

Docker is a platform for running applications in **containers**. A container is an isolated process - it has its own filesystem, networking, and dependencies, but shares the operating system kernel of the host. This makes containers much lighter than virtual machines, which require a full OS per instance.

The key benefit for broadcast work is reproducibility. If someone has built and tested a containerised NMOS registry or MQTT broker, you can run an identical copy of it on your machine with a single command. No dependency installation, no version mismatches, no "works on my machine" problems.

Docker runs on Linux natively. On macOS and Windows it runs Linux containers inside a lightweight VM, which is handled transparently by Docker Desktop.

## Core Concepts

### Images

An **image** is a read-only template for a container - a snapshot of a filesystem including the application, its runtime, libraries, and configuration. Images are built from a `Dockerfile` (a recipe file) and are stored in registries.

**Docker Hub** is the default public registry. Images like `ubuntu`, `python`, or `nginx` are hosted there and can be pulled directly. Project-specific images, such as the NMOS tools used in the Easy NMOS example below, are also published there by their maintainers.

### Containers

A **container** is a running instance of an image. You can run multiple containers from the same image simultaneously. Each container is isolated by default, though you can expose ports and mount directories to connect it to the outside world.

Containers are ephemeral by design. Anything written inside a container's filesystem that is not mapped to a host directory disappears when the container is removed. For persistent data (configuration files, databases), you mount a directory from the host into the container.

### The Docker Daemon

The **Docker daemon** (`dockerd`) is the background service that manages images, containers, networks, and volumes. The `docker` CLI talks to the daemon. On Linux, the daemon runs as a system service. On macOS and Windows, Docker Desktop manages it.

### Docker Compose

**Docker Compose** is a tool for defining and running multi-container applications. Rather than running individual `docker run` commands for each service, you define the entire stack in a `docker-compose.yml` file and bring it up with a single command. For broadcast infrastructure with multiple interdependent services - registry, node, testing tool - this is the practical way to manage things.

## Basic Commands

**Pull an image from a registry**

```bash
docker pull ubuntu:22.04
```

Downloads the image to your local machine without running it. The tag (`22.04`) specifies the version. Without a tag, Docker defaults to `latest`.

**Run a container**

```bash
docker run -it ubuntu:22.04 bash
```

Creates and starts a container from the image. `-it` gives an interactive terminal session. When you exit the shell, the container stops.

```bash
docker run -d -p 8080:80 nginx
```

`-d` runs the container in the background (detached). `-p 8080:80` maps port 8080 on the host to port 80 inside the container.

**List running containers**

```bash
docker ps
```

Shows all running containers with their ID, image, status, and mapped ports.

```bash
docker ps -a
```

Shows all containers including stopped ones.

**View container logs**

```bash
docker logs <container_id>
docker logs -f <container_id>
```

Shows the stdout/stderr output of a container. `-f` follows the log in real time, useful for watching a service start up.

**Execute a command inside a running container**

```bash
docker exec -it <container_id> bash
```

Opens a shell inside a running container. Useful for inspecting configuration or running diagnostics without stopping the container.

**Stop and remove a container**

```bash
docker stop <container_id>
docker rm <container_id>
```

`stop` sends a graceful shutdown signal. `rm` removes the stopped container. To remove a running container immediately: `docker rm -f <container_id>`.

**List and remove images**

```bash
docker images
docker rmi <image_name>
```

**Pull updated images**

```bash
docker pull <image_name>
```

Images are not automatically updated. Running `docker pull` again fetches the latest version if the tag (e.g. `latest`) has been updated upstream.

### Docker Compose Commands

```bash
docker-compose up          # Start all services defined in docker-compose.yml
docker-compose up -d       # Start in detached (background) mode
docker-compose down        # Stop and remove all containers in the stack
docker-compose pull        # Pull updated images for all services
docker-compose logs -f     # Follow logs from all services
docker-compose ps          # Show status of all services in the stack
```

Run these commands from the directory containing your `docker-compose.yml`.

## Example: Easy NMOS

[Easy NMOS](https://github.com/rhastie/easy-nmos) by Rob Hastie is a Docker Compose-based starter kit for deploying a working NMOS environment with minimal setup. It spins up three containers:

- **NMOS Registry** - an nmos-cpp registry/controller instance
- **Virtual NMOS Node** - an nmos-cpp node that automatically registers with the registry
- **AMWA NMOS Testing Tool** - the standard NMOS test suite for validating behaviour

This is a useful reference deployment for anyone learning NMOS, testing devices against a known-good registry, or validating IS-04/IS-05/IS-08 behaviour without needing physical hardware.

### Prerequisites

A Linux host with Docker and Docker Compose installed, and three unused IP addresses on your network subnet. The default configuration uses `192.168.6.101`, `192.168.6.102`, and `192.168.6.103` - these need to be changed to addresses that are free on your network.

### Getting the Project

```bash
git clone https://github.com/rhastie/easy-nmos.git
cd easy-nmos
```

### Configuration

Open `docker-compose.yml` and update three things to match your environment:

1. **IP addresses** - replace `192.168.6.101`, `192.168.6.102`, `192.168.6.103` with unused addresses on your subnet
2. **Subnet** - update the network subnet to match your network (e.g. `192.168.1.0/24`)
3. **Network interface** - change `ens33` to your host's network interface name (find it with `ip link` or `ifconfig`)

The project uses a **macvlan** network, which assigns each container its own MAC address and IP on the physical network. This is important for NMOS because mDNS discovery requires containers to be directly visible on the network rather than hidden behind NAT. Each container gets an IP that other devices on the network can reach directly.

### Starting the Environment

Pull the latest images first, then bring the stack up:

```bash
docker-compose pull
docker-compose up -d
```

After a few seconds, the three services will be running. You can check their status with:

```bash
docker-compose ps
```

### Accessing the Services

| Service | Address | Port |
|---|---|---|
| NMOS Registry UI | `http://192.168.6.101` | 80 |
| Registry API | `http://192.168.6.101:81` | 81 |
| MQTT Broker | `192.168.6.101` | 1883 |
| Virtual Node | `http://192.168.6.102` | 80 |
| NMOS Testing Tool | `http://192.168.6.103:5000` | 5000 |

Replace `192.168.6.x` with whatever addresses you configured. The NMOS Testing Tool at port 5000 lets you run the AMWA test suites against the registry and node, or against any IS-04/IS-05 device on your network.

### Watching the Logs

```bash
docker-compose logs -f
```

This is useful when first bringing the stack up - you can watch the registry and virtual node initialise and confirm that the node has successfully registered. If something is not working, the logs will usually tell you why.

### Stopping the Environment

```bash
docker-compose down
```

This stops and removes all three containers. The next `docker-compose up` will recreate them from the images. Because the configuration is in the mounted files rather than written inside the containers, nothing is lost.

## Why This Matters for Broadcast

The Easy NMOS example illustrates why Docker is useful for broadcast infrastructure. A three-service NMOS stack that would previously have required careful manual installation on a Linux machine - resolving Python dependencies for the testing tool, building nmos-cpp from source, configuring mDNS - is instead defined in a single YAML file and running in minutes. The configuration is version-controlled, reproducible, and portable.

The same principle applies to other tools common in IP broadcast workflows: stream monitors, RTSP/SRT servers, SDP generators, PTP monitoring dashboards. If a tool is available as a Docker image, the operational overhead drops significantly compared to a bare-metal installation. Updates are a `docker pull` and a container restart. Rollbacks are a matter of specifying the previous image tag.

For engineers building and maintaining broadcast IP infrastructure, learning Docker is a practical investment. The concepts are not complex, and the payoff in deployment consistency is significant.
