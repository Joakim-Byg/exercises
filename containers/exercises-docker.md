# Container Management Exercises
These exercises are designed to give you hands-on experience with Docker, the leading containerization platform. 
By the end of these exercises, you have had hands on experiences with the essential commands for managing containers 
and images.

## Exercise 1: Getting Started - Running Your First Container
**Objective:** Verify Docker installation and run a very basic container.

### 1. Verify Docker Installation:

* Open your terminal.

* Type `docker info` and press Enter.

* **Expected Output:** You should see detailed information about your Docker client and server. If you get an error, 
  Docker might not be running or installed correctly.

* Type `docker version` and press Enter.

* **Expected Output:** You should see the client and server versions of Docker.

### 2. Run the hello-world container:

* This is a minimal image designed to test your Docker installation.

* `docker run hello-world`
  
* **Observe:** What message is displayed? What happens to the container after it runs?

## Exercise 2: Managing Running Containers
**Objective:** Learn to run containers in the background, list them, and view their logs.

### 1. Run a simple Nginx web server in detached mode:

* We'll run an Nginx web server, which will keep running in the background.

* `docker run -d --name my-nginx nginx`

* **Explanation:**

  * `-d`: Runs the container in "detached" mode (in the background).

  * `--name my-nginx`: Assigns a human-readable name to your container.

  * `nginx`: The name of the Docker image to use.

* **Observe:** What is the output of this command? (It should be a long string, the container ID).

### 2. List running containers:

* `docker ps`

* **Observe**: Do you see your my-nginx container listed? Pay attention to its `CONTAINER ID`, `IMAGE`, `COMMAND`, 
  `CREATED`, `STATUS`, `PORTS`, and `NAMES`.

### 3. View container logs:

* `docker logs my-nginx`
  
* **Observe**: What information do the logs provide? This is useful for debugging.

### 4. Inspect container details:

* `docker inspect my-nginx`

* **Observe**: Scroll through the extensive JSON output. What kind of detailed information can you find about the 
  container's configuration, network settings, and volumes?

## Exercise 3: Stopping and Removing Containers
**Objective**: Learn how to gracefully stop and remove containers.

### 1. Stop the my-nginx container:

* `docker stop my-nginx`

* **Observe**: What is the output?

* Verify it's stopped: `docker ps` (it should no longer appear in the list of running containers).

### 2. List all containers (including stopped ones):

* `docker ps -a`

* **Observe**: Now `my-nginx` should appear, but its `STATUS` should indicate it's exited.

### 3. Remove the my-nginx container:

* `docker rm my-nginx`

* **Observe**: What is the output?

* Verify it's removed: `docker ps -a` (it should no longer appear at all).

### 4. Clean up previous `hello-world` containers:

* You might have several `hello-world` containers from Exercise 1. They run and exit immediately.

* List them: `docker ps -a -f "ancestor=hello-world"`

* Remove them all at once: `docker rm $(docker ps -a -q -f "ancestor=hello-world")`

* **Explanation**:

  * `docker ps -a -q`: Lists all container IDs, quietly (only the IDs).

  * `-f "ancestor=hello-world"`: Filters by containers created from the `hello-world` image.

  * `$(...)`: Command substitution, meaning the output of the inner command becomes arguments for the outer command.

## Exercise 4: Managing Docker Images
**Objective**: Understand how to pull images and manage them locally.

### 1. List local images:

* `docker images`

* **Observe**: What images do you currently have? (You should see `hello-world` and `nginx`).

### 2. Pull a new image (e.g., Ubuntu):

* `docker pull ubuntu:latest`

* **Observe:** Watch the download process.

* Verify it's downloaded: `docker images`

### 3. Run a container from the new image and interact with it:

* `docker run -it ubuntu:latest bash`

* **Explanation:**

  * `-it`: Combines `-i` (interactive) and `-t` (pseudo-TTY), allowing you to interact with the container's shell.

  * `bash`: The command to run inside the Ubuntu container (starts a bash shell).

* Inside the container: Try some basic Linux commands like `ls /`, `pwd`, `apt update`.

* Exit the container: Type exit and press Enter.

* **Observe**: What happens to the container after you exit? (It stops).

### 4. Remove an image:

* First, ensure no containers are using the image you want to remove (check `docker ps -a`).

* `docker rmi ubuntu:latest`

* **Observe:** What is the output?

* Verify it's removed: `docker images`

