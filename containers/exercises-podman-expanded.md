# Chapter 2: Container Management with Podman

Podman is a daemonless, rootless container engine designed to run OCI (Open Container Initiative) containers and images. Unlike Docker, Podman does not rely on a background daemon process running with root privileges, making it a highly secure alternative for local container management. This chapter covers the essential concepts and commands for managing containers and images using Podman.

## Learning objectives

After this chapter you can:
- List and interpret the lifecycle states of containers managed by Podman.
- Pull images and run containers interactively or in the background.
- Configure port mapping and query active forwarding paths.
- Mount host directories as volumes, applying SELinux relabeling flags.
- Securely build custom images with a Containerfile using non-privileged users.

## Before you start

To complete this chapter, you must have Podman installed on your host machine.

On macOS or Windows, you will need to initialize and start a Podman virtual machine to execute container processes:

```bash
podman machine init
podman machine start
```

If you are running on an SELinux-enabled Linux host, keep in mind that Podman requires specific volume mount suffixes (`:Z` or `:z`) to allow containers access to files owned by the host system. If you are running on macOS or Windows, these suffixes are not supported and will cause errors.

All command blocks in this chapter assume a local bash or zsh shell on a machine with Podman active.

## Verifying the Podman Environment

Before running containers, you should inspect your container engine's environment and configuration.

To do this, use `podman info` to view the host platform's settings, followed by `podman version` to check software details.

```bash
podman info
podman version
```

### Explaining the diagnostic commands

- **`podman info`** (**podman** **info**rmation)
  - **What it does:** Displays system-wide information about the Podman installation, host environment, storage configuration, and runtime backend.
  - **Memory hook:** Think of this as the main dashboard showing the system configuration under which your container processes run.

- **`podman version`** (**podman** **version**)
  - **What it does:** Displays the exact client and engine versions.
  - **Memory hook:** Use this to verify that the CLI binary can query the local runtime environment.

### Running a test container

With Podman active, run a minimal hello container to verify the end-to-end download and execution flow:

```bash
podman run quay.io/podman/hello
```

- **`podman run`** (**podman** **run**)
  - **What it does:** Downloads an image (if not present locally), creates an isolated container runtime environment, and starts it.
  - **Memory hook:** This is the primary command used to start a new container process.

This command fetches the minimal testing image from Quay (a public container registry), instantiates it, and prints an informational greeting before exiting the process.

## Managing Containers and Their Lifecycle

Containers are isolated process groups managed directly by the local system. Since Podman is daemonless, there is no background parent process; instead, container processes run directly under your user session.

### Running a Background Web Server

To run long-lived applications like web servers without locking up your terminal window, start them in the background (detached mode):

```bash
podman run -d --name my-nginx nginx
```

The `-d` flag detaches your terminal session, while `--name` assigns the alias `my-nginx` to simplify subsequent management.

### Checking Process Status

To see currently running containers, use `podman ps`:

```bash
podman ps
```

- **`podman ps`** (**podman** **p**rocess **s**tatus)
  - **What it does:** Lists active, running containers on the local host.
  - **Memory hook:** This acts exactly like the Unix `ps` utility but is restricted to containerized processes.

The output shows columns for the container ID, active image, running command, creation timestamp, current status, exposed network ports, and the designated alias.

### Retrieving Logs and Configuration Details

To check what is happening inside your background containers, inspect their console output streams with `podman logs`:

```bash
podman logs my-nginx
```

- **`podman logs`** (**podman** **logs**)
  - **What it does:** Retrieves and displays stdout and stderr streams recorded from the containerized application.
  - **Memory hook:** This is your primary diagnostic trace for inspecting errors or queries written by background services.

For low-level network, storage, and process configurations, use `podman inspect`:

```bash
podman inspect my-nginx
```

- **`podman inspect`** (**podman** **inspect**)
  - **What it does:** Prints a comprehensive JSON array containing all environment variables, storage paths, and configuration schemas of the container.
  - **Memory hook:** Think of this as an exhaustive technical specification sheet for a container's configuration.

### Stopping and Cleaning Up Containers

When a service is no longer needed, you can gracefully stop and then permanently delete its container.

```bash
podman stop my-nginx
```

- **`podman stop`** (**podman** **stop**)
  - **What it does:** Requests a graceful shutdown of the container's primary process by sending `SIGTERM`, followed by a `SIGKILL` if it doesn't shut down within a grace period.
  - **Memory hook:** This is like sending a shutdown signal to an operating system rather than cutting power.

To view all containers, including those that have exited or stopped:

```bash
podman ps -a
```

To permanently remove the container and its associated read-write filesystem from disk:

```bash
podman rm my-nginx
```

- **`podman rm`** (**podman** **r**e**m**ove)
  - **What it does:** Deletes the stopped container's metadata and read-write layer.
  - **Memory hook:** This removes the specific container process instance while leaving the original image unchanged.

### Batch Cleanup of Extracted Containers

When multiple ephemeral containers have exited, you can clean them up programmatically by passing their quiet IDs to a loop:

```bash
for container_id in $(podman ps -a -q -f "ancestor=quay.io/podman/hello"); do
    podman rm "$container_id"
done
```

The `-q` flag prints only the short container IDs, and `-f` filters by the parent image.

> [!WARNING]
> **Common pitfalls**
>
> - **Removing running containers** — Attempting to run `podman rm` on an active container will fail. You must stop it first, or use the force flag: `podman rm -f my-nginx`.

## Managing Local Images

Images serve as the static, read-only layer blueprints from which containers are launched.

### Listing Stored Images

To list all container templates downloaded or built on your machine:

```bash
podman images
```

- **`podman images`** (**podman** **images**)
  - **What it does:** Lists local images with their repository namespace, version tag, ID, and physical size.
  - **Memory hook:** This displays the catalog of static file templates available to launch containers on your host.

### Downloading New Images

To fetch an image directly from a public container registry:

```bash
podman pull ubuntu:latest
```

- **`podman pull`** (**podman** **pull**)
  - **What it does:** Downloads filesystem layers from a registry and caches them locally.
  - **Memory hook:** This pulls down the files needed to run an application before starting it.

### Running Interactive Environments

You can start a temporary interactive container to execute commands directly inside its filesystem:

```bash
podman run -it ubuntu:latest bash
```

The combined `-it` flags allocate a pseudo-TTY and keep standard input open, allowing you to use bash inside the container. Type `exit` to close the shell and stop the container process.

### Deleting Local Images

To free up storage space, remove images that are no longer in use:

```bash
podman rmi ubuntu:latest
```

- **`podman rmi`** (**podman** **r**e**m**ove **i**mage)
  - **What it does:** Deletes a locally cached image from disk if no container relies on it.
  - **Memory hook:** The ending `i` specifically targets "images", preventing conflicts with container deletion commands.

## Interacting with Running Containers

You can launch new processes inside an already-active background container for administrative or diagnostic purposes.

### Running Commands in Background Containers

Start Nginx and publish port 8084:

```bash
podman run -d --name my-nginx -p 8084:80 nginx
```

Instead of opening a new shell, execute a quick command inside the container to inspect its system configuration:

```bash
podman exec my-nginx getent passwd | awk -F: '{ print $1}'
```

- **`podman exec`** (**podman** **exec**ute)
  - **What it does:** Runs a new process inside an active, running container without restarting its main application.
  - **Memory hook:** Use this to run single-use administration commands inside a container.

### Opening an Interactive Shell in an Active Container

To inspect files directly inside the active web server, open an interactive bash shell:

```bash
podman exec -it my-nginx bash
```

You can test local connections by running `curl http://localhost:8084` from your host terminal to verify that the background container process is responding.

To clean up:

```bash
podman rm -f my-nginx
```

## Exposing Services with Port Mapping

Containerized applications listen on private, isolated network interfaces inside their namespaces. To make these services accessible to the host machine or external clients, you must configure port mapping.

### Mapping Container Ports to Host Ports

Start Nginx and map host port 8080 to container port 80:

```bash
podman run -d --name my-web-server -p 8080:80 nginx
```

The `-p 8080:80` flag tells the engine to capture traffic bound for port 8080 on the host and redirect it to port 80 inside the container.

### Verifying Port Forwarding

Test the connection using curl:

```bash
curl http://localhost:8080
```

To see which host ports map to specific container interfaces, use `podman port`:

```bash
podman port my-web-server 80
```

- **`podman port`** (**podman** **port**)
  - **What it does:** Displays the network mapping between host interfaces and container ports.
  - **Memory hook:** Use this command to trace exactly which network port on the host routes to a specific container service port.

To clean up:

```bash
podman stop my-web-server
podman rm my-web-server
```

## Persisting Data with Volume Mounts

When a container process is deleted, its writable filesystem layer is destroyed along with it. To preserve files permanently or inject configuration from your host machine, you must configure bind mounts.

### Volume Mount Options and Platform Differences

> [!NOTE]
> **If you are running Podman on macOS or Windows, skip this section and omit the `:Z` suffix from all subsequent commands.** These host platforms do not enforce SELinux. Attempting to use `:Z` or `:z` flags on macOS/Windows host mounts will cause container startup to fail with an error (`lsetxattr: operation not supported`) because the shared host filesystem driver cannot apply Linux SELinux labels.

On Linux host systems where SELinux is active (such as Red Hat, Fedora, or CentOS), the kernel blocks containers from reading or writing to files on the host unless you authorize access. You do this by appending an SELinux label suffix to your volume mounts:

- **`:Z` (Private Mount)**: Tells Podman to relabel the host path to be private and unshared. Only this specific container can access the directory.
- **`:z` (Shared Mount)**: Tells Podman to apply a shared label, allowing multiple distinct containers to access the host directory concurrently.

*Note: Never use these suffixes on system directories.*

Throughout the rest of this chapter, the volume examples include the `:Z` suffix for SELinux compatibility. If you are on macOS or Windows, remember to strip the `:Z` flag from your commands (for example, use `-v ~/nginx-html:/usr/share/nginx/html` instead of `-v ~/nginx-html:/usr/share/nginx/html:Z`).

### Mounting Host Directories for Content Delivery

Create a directory on your host and write an HTML file:

```bash
mkdir ~/nginx-html
echo "<h1>Hello from Podman Volume</h1>" > ~/nginx-html/index.html
```

Launch Nginx and map this directory over the default Nginx index path, appending `:Z` for proper SELinux labeling:

```bash
podman run -d --name volume-nginx -p 8081:80 -v ~/nginx-html:/usr/share/nginx/html:Z nginx
```

Verify that Nginx serves your host file:

```bash
curl http://localhost:8081
```

If you modify the host file, the changes are reflected immediately:

```bash
echo "<h1>Updated Content</h1>" > ~/nginx-html/index.html
curl http://localhost:8081
```

### Providing Configuration Files

You can mount individual host configuration files into the container. Create a custom server config:

```bash
mkdir ~/nginx-conf
echo "server { listen 80; location / { return 200 'Hello from Custom Config'; } }" > ~/nginx-conf/custom.conf
```

Launch Nginx, mounting your custom file over the default site configuration path:

```bash
podman run -d --name custom-conf-nginx -p 8082:80 -v ~/nginx-conf/custom.conf:/etc/nginx/conf.d/default.conf:Z nginx
```

Test the server response:

```bash
curl http://localhost:8082
```

If you modify the configuration, you must restart the container process so the application reads the updated host file:

```bash
echo "server { listen 80; location / { return 200 'Config updated\n'; } }" > ~/nginx-conf/custom.conf
podman restart custom-conf-nginx
curl http://localhost:8082
```

To clean up:

```bash
for container_id in $(podman ps -a -q -f "name=nginx"); do
    podman rm -f "$container_id"
done
rm -rf ~/nginx-html ~/nginx-conf
```

### Extracting Application Logs

Volumes can also capture log files on the host disk, making them persistent and accessible even after the container process has terminated.

Create a logs folder on the host:

```bash
mkdir -p ~/app-logs
```

Start an Alpine Linux container that continuously appends timestamps to a log file, passing the `:Z` suffix:

```bash
podman run -d --name log-generator -v ~/app-logs:/app/logs:Z alpine \
    sh -c 'while true; do echo "[$(date +"%F %H:%M:%S")] Log entry from container" >> /app/logs/app.log; sleep 1; done'
```

Monitor the log output on the host:

```bash
tail -f ~/app-logs/app.log
```

Stop and remove the container, then verify that your logs remain preserved on disk:

```bash
podman stop log-generator
podman rm log-generator
cat ~/app-logs/app.log
rm -rf ~/app-logs
```

## Building Custom Images

To run your own custom software, write a recipe file named a `Containerfile` and compile it into an executable image.

### Writing a Basic Containerfile

Create a directory for your project:

```bash
mkdir my-app && cd my-app
echo "<h1>My Custom Web App</h1>" > index.html
```

Create a file named `Containerfile` and add the following content:

```containerfile
# Use Nginx as the base image
FROM nginx:latest

# Copy your custom index.html into the web root
COPY index.html /usr/share/nginx/html/index.html

# Expose port 80
EXPOSE 80

# Command to run when starting the container
CMD ["nginx", "-g", "daemon off;"]
```

Compile the image using `podman build`:

```bash
podman build -t my-custom-nginx .
```

- **`podman build`** (**podman** **build**)
  - **What it does:** Builds a new OCI image using instructions defined in a local `Containerfile` or `Dockerfile`.
  - **Memory hook:** Think of this as compiling your build context and configuration recipe into a static template.

Verify and run your custom image locally:

```bash
podman images
podman run -d --name custom-web -p 8083:80 my-custom-nginx
curl http://localhost:8083
```

To see the distinct filesystem layers used to construct your image:

```bash
podman history my-custom-nginx
```

- **`podman history`** (**podman** **history**)
  - **What it does:** Lists each layered filesystem step, creator instruction, and physical size added to the image during compilation.
  - **Memory hook:** This acts like a version control history log showing every filesystem modification step.

### Distributing Images to Container Registries

Container registries let you publish and distribute built images across different hosts and deployment environments.

Before pushing an image, you must authenticate to the container registry:

```bash
echo "$REG_PASS" | podman login -u <user-name> --password-stdin <registry-name>
```

> [!TIP]
> **Security Tip:** On macOS, you can replace `echo "$REG_PASS"` with `pbpaste` to pipe the password directly from your clipboard. This prevents your credentials from being exposed in plain text in your terminal or saved to your shell history file.

Tag your image with your registry address and namespace path:

```bash
podman tag my-custom-nginx:latest <registry-name>/<registry-project-name>/<username>/my-custom-nginx:latest
```

- **`podman tag`** (**podman** **tag**)
  - **What it does:** Creates an alias pointing to a source image ID, allowing you to namespace and version your templates.
  - **Memory hook:** Think of this as creating an alternative name or reference path pointing to your existing local image.

Now, push the tagged image to your container registry:

```bash
podman push <registry-name>/<registry-project-name>/<username>/my-custom-nginx:latest
```

- **`podman push`** (**podman** **push**)
  - **What it does:** Uploads the image and its layered filesystem blocks to a remote OCI container registry.
  - **Memory hook:** This publishes your template, making it available for remote servers to pull.

Test the published image by deleting your local image and pulling it back down:

```bash
podman rmi <registry-name>/<registry-project-name>/<username>/my-custom-nginx:latest
podman pull <registry-name>/<registry-project-name>/<username>/my-custom-nginx:latest
podman run -d --name registry-nginx -p 8083:80 <registry-name>/<registry-project-name>/<username>/my-custom-nginx:latest
```

To clean up:

```bash
podman rm -f custom-web registry-nginx
podman rmi my-custom-nginx:latest <registry-name>/<registry-project-name>/<username>/my-custom-nginx:latest
cd ..
rm -rf my-app
```

### Enhancing Security: Running as a Non-Privileged User

Running containers as the root user poses a significant security risk; if an application is compromised, the attacker can gain full administrative rights inside the container. Best practices require configuring a dedicated, non-privileged user within your container configuration.

Create a secure build directory:

```bash
mkdir my-app-secure && cd my-app-secure
echo "<h1>Secure Web App</h1>" > index.html
```

Create a `Containerfile` that sets up a dedicated, non-root user:

```containerfile
FROM alpine:latest

# Create a non-root group and user
RUN addgroup -S appgroup && adduser -S -D -H -u 1500 -G appgroup appuser

# Copy application files
COPY index.html /var/www/html/index.html

# Install Python and assign web directory ownership to your user
RUN apk add --no-cache python3 && chown -R appuser:appgroup /var/www/html

# Switch runtime contexts to your non-privileged user
USER appuser
WORKDIR /var/www/html

# Expose an unprivileged port
EXPOSE 8080

# Serve files using Python
CMD ["python3", "-m", "http.server", "8080"]
```

Compile and run your secure container image:

```bash
podman build -t my-secure-app .
podman run -d --name secure-web -p 8084:8080 my-secure-app
```

Verify that the process runs under your secure, non-privileged user ID:

```bash
podman exec secure-web id
```

You should see an output indicating the active user ID (`uid=1500(appuser)`).

To clean up:

```bash
podman rm -f secure-web
podman rmi my-secure-app
cd ..
rm -rf my-app-secure
```

## Multi-Container Orchestration with Compose

You can define and orchestrate multi-container application environments using YAML configuration files managed by Podman Compose.

### Defining a Compose File

Create a directory for your application project:

```bash
mkdir my-compose-app && cd my-compose-app
```

Create a configuration file named `compose.yaml` to define your web service, using the `:Z` suffix for SELinux compliance:

```yaml
services:
  web:
    image: nginx:latest
    ports:
      - "8083:80"
    volumes:
      - ./html:/usr/share/nginx/html:Z
```

Create the mounted host directories and files:

```bash
mkdir html
echo "<h1>Hello from Podman Compose</h1>" > html/index.html
```

### Operating Compose Applications

To orchestrate and launch your service stack, run `podman compose up`:

```bash
podman compose up -d
```

- **`podman compose`** (**podman** **compose**)
  - **What it does:** Orchestrates multi-container applications by mapping services to Podman's local container commands.
  - **Memory hook:** Think of this as a coordinator directing multiple dependent containers as a single, functional project.

The `-d` flag runs the services in the background. Verify connection reachability:

```bash
curl http://localhost:8083
```

To stop and clean up all resources, containers, and networks generated by your configuration:

```bash
podman compose down
```

The `down` command stops the active container processes and removes the containers and network setup.

To clean up:

```bash
cd ..
rm -rf my-compose-app
```
