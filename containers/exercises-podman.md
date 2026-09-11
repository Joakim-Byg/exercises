# Podman Container Management Exercises
These exercises give you hands-on experience with Podman, a daemonless container engine for running OCI containers and
images. By the end of these exercises, you will have practiced the essential commands for managing containers and
images.

> These exercises use rootless Podman where possible. On macOS or Windows, initialize a Podman virtual machine once with
> `podman machine init`, then start it with `podman machine start`. On an SELinux-enabled Linux host, use the `:Z` or
> `:z` volume suffixes shown in Exercise 7 so Podman can access bind-mounted files.

## Exercise 1: Getting Started - Running Your First Container
**Objective:** Verify Podman and run a very basic container.

### 1. Verify Podman:

* Open your terminal.

* Type `podman info` and press Enter.

* **Expected Output:** You should see information about your Podman installation, storage, networking, and runtime. If
  you get an error, Podman might not be installed or its virtual machine might not be running.

* Type `podman version` and press Enter.

* **Expected Output:** You should see the installed Podman version.

### 2. Run the Podman hello container:

* This is a minimal image designed to test your Podman installation.

* `podman run quay.io/podman/hello`
  
* **Observe:** What message is displayed? What happens to the container after it runs?

## Exercise 2: Managing Running Containers
**Objective:** Learn to run containers in the background, list them, and view their logs.

### 1. Run a simple Nginx web server in detached mode:

* We'll run an Nginx web server, which will keep running in the background.

* `podman run -d --name my-nginx nginx`

* **Explanation:**

  * `-d`: Runs the container in "detached" mode (in the background).

  * `--name my-nginx`: Assigns a human-readable name to your container.

  * `nginx`: The name of the Nginx image to use.

* **Observe:** What is the output of this command? (It should be a long string, the container ID).

### 2. List running containers:

* `podman ps`

* **Observe**: Do you see your my-nginx container listed? Pay attention to its `CONTAINER ID`, `IMAGE`, `COMMAND`, 
  `CREATED`, `STATUS`, `PORTS`, and `NAMES`.

### 3. View container logs:

* `podman logs my-nginx`
  
* **Observe**: What information do the logs provide? This is useful for debugging.

### 4. Inspect container details:

* `podman inspect my-nginx`

* **Observe**: Scroll through the extensive JSON output. What kind of detailed information can you find about the 
  container's configuration, network settings, and volumes?

## Exercise 3: Stopping and Removing Containers
**Objective**: Learn how to gracefully stop and remove containers.

### 1. Stop the my-nginx container:

* `podman stop my-nginx`

* **Observe**: What is the output?

* Verify it's stopped: `podman ps` (it should no longer appear in the list of running containers).

### 2. List all containers (including stopped ones):

* `podman ps -a`

* **Observe**: Now `my-nginx` should appear, but its `STATUS` should indicate it's exited.

### 3. Remove the my-nginx container:

* `podman rm my-nginx`

* **Observe**: What is the output?

* Verify it's removed: `podman ps -a` (it should no longer appear at all).

### 4. Clean up previous Podman hello containers:

* You might have several `quay.io/podman/hello` containers from Exercise 1. They run and exit immediately.

* List them: `podman ps -a -f "ancestor=quay.io/podman/hello"`

* Remove them all at once:
  ```shell
  for container_id in $(podman ps -a -q -f "ancestor=quay.io/podman/hello"); do
      podman rm "$container_id"
  done
  ```

* **Explanation**:

  * `podman ps -a -q`: Lists all container IDs, quietly (only the IDs).

  * `-f "ancestor=quay.io/podman/hello"`: Filters by containers created from the Podman hello image.

  * The `for` loop removes each matching container and does nothing when there are no matches.

## Exercise 4: Managing Podman Images
**Objective**: Understand how to pull images and manage them locally.

### 1. List local images:

* `podman images`

* **Observe**: What images do you currently have? (You should see `quay.io/podman/hello` and `nginx`).

### 2. Pull a new image (e.g., Ubuntu):