## Exercise 5: Interacting with running containers
**Objective:** Learn how to interact with containers running in the background. We already learned how to list them and
see their logs, now we will enable us to attach to them and further understand their current state. 
### 1. Rerun the simple Ngninx
* `docker run -d --name my-nginx nginx`
* Make sure it is running as expected with `docker ps -f "name=my-nginx"`
### 2. Attach to the container
* Make a command that extracts user information directly from the running container:
   * `docker exec my-nginx getent passwd | awk -F: '{ print $1}'`
   *  **Explanation:**
     * `exec` is the docker command that lets us attach to a background container
     * `getent passwd | awk -F: '{ print $1}'` is the command we want to be executed inside the container, in this case
       called `my-nginx`. This command extracts user-information inside the container OS and prints the usernames part, 
       through the `awk` command.  

* Now attach to the container with the `bash` command:
   * `docker exec -it my-nginx bash`
   * **Explanation:**
     * The `-it` flags we used previously can be combine `exec` to attach interactively with the background container.
     * `bash` (the terminal) is the command we execute on the container.

* Run `curl localhost`
* **Observe:** What are looking at? what are the implications?

## Exercise 6: Port Mapping - Making Container Services Accessible
**Objective**: Expose a container's service to your host machine.

### 1. Run an Nginx container and map its port:

*  `docker run -d --name my-web-server -p 8080:80 nginx`

*  **Explanation:**
   * `-p 8080:80`: Maps port `8080` on your host machine to port `80` inside the container.

* **Observe**: What is the output?

### 2. Access the Nginx server from your browser:

  * Open your web browser and navigate to http://localhost:8080 (or http://your-linux-vm-ip:8080).

  * **Expected**: You should see the default Nginx welcome page.

### 3. Verify port mapping with docker ps:

  *  `docker ps`
  * **Observe**: Look at the `PORTS` column for `my-web-server`. It should show `0.0.0.0:8080->80/tcp`.

### 4. Clean up:
*  `docker stop my-web-server`
*  `docker rm my-web-server`

## Exercise 7: Volumes - Configuring and Observing Containers
**Objective**: Understanding how volumes can be used as external interface for the application inside a container.

Volumes have many purposes including detached storage, but can also be a powerful mechanism for interacting with 
containers, acting as an external interface for both providing 
input (configuration) and retrieving output (logs, data). This approach keeps your containers stateless and easily 
replaceable, which is crucial for scalable and manageable setups.

### 1. Create a directory on your host for Nginx content:
* `mkdir ~/nginx-html`

* `echo "<h1>Hello from Docker Volume</h1>" > ~/nginx-html/index.html`

### 2. Run an Nginx container with a volume mount:

* `docker run -d --name volume-nginx -p 8081:80 -v ~/nginx-html:/usr/share/nginx/html nginx`

* **Explanation:** 
  * `-v ~/nginx-html:/usr/share/nginx/html` mounts your host directory ~/nginx-html to the container's
    Nginx web root directory at `/usr/share/nginx/html`.

### 3. Access the custom Nginx page: 
* Either use `curl http://localhost:8081` to see the served content or open your web browser and navigate to the public 
  IP of your environment http://your-linux-vm-ip:8081.

* **Expected:** You should see "Hello from Docker Volume".

### 4. Modify the host file and observe changes:

* `echo "<h1>Updated Content</h1>" > ~/nginx-html/index.html`

* Refresh your browser page.

* **Expected**: The content should update immediately, demonstrating persistence.

### 5. Configuration as artifact
* Create a custom Nginx configuration file on your host:
  * `mkdir ~/nginx-conf`
  * `echo "server { listen 80; location / { return 200 'Hello from Custom Config'; } }" > ~/nginx-conf/custom.conf`
    
* Run an Nginx container, mounting your custom config:
  * `docker run -d --name custom-conf-nginx -p 8082:80 -v ~/nginx-conf/custom.conf:/etc/nginx/conf.d/default.conf nginx`

  * **Explanation**: The `-v ~/nginx-conf/custom.conf:/etc/nginx/conf.d/default.conf` flag mounts your host's 
    `custom.conf` file directly over the default Nginx configuration file inside the container.

* Access the Nginx server from your terminal with `curl http://localhost:8082`
  * **Expected**: You should see "Hello from Custom Config". This confirms the container is using your external configuration.
### 6. Modify the configuration and observe:
* Overwrite the existing Nginx configuration file on your host:
  * `echo "server { listen 80; location / { return 200 'Config updated\n'; } }" > ~/nginx-conf/custom.conf`
* Access the Nginx server from your terminal with `curl http://localhost:8082`
  * **Observe:** Is the response as expected?

* Restart the container with `docker restart custom-conf-nginx`
  * **Observe:** What is the output from `curl http://localhost:8082`?

### 7. Nginx clean up:

* `docker stop $(docker ps -a -q -f "name=nginx")`

