# Welcome to the Container Basics session modules

This module has two sections: Part 1, Linux Administration, and Part 2, Container Management.

- [Linux Administration exercises](linux-foundation/exercises.md)
- [Docker Container Management exercises](containers/exercises-docker.md)
- [Podman Container Management exercises](containers/exercises-podman.md)

## Part 1. Linux Administration
To begin with, we should (re)visit some Linux Administration Hands-On Exercises! The goal of this session is to move 
beyond theory and give you practical, hands-on experience with the core skills to navigate and manage Linux in a 
day-to-day setting. We will work directly on the command line to: have an entry to, get a reference to, and a functional
understanding of how a Linux system works.

### A Quick Tour of Our Linux Exercises
Here is a brief overview of the modules we will cover today. Each section is designed to build upon the last, taking you
from basic navigation to systemic automation.

#### Module 1: Linux Fundamentals
This first section is all about the essentials. We will focus on navigating the filesystem, manipulating files and 
directories, and using build-in tools to process text directly from the command line.

#### Module 2: System Administration
Here, we delve into managing the software and services that are the basics of any Linux server. You will practice
installing applications and controlling the background daemons that keep the system running.

#### Module 3: Security
In this module, we will cover the critical tasks of managing who can access the system and what they are allowed to do. 
We will create users and groups and control file access using permissions.

#### Module 4: Storage
Every system needs storage. This section will guide you through the process of adding a new disk to a server, from
creating partitions to formatting and mounting a filesystem so it becomes available for use.

#### Module 5: Networking
Here, we will explore how a Linux machine communicates with the outside world. You will learn to inspect network
configurations, test connectivity, and manage basic firewall rules to secure the system.

#### Module 6: Automation with Scripting
Finally, we will tie everything together by automating common administrative tasks. This module will introduce you to
basic Bash and Python scripting, and show you how to integrate your scripts into the shell to create powerful, custom 
commands.


## Part 2: Docker Container Management
Welcome to your hands-on journey into Docker and container management! These exercise sessions are designed to give you 
practical experience with the fundamental commands and concepts. Our main objective is to get familiarity with a set of
essential operations commonly used when developing, operating and maintaining containers and hopefully preparing you to
leverage this technology in your daily work.

You'll be working through a series of exercises, each focusing on a key aspect of container management. 
Get ready to dive in and learn by doing!

### Your Exercise Journey: A Quick Overview
Here's a brief introduction to what you'll be covering in each section:

#### Exercise 1: Getting Started - Running Your First Container
This initial exercise will help you confirm your container runtime (i.e. your Docker installation) is working correctly 
and introduce you to the very first command you'll use: `docker run`. It's your basic "hello world" moment in the 
containerized universe.

#### Exercise 2: Managing Running Containers
Once you can run a container, you'll need to manage it. This section focuses on launching containers in the background,
listing them to see their status, and inspecting their logs to understand what's happening inside.

#### Exercise 3: Stopping and Removing Containers
Containers have a lifecycle. Here, you'll learn how to gracefully stop containers that are no longer needed and how to
permanently remove them, ensuring a clean Docker environment.

#### Exercise 4: Managing Docker Images
Images are the blueprints for your containers. This exercise will teach you how to pull pre-built images from Docker Hub
and manage the images stored on your local machine.

#### Exercise 5: Port Mapping - Making Container Services Accessible
Containers often run services that need to be accessed from outside. This section will show you how to "map" ports,
allowing external traffic to reach services running inside your containers.

#### Exercise 6: Volumes - Dynamic Configuration and Observation
Volumes provide a way to manage external data for your containers. In this exercise, you'll explore how volumes  can be
used to inject configuration files and dynamic content (input) into a running container. You will also learn to extract
logs or other generated data (output), allowing you to observe and control containerized services dynamically, while
also understanding how applications within containers react to these external changes.

#### Exercise 7: Building Custom Images with Dockerfile (Introduction)
Ready to create your own containerized applications? This exercise provides an introduction to Dockerfiles, the simple
text files that contain instructions for building your very own custom Docker images.

#### Exercise 8: Introduction to Docker Compose (Optional/Simple)
For applications with multiple services (like a web server and a database), Docker Compose simplifies management. This 
optional exercise provides a basic overview of how to define and run multi-container applications with a single command.

You are encouraged to experiment, ask questions, and explore beyond the exercises. The best way to learn is by doing! 
Good luck, and enjoy your containerized journey!

## Part 3: Podman Container Management

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