* `podman pull ubuntu:latest`

* **Observe:** Watch the download process.

* Verify it's downloaded: `podman images`

### 3. Run a container from the new image and interact with it:

* `podman run -it ubuntu:latest bash`

* **Explanation:**

  * `-it`: Combines `-i` (interactive) and `-t` (pseudo-TTY), allowing you to interact with the container's shell.

  * `bash`: The command to run inside the Ubuntu container (starts a bash shell).

* Inside the container: Try some basic Linux commands like `ls /`, `pwd`, `apt update`.

* Exit the container: Type exit and press Enter.

* **Observe**: What happens to the container after you exit? (It stops).

### 4. Remove an image:

* First, ensure no containers are using the image you want to remove (check `podman ps -a`).

* `podman rmi ubuntu:latest`

* **Observe:** What is the output?

* Verify it's removed: `podman images`

## Exercise 5: Interacting with running containers
**Objective:** Learn how to run commands inside containers running in the background.
### 1. Rerun the simple Nginx
* `podman run -d --name my-nginx -p 8084:80 nginx`
* Make sure it is running as expected with `podman ps -f "name=my-nginx"`
### 2. Run commands inside the container
* Make a command that extracts user information directly from the running container:
   * `podman exec my-nginx getent passwd | awk -F: '{ print $1}'`
   *  **Explanation:**
     * `exec` runs a command inside a running container without replacing its main process.
     * `getent passwd | awk -F: '{ print $1}'` is the command we want to be executed inside the container, in this case
       called `my-nginx`. This command extracts user-information inside the container OS and prints the usernames part, 
       through the `awk` command.  

* Now attach to the container with the `bash` command:
   * `podman exec -it my-nginx bash`
   * **Explanation:**
     * The `-it` flags create an interactive terminal for the running container.
     * `bash` (the terminal) is the command we execute on the container.

* From the host terminal, run `curl http://localhost:8084`.
* **Observe:** The container is still running in the background, but its HTTP service is reachable through the
  published host port. What are the security implications of publishing a port?

### 3. Clean up
* `podman rm -f my-nginx`

## Exercise 6: Port Mapping - Making Container Services Accessible
**Objective**: Expose a container's service to your host machine.

### 1. Run an Nginx container and map its port:

*  `podman run -d --name my-web-server -p 8080:80 nginx`

*  **Explanation:**
   * `-p 8080:80`: Maps port `8080` on your host machine to port `80` inside the container.

* **Observe**: What is the output?

### 2. Access the Nginx server from your browser:

  * Open your web browser and navigate to http://localhost:8080 (or http://your-linux-vm-ip:8080).

  * **Expected**: You should see the default Nginx welcome page.

### 3. Verify port mapping with podman:

  *  `podman ps`
  * `podman port my-web-server 80`
  * **Observe**: The output should show that host port `8080` forwards to container port `80`.

### 4. Clean up:
*  `podman stop my-web-server`
*  `podman rm my-web-server`

## Exercise 7: Volumes - Configuring and Observing Containers
**Objective**: Use bind mounts as an external interface for an application inside a container.

Bind mounts let you provide input such as configuration files and retrieve output such as logs. This keeps the
application data outside the container, so the container can be replaced without losing that data.

### SELinux labels for bind mounts
On an SELinux-enabled Linux host, Podman needs permission to access files mounted from the host. Add `:Z` to a mount
that belongs to one container. Use `:z` when the same host content is shared by multiple containers.

The examples below use `:Z` because each host directory or file belongs to one container. Podman relabels the mounted
content automatically. Do not use these options on system directories.

### 1. Create a directory on your host for Nginx content:
* `mkdir ~/nginx-html`

* `echo "<h1>Hello from Podman Volume</h1>" > ~/nginx-html/index.html`

### 2. Run an Nginx container with a volume mount:

* `podman run -d --name volume-nginx -p 8081:80 -v ~/nginx-html:/usr/share/nginx/html:Z nginx`

* **Explanation:** 
  * `-v ~/nginx-html:/usr/share/nginx/html:Z` mounts your host directory ~/nginx-html to the container's
    Nginx web root directory at `/usr/share/nginx/html`.

### 3. Access the custom Nginx page: 
* Either use `curl http://localhost:8081` to see the served content or open your web browser and navigate to the public 
  IP of your environment http://your-linux-vm-ip:8081.

* **Expected:** You should see "Hello from Podman Volume".

### 4. Modify the host file and observe changes:

* `echo "<h1>Updated Content</h1>" > ~/nginx-html/index.html`

* Refresh your browser page.

* **Expected**: The content should update immediately, demonstrating persistence.

### 5. Configuration as artifact
* Create a custom Nginx configuration file on your host:
  * `mkdir ~/nginx-conf`
  * `echo "server { listen 80; location / { return 200 'Hello from Custom Config'; } }" > ~/nginx-conf/custom.conf`
    
* Run an Nginx container, mounting your custom config:
  * `podman run -d --name custom-conf-nginx -p 8082:80 -v ~/nginx-conf/custom.conf:/etc/nginx/conf.d/default.conf:Z nginx`

  * **Explanation**: The `-v ~/nginx-conf/custom.conf:/etc/nginx/conf.d/default.conf:Z` flag mounts your host's
    `custom.conf` file directly over the default Nginx configuration file inside the container.

* Access the Nginx server from your terminal with `curl http://localhost:8082`
  * **Expected**: You should see "Hello from Custom Config". This confirms the container is using your external configuration.
### 6. Modify the configuration and observe:
* Overwrite the existing Nginx configuration file on your host:
  * `echo "server { listen 80; location / { return 200 'Config updated\n'; } }" > ~/nginx-conf/custom.conf`
* Access the Nginx server from your terminal with `curl http://localhost:8082`
  * **Observe:** Is the response as expected?

* Restart the container with `podman restart custom-conf-nginx`
  * **Observe:** What is the output from `curl http://localhost:8082`?

### 7. Nginx clean up:

* Stop and remove all containers whose names contain `nginx`:
  ```shell
  for container_id in $(podman ps -a -q -f "name=nginx"); do
      podman rm -f "$container_id"
  done
  ```

* `rm -rf ~/nginx-html ~/nginx-conf` (removes the exercise directories and their contents)

### 8. Extracting Logs and Output from a Container

**Goal**: Run a simple application that generates logs, and then mount a volume to capture these logs on your host 
machine. This demonstrates how to retrieve output from a container for analysis or persistence.

* Create a directory on your host for logs:

  * `mkdir -p ~/app-logs`

* Run a simple `alpine` container that writes to a log file, mounting the log directory:

  * ```shell
    podman run -d --name log-generator -v ~/app-logs:/app/logs:Z alpine \
        sh -c 'while true; do time echo "[$(date +"%F %H:%M:%S")] Log entry from container" >> /app/logs/app.log; sleep 1; done'
    ```

  * **Explanation**: This command runs a simple shell script inside an `alpine` container. The script continuously
    writes timestamped messages to `app.log` within the `/app/logs` directory, which is mounted from your host's 
    `~/app-logs`.

* **Observe** the logs on your host:

  * `tail -f ~/app-logs/app.log`

  * **Expected**: You should see log entries appearing in real-time, demonstrating that the container's output is being
    written directly to your host file system.

### 9. Log clean up:

* `podman stop log-generator`

* `podman rm log-generator`

* `rm -rf ~/app-logs` (removes the log directory and its contents from your host)

## Exercise 8: Building Custom Images with a Containerfile
**Objective**: Understand the basics of creating your own Podman images and how you can enhance security by running
applications inside containers with non-privileged users.
### 1. Basic image building:
* **Goal:** Understand the basics of creating your own container images.

* **Steps:**
  1. Create a new directory for your Containerfile (and change to it):

     * `mkdir my-app`

     * `cd my-app`

  2. Create a simple `index.html` file:

     * `echo "<h1>My Custom Web App</h1>" > index.html`

  3. Create a `Containerfile`:

     * Using your preferred text editor (e.g., `nano Containerfile` or `vi Containerfile`), add the following content:
       ```Containerfile
       # Use an official Nginx image as the base
       FROM nginx:latest
  
       # Copy your custom index.html into the Nginx web root
       COPY index.html /usr/share/nginx/html/index.html
  
       # Expose port 80 (optional, but good practice for documentation)
       EXPOSE 80
  
       # Command to run when the container starts (Nginx's default command)
       CMD ["nginx", "-g", "daemon off;"]
       ```
  4. Build your image:

     * Make sure you are in the `my-app` directory, where `Containerfile` and `index.html` are.
     
       * `podman build -t my-custom-nginx .`
       
       * **Explanation:**
         * `-t my-custom-nginx`: Tags your new image with the name `my-custom-nginx`.
         
         * `.`: Specifies the "build context" (the current directory), where Podman will look for the `Containerfile` and
           other files.
         
       * **Observe**: Watch the build process. Each instruction in the Containerfile corresponds to a build step.

  5. Verify your new image:

     * `podman images`

     * **Expected:** You should see `my-custom-nginx` listed.

  6. Run a container from your custom image:

     * `podman run -d --name custom-web -p 8083:80 my-custom-nginx`

  7. Access your custom web app:

     * Open your web browser and navigate to http://your-linux-vm-ip:8083.

     * **Expected**: You should see "My Custom Web App".
  8. To inspect the image layers and when they were added, use `podman history my-custom-nginx`:
     * **Expected:** The timing of the layers should fit your latest build time. 
### 2. Using container registries and tags
**Objective:** Understand that container images can be distributed through registries. These exercises cover pulling and
pushing images and selecting specific image versions.

**Steps:** 

1. Log in to the Container Registry. The registry name passed to the CLI is the short name, not the login server:
    1. `az login --identity`
    2. Get an access token and pass it to Podman:
       ```shell
       ACR_TOKEN="$(az acr login --name <registry-name> --expose-token --output tsv --query accessToken)"
       printf '%s' "$ACR_TOKEN" | podman login your.favorite.registry.io \
         --username <registry-name> --password-stdin
       unset ACR_TOKEN
       ```
2. Before you can push the container image to the registry, the image must be **tagged** accordingly:
    * `podman tag my-custom-nginx:latest <registry-name>/<registry-project-name>/<username>/my-custom-nginx:latest`
    * `podman images`
    * **Expect:** Shows you multiple images with same `SIZE` but with different `REPOSITORY` 
      (`<registry-name>.azurecr.io/<username>/my-custom-nginx` and `my-custom-nginx`) and `IMAGE ID`, but with `TAG` is 
      "latest" for both. 
    * **Explanation**:
       * `REPOSITORY` denotes where the image can be found i.e. the container registry `<registry-name>.azurecr.io`, at 
         path `<username>/my-custom-nginx`.
       * `TAG` as "latest" is given to an image per default if no tag is defined. We will return to defining tags 
         explicitly.
3. `podman push <registry-name>/<registry-project-name>/<username>/my-custom-nginx:latest`
    * **Observe:**  Watch the upload process.
4. To verify that the registry has the container image:
    * `podman rmi <registry-name>/<registry-project-name>/<username>/my-custom-nginx:latest`
    * `podman images`
       * **Observe:** The image is not listed
    * `podman pull <registry-name>/<registry-project-name>/<username>/my-custom-nginx:latest`
    * `podman images`
       * **Observe:** The image has returned
    * `podman rmi <registry-name>/<registry-project-name>/<username>/my-custom-nginx:latest`
    * `podman run -d --name registry-nginx <registry-name>/<registry-project-name>/<username>/my-custom-nginx:latest`
      * **Observe:** The image is fetched similarly to when we ran `podman run -d --name my-nginx nginx`.