* `docker rm $(docker ps -a -q -f "name=nginx")`

* `rm -rf ~/nginx-*` (removes the directories and its contents)

### 8. Extracting Logs and Output from a Container

**Goal**: Run a simple application that generates logs, and then mount a volume to capture these logs on your host 
machine. This demonstrates how to retrieve output from a container for analysis or persistence.

* Create a directory on your host for logs:

  * `mkdir -p ~/app-logs`

* Run a simple `alpine` container that writes to a log file, mounting the log directory:

  * ```shell
    docker run -d --name log-generator -v ~/app-logs:/app/logs alpine \
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

* `docker stop log-generator`

* `docker rm log-generator`

* `rm -rf ~/app-logs` (removes the log directory and its contents from your host)

## Exercise 8: Building Custom Images with Dockerfile
**Objective**: Understand the basics of creating your own Docker images and how you can enhance security by running
applications inside containers with non-privileged users.
### 1. Basic image building:
* **Goal:** Understand the basics of creating your own Docker images.

* **Steps:**
  1. Create a new directory for your Dockerfile (and change to it):

     * `mkdir my-app`

     * `cd my-app`

  2. Create a simple `index.html` file:

     * `echo "<h1>My Custom Web App</h1>" > index.html`

  3. Create a `Dockerfile`:

     * Using your preferred text editor (e.g., nano Dockerfile or vi Dockerfile), add the following content:
       ```Dockerfile
       # Use an official Nginx image as the base
       FROM nginx:latest
  
       # Copy your custom index.html into the Nginx web root
       COPY index.html /usr/share/nginx/html/index.html
  
       # Expose port 80 (optional, but good practice for documentation)
       EXPOSE 80
  
       # Command to run when the container starts (Nginx's default command)
       CMD ["nginx", "-g", "daemon off;"]
       ```
  4. Build your Docker image:

     * Make sure you are in the my-app directory (where Dockerfile and index.html are).
     
       * `docker build -t my-custom-nginx .`
       
       * **Explanation:**
         * `-t my-custom-nginx`: Tags your new image with the name `my-custom-nginx`.
         
         * `.`: Specifies the "build context" (the current directory), where Docker will look for the `Dockerfile` and 
           other files.
         
       * **Observe**: Watch the build process. Each line in the Dockerfile corresponds to a step.

  5. Verify your new image:

     * `docker images`

     * **Expected:** You should see `my-custom-nginx` listed.

  6. Run a container from your custom image:

     * `docker run -d --name custom-web -p 8083:80 my-custom-nginx`

  7. Access your custom web app:

     * Open your web browser and navigate to http://your-linux-vm-ip:8083.

     * **Expected**: You should see "My Custom Web App".
  8. To inspect images on this level and at what time the various layers have been added, use `docker history`:
     * **Expected:** The timing of the layers should fit your latest build time. 
### 2. Using container registries and tags
**Objective:** Understanding that container images are essentially distributions. We explore registries as a means to 
make our containers available for other users. These basic exercises covers pulling and pushing images and how we pick 
specific versions of the images.

**Steps:** 

1. Loging into container registries can typically be done with a `docker login -u <user> <registry-address>`, however in
   our setting we will utilize a registry residing in Microsoft Azure, so our commands will be as follows:
    1. `az login --identity`
    2. `az acr login --name <registry-name>.azurecr.io`
2. Before you can push the container image to the registry, the image must be **tagged** accordingly:
    * `docker build -t <registry-name>.azurecr.io/<username>/my-custom-nginx .`
    * `docker images`
    * **Expect:** Shows you multiple images with same `SIZE` but with different `REPOSITORY` 
      (`<registry-name>.azurecr.io/<username>/my-custom-nginx` and `my-custom-nginx`) and `IMAGE ID`, but with `TAG` is 
      "latest" for both. 
    * **Explanation**:
       * `REPOSITORY` denotes where the image can be found i.e. the container registry `<registry-name>.azurecr.io`, at 
         path `<username>/my-custom-nginx`.
       * `TAG` as "latest" is given to an image per default if no tag is defined. We will return to defining tags 
         explicitly.
3. `docker push <registry-name>.azurecr.io/<username>/my-custom-nginx`
    * **Observe:**  Watch the upload process.
4. To verify that the registry has the container image:
    * `docker rmi <registry-name>.azurecr.io/<username>/my-custom-nginx`
    * `docker images`
       * **Observe:** The image is not listed
    * `docker pull <registry-name>.azurecr.io/<username>/my-custom-nginx`
    * `docker images`
       * **Observe:** The image has returned
    * `docker rmi <registry-name>.azurecr.io/<username>/my-custom-nginx`
    * `docker run -d --name my-custom-nginx <registry-name>.azurecr.io/<username>/my-custom-nginx`
      *  **Observe:** The image is fetched similarly to when we ran `docker run -d --name my-nginx nginx`  
5. As we utilized the exact same Dockerfile for `my-custom-nginx` and 
   `<registry-name>.azurecr.io/<username>/my-custom-nginx`, we should align the `IMAGE ID` accordingly:
   * `docker rmi my-custom-nginx`
   * `docker tag <registry-name>.azurecr.io/<username>/my-custom-nginx my-custom-nginx`
   * **Observe:** Using `docker images` should reveal that `IMAGE ID`
6. **Clean up:** It is time to clean up; utilise your learned commands to get an overview of what you have created and remove the clutter.
    * **Hint:**
     * `docker ps -a -f "name=<>"`
     * `docker stop` 
     * `docker rm`
     * `docker images`
     * `docker rmi`
### 3. Running Applications as Non-Privileged Users (Security Best Practice)
* **Goal:** Create a Dockerfile that runs the application inside the container as a dedicated, non-root user. This
  significantly enhances security by limiting the potential impact of a compromise.

* **Steps:**
  1. Navigate back to your my-app directory (or create a new one):

     * `mkdir -p my-app-secure && cd my-app-secure`

     * `echo "<h1>Secure Web App</h1>" > index.html`
  2. Create a `Dockerfile` for a non-root user:

     * Using your preferred text editor, add the following content:
       ```Dockerfile
       # Use a smaller, more secure base image if possible (e.g., alpine)
       FROM alpine:latest
       
       # Create a non-root user and group
       RUN addgroup -S appgroup && adduser -u 1500 -S appuser -G appgroup
       
       # Copy your custom index.html into the Nginx web root
       COPY index.html /var/www/html/index.html
       
       # Install Python and change ownership
       RUN apk add --no-cache python3 && chown -R appuser:appgroup /var/www/html
       USER appuser
       WORKDIR /var/www/html
       EXPOSE 80
       CMD ["python3", "-m", "http.server", "80"]
       ```
     * Let's stick with the Nginx example for consistency, but be aware of the port 80 limitation for non-root users.

## Exercise 9: Introduction to Docker Compose (Optional/Simple)
**Objective**: Understand how to manage multi-container applications with Docker Compose.

* Prerequisite: Ensure Docker Compose is installed on your system (often comes with Docker Desktop or can be installed 
  separately).

### 1. Create a new directory for your Compose project:

* `mkdir my-compose-app`

* `cd my-compose-app`

### 2. Create a `docker-compose.yml` file:

* This file defines your services (containers) and how they relate.

* Using your preferred text editor, add the following content:

  ```yaml
  version: '3.8' # Specify the Compose file format version
  
  services:
    web:
      image: nginx:latest
      ports:
        - "8083:80"
      volumes:
        - ./html:/usr/share/nginx/html # Mount a local 'html' directory
  
  # You could add another service here, e.g., a simple Python app
  # app:
  #   image: python:3.9-slim-buster
  #   command: python -m http.server 8000
  #   ports:
  #     - "8084:8000"
  #   volumes:
  #     - ./python-app:/app
  #   working_dir: /app
  ```
### 3. Create the `html` directory and an `index.html` file:

* `mkdir html`

* `echo "<h1>Hello from Docker Compose</h1>" > html/index.html`

### 4. Start your services with Docker Compose:

* Make sure you are in the `my-compose-app` directory.

* `docker compose up -d`

* **Explanation**:

  * `up`: Creates and starts the services defined in `docker-compose.yml`.

  * `-d`: Runs the services in detached mode (background).

* **Observe**: Docker Compose will pull images (if not present) and start the containers.

### 5. Verify running services:

* `docker ps`

* **Expected**: You should see a container named my-compose-app-web-1 (or similar).

### 6. Access your web server:

* Open your web browser and navigate to http://localhost:8083 (or http://your-linux-vm-ip:8083).

* **Expected**: You should see "Hello from Docker Compose!".

### 7. Stop and remove services:

* `docker compose down`

* **Explanation**: Stops and removes all containers, networks, and volumes created by `docker compose up`.

* **Observe**: Verify with `docker ps -a` that the containers are gone.

### 8. Clean up:

* `cd ..`

* `rm -rf my-compose-app`

## Conclusion
These exercises cover the fundamental commands and concepts for managing containers. Practice these commands regularly 
to build muscle memory and confidence. Remember that the Docker documentation is an excellent resource for more in-depth
information and advanced topics!