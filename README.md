# Welcome to the Podman Container Basics session modules

This module has two sections: Part 1, Linux Administration, and Part 2, Podman Container Management.

- [Linux Administration exercises](linux-foundation/exercises.md)
- [Podman Container Management exercises](containers/exercises.md)

## Part 1: Linux Administration

These exercises provide practical experience with the command line and the core skills needed to manage a Linux
system.

### Module 1: Linux Fundamentals

Navigate the filesystem, create and manage files and directories, and process text from the command line.

### Module 2: System Administration

Install software, inspect processes, and manage the services that keep a Linux system running.

### Module 3: Security

Create users and groups, manage ownership, and control access with Linux permissions.

### Module 4: Storage

Create partitions, format filesystems, mount storage, and configure persistent mounts with `/etc/fstab`.

### Module 5: Networking

Inspect network configuration, test connectivity and DNS, and manage basic firewall rules.

### Module 6: Automation with Scripting

Write Bash and Python scripts for user onboarding and disk monitoring, then make shell functions available as commands.

## Part 2: Podman Container Management

These exercises build practical experience with Podman and the commands used to run, inspect, publish, configure, build,
and distribute containers.

On macOS or Windows, initialize a Podman virtual machine once with `podman machine init`, then start it with
`podman machine start`. The exercises use rootless Podman where possible. On an SELinux-enabled Linux host, use the
`:Z` or `:z` suffixes on bind mounts as shown in the volume exercise.

### Exercise 1: Getting Started

Verify Podman with `podman info` and `podman version`, then run the Podman hello container.

### Exercise 2: Managing Running Containers

Run Nginx in the background, list running containers, inspect logs, and examine container configuration.

### Exercise 3: Stopping and Removing Containers

Stop and remove containers, inspect stopped containers, and clean up the completed hello-container runs.

### Exercise 4: Managing Podman Images

List local images, pull Ubuntu, start an interactive container, and remove an image.

### Exercise 5: Interacting with Running Containers

Use `podman exec` to run commands inside a background container and access its published HTTP service.

### Exercise 6: Port Mapping

Publish an Nginx service from the container to the host and verify the mapping with `podman port`.

### Exercise 7: Bind Mounts, Configuration, and Logs

Use host directories and files as external input and output. The examples include SELinux-safe `:Z` bind mounts, live
content updates, custom Nginx configuration, and log collection.

### Exercise 8: Building Custom Images with a Containerfile

Build a custom Nginx image, inspect its layers, push and pull an image through Azure Container Registry, and run an
application as a non-root user.

### Exercise 9: Podman Compose

Use `podman compose` with a Compose provider to define, start, inspect, and remove a simple multi-container application.

The detailed exercises are designed for learning by doing. Review the expected output, answer the reflection questions,
and clean up the containers and files created during each exercise.