5. Stop and remove `registry-nginx`, then compare the local tag with the registry tag:
   * `podman rm -f registry-nginx`
   * `podman rmi my-custom-nginx:latest`
   * `podman tag <registry-name>.azurecr.io/<username>/my-custom-nginx:latest my-custom-nginx:latest`
   * **Observe:** `podman images` should show the same image ID for both tags.
6. **Clean up:** Use the commands you have learned to review and remove what you created.
    * **Hint:**
      * `podman ps -a -f "name=<>"`
      * `podman stop`
      * `podman rm`
      * `podman images`
      * `podman rmi`
    * Remove the container from the basic image-building exercise before continuing:
      * `podman rm -f custom-web`
### 3. Running Applications as Non-Privileged Users (Security Best Practice)
* **Goal:** Create a Containerfile that runs the application inside the container as a dedicated, non-root user. This
  limits the impact of a compromised application.

* **Steps:**
  1. Navigate back to your `my-app` directory or create a new one:

     * `mkdir -p my-app-secure && cd my-app-secure`

     * `echo "<h1>Secure Web App</h1>" > index.html`
  2. Create a `Containerfile` for a non-root user:

     * Using your preferred text editor, add the following content:
       ```Containerfile
       FROM alpine:latest
       
       RUN addgroup -S appgroup && adduser -S -D -H -u 1500 -G appgroup appuser
       COPY index.html /var/www/html/index.html
       RUN apk add --no-cache python3 && chown -R appuser:appgroup /var/www/html
       USER appuser
       WORKDIR /var/www/html
       EXPOSE 8080
       CMD ["python3", "-m", "http.server", "8080"]
       ```
  3. Build and run the image:
     * `podman build -t my-secure-app .`
     * `podman run -d --name secure-web -p 8084:8080 my-secure-app`
  4. Verify the application and user:
     * Open `http://localhost:8084` in a browser, or run `curl http://localhost:8084`.
     * Run `podman exec secure-web id`.
     * **Expected:** The web page shows "Secure Web App" and the process runs as `appuser`, not `root`.
  5. Clean up:
     * `podman rm -f secure-web`

## Exercise 9: Introduction to Podman Compose (Optional/Simple)
**Objective**: Understand how to manage multi-container applications with Podman Compose.

* Prerequisite: Install Podman and a Compose provider such as `podman-compose`. The `podman compose` command delegates
  to an installed Compose provider.

### 1. Create a new directory for your Compose project:

* `mkdir my-compose-app`

* `cd my-compose-app`

### 2. Create a `compose.yaml` file:

* This file defines your services (containers) and how they relate.

* Using your preferred text editor, add the following content:

  ```yaml
  services:
    web:
      image: nginx:latest
      ports:
        - "8083:80"
      volumes:
        - ./html:/usr/share/nginx/html:Z # Relabel the bind mount for SELinux
  ```
### 3. Create the `html` directory and an `index.html` file:

* `mkdir html`

* `echo "<h1>Hello from Podman Compose</h1>" > html/index.html`

### 4. Start your services with Podman Compose:

* Make sure you are in the `my-compose-app` directory.

* `podman compose up -d`

* **Explanation**:

  * `up`: Creates and starts the services defined in `compose.yaml`.

  * `-d`: Runs the services in detached mode (background).

* **Observe**: Podman Compose will pull images if they are not present and start the containers.

### 5. Verify running services:

* `podman ps`

* **Expected**: You should see a container for the `web` service.

### 6. Access your web server:

* Open your web browser and navigate to `http://localhost:8083` or `http://your-linux-vm-ip:8083`.

* **Expected**: You should see "Hello from Podman Compose!".

### 7. Stop and remove services:

* `podman compose down`

* **Explanation**: Stops and removes the containers and network created by this Compose project. Volumes are kept
  unless you add the provider's volume-removal option.

* **Observe**: Verify with `podman ps -a` that the containers are gone.

### 8. Clean up:

* `cd ..`

* `rm -rf my-compose-app`

## Conclusion
These exercises cover the fundamental commands and concepts for managing containers with Podman. Practice the commands
regularly to build confidence, and use the Podman documentation when you need more detail.
