# KittyCash 


### [Docker Setup](#docker-setup)
- [Kittycash (Flask app)](#kittycash-flask-app)  
  - [Dockerfile](#kittycash-dockerfile)
    - [gunicorn.sh](#gunicorn) 
- [DB (PostgreSQL database)](#db-postgresql-database)
  - [Dockerfile](#db-dockerfile) 
- [Redis (Caching Layer)](#redis-caching-layer)
  - [Dockerfile](#redis-dockerfile) 
- [Nginx (Reverse Proxy)](#nginx-reverse-proxy)
  - [Dockerfile](#nginx-dockerfile)
  - [nginx.conf](#nginx-conf)
- [Volumes & Networks](#volumes-and-networks)

---

## Docker Setup
- **This section covers all services orchestrated in the docker-compose.yml file.** 
  - This file defines multiple services (containers) and how they interact. Your setup includes:
     - kittycash (Flask app)
     - db (PostgreSQL database)
     - redis (Caching layer)
     - nginx (Reverse proxy)
       
These services are connected via a custom Docker network (cool_network), and the database uses a persistent volume (postgres_data_new).

**```Build Command : sudo ./build_and_push.py --env  test-docker --registry docker --build --service all```
or ```sudo python3 ./build_and_push.py --env test-docker --registry docker --build --service all```**

## Kittycash (Flask app)
**This section contains details about the Kittycash Flask application.** 
The build section in docker-compose.yml tells Docker how to create an image for the service. Instead of pulling an image from a registry like Docker Hub, you can create one from a local Docker file.
- The context: tells Docker where to look for the Dockerfile and any related files when building the image.
- The. (dot) refers to the current directory where the docker-compose.yml file is located.
- args allows passing build-time arguments to the Dockerfile.
  - ${DEPLOY} comes from an environment variable (from .env or the shell).
- Uses an image from a private registry.
  - ${IMAGE_REGISTRY} → Registry URL.
  - ${ENV} → Deployment environment (e.g., dev, prod).
  - ${TAG} → Image version.
- Mounts local directories inside the container:
  - ~/.aws:/root/.aws → Mounts AWS credentials (to access AWS services).
- tmpfs ensures temporary files are stored in memory, improving performance.
- Connects the container to the cool_network network (defined later).
- Passes environment variables to the container.
- ${} means values are loaded from .env.
- Ensures kittycash starts only after Redis is healthy.

### kittycash-dockerfile
The base image was originally python:3.8.16-alpine3.16 but was changed to Ubuntu:focal (Ubuntu 20.04) to provide a more flexible environment with more available dependencies.
- **Exposes port 8080** to allow network access to the application inside the container.
- Sets **/kittyCash** as the working directory for executing all commands.
- **Defines Build-Time Variables:**  
  - WORKINGDIR: The root directory of the project.  
  - PROJECTDIR: Path where the Flask application resides.  
  - MIGRATIONDIR: Directory for database migrations.  
  - ENTRYPOINTSCRIPT: Path to the script that runs the application (`gunicorn.sh`).  
  - DATABASEDIR: Directory for persistent database files.  
  - WEBDIR: Directory for frontend or static web files.  
  - environment: Stores the runtime environment (e.g., dev, prod).
- Prints the WORKINGDIR value for debugging purposes.
- Set Environment Variables
  - PROJECTDIR: Sets the project directory as an environment variable.
  - DEBIAN_FRONTEND=noninteractive: Prevents interactive prompts when installing packages.
  - ENVIRONMENT: Stores the runtime environment.
- Declares persistent storage locations for the database and web-related files.
- Installs required system dependencies, including:
  - Python development tools (python-dev, python3-dev).
  - **Compiler tools (g++, gcc, cmake).**
  - **Redis server (redis-server).**
  - Python package manager (python3-pip).
  - MySQL client (mysql-client).
  - **Networking tools (inetutils-ping, net-tools).**
  - Other utilities (vim, dialog, systemd, bash, git).
- Downloads and installs the **Amazon CloudWatch Agent** for monitoring.
- Copies a CloudWatch configuration file specific to the environment.
- Sets the timezone to US and installs timezone data (tzdata).
- Installs the Nginx web server.
- Prints the installed pip version.
- Replaces the default shell (sh) with bash for better script compatibility.
- Copies the following items:
  - requirements.txt (Python dependencies).
  - etc/ folder (configurations).
  - .bashrc (custom shell settings).
  - env.json (environment-specific settings).
- Setup AWS Credemtials by creating a directory for AWS credentials and copies them to /root/.aws/.
- Copies the Flask app (login/) and migration scripts (migrations/).
- Copies the gunicorn.sh and entrypoint.sh scripts.
- Creates a Python virtual environment and installs required dependencies.
- Check and Debug Installation.
  - Lists files for debugging.
  - Sets the hostname to kittycash-server.
  - Ensures gunicorn and celery are properly installed.
- Ensures that project scripts are included in the system PATH.
- Grants execution permissions to startup scripts.
- Specifies **gunicorn.sh** as the startup script.

  ##### gunicorn
  
  **This script is the main entry point for starting the kittycash Flask application. It performs the following key tasks:**

  - **Sets up necessary directories** – Ensures that required folders for storing profile and group photos exist.
  - **Adjusts permissions** – Grants full access permissions to the project directory.
  - **(Optional) Starts the Redis server** – (Commented out) If enabled, it would launch a Redis instance.
  - **Starts the Flask application with Gunicorn** – Uses gunicorn to run the Flask application (login:create_app()).
  - **Starts the Celery worker** – Launches a Celery worker to handle background tasks asynchronously.
  - **Ensures services are running** – Verifies that both Gunicorn and Celery started successfully.
  - **Waits for services** – Ensures the script doesn’t exit while the application is running.

**Breakdown**

- ```#!/bin/sh```
   - Specifies that this script should be executed using sh (Bourne shell).
   - Ensures compatibility across various Unix/Linux environments.
- ```# amazon-cloudwatch-agent-ctl -a start -m on```
   - If uncommented, this line would start the AWS CloudWatch Logs agent.
   - It helps in monitoring logs by sending them to AWS CloudWatch.
- Creates necessary directories under $PROJECTDIR for storing profile and group photos.
- -p ensures the command doesn’t fail if the directory already exists.
- The || { echo "..."; exit 1; } syntax means:
   - If mkdir fails, an error message is printed.
   - The script exits with status 1, preventing further execution.

- ```chmod 777 -R "$PROJECTDIR"```
   - Grants full read (r), write (w), and execute (x) permissions to everyone on all files and subdirectories inside $PROJECTDIR.
   - This is useful for ensuring the application has the necessary access but can be a security risk if the system is exposed.

- Start Redis Server
   - Starts a Redis server using nohup, which allows the process to continue running in the background.
   - Redirects both standard output and errors to redis_stdout.log.
   - Captures the Redis process ID (redis_pid).
   - Check if the process is running using kill -0 $redis_pid (which doesn’t terminate, just checks if it exists).
   - If Redis fails to start, prints an error message and exits.
- Start KittyCash Service with Gunicorn
   - ```nohup``` Ensures the process doesn’t stop even if the terminal session is closed.
   - ```./venv/bin/gunicorn``` Runs gunicorn from the virtual environment.
   - ```--worker-class eventlet``` Uses Eventlet for asynchronous processing.
   - ```-w 5``` Starts with 5 worker processes to handle requests concurrently.
   - ```--bind 0.0.0.0:8080``` Binds the app to port 8080 on all network interfaces.
   - ```-m 007``` Sets the umask value (file permission mode).
   - ```'login:create_app()'``` Specifies the Flask app to run. create_app() is a factory function inside login.
   - ```--error-logfile gunicorn_error.log``` Logs errors to gunicorn_error.log.
   - ```> gunicorn_stdout.log 2>&1``` Redirects both standard output and errors to gunicorn_stdout.log.
   - ```&``` Runs the process in the background.
   - Captures the process ID ***kc_service_pid*** to monitor its status.
   - Uses kill ```-0 $kc_service_pid``` to check if the Gunicorn process is running
   - If it isn’t running, prints an error message and exits.
- Start Celery Worker(Runs a Celery worker for background tasks)
   - ```nohup``` Runs in the background.
   - ```./venv/bin/celery``` -A login.celery worker: Starts Celery using the celery object from the login module.
   - ```-c 1``` Uses only 1 concurrent worker.
   - ```--without-gossip --without-mingle --without-heartbeat``` Disables unnecessary networking features (reducing overhead).
   - ```-f celery.logs``` Logs output to celery.logs.
   - ```-l debug``` Sets logging level to debug.
   - ```-P threads --pool=solo``` Uses a single-threaded execution model.
   - ```> celery_stdout.log 2>&1``` Redirects output to celery_stdout.log.
   - ```&``` Runs the process in the background.
   - Captures the process ID ***kc_celery_service_pid*** for monitoring.
   - Checks if the Celery process is running.
   - If not, prints an error and exits.
- ```wait``` suspends script execution until all specified processes (Gunicorn and Celery) have finished.
   - Prevents the script from exiting immediately, ensuring services remain running.
 

**Summary**
  - Ensures required directories exist.
  - Grants full access permissions.
  - (Optional) Starts Redis.
  - Starts the Flask app using Gunicorn.
  - Starts a Celery worker for background tasks.
  - Checks that both services are running.
  - Keeps the script alive while services are running.



---

## DB (PostgreSQL database)
**Details about the PostgreSQL database setup.**

- Builds the database image using the Dockerfile inside postgres/.
- Uses a Pre-Built Image by pulling a PostgreSQL image from the private registry.
- Sets a custom container name to **kittycash_db**
- Sets PostgreSQL credentials and database name.
- Maps PostgreSQL's default port 5432 to the host.
- Ensures the database restarts if it crashes.
- Stores PostgreSQL data in postgres_data_new (defined later).[Persistent Storage]
- Connects the container to **cool_network**.
- Health Check:
  - Uses pg_isready to check if PostgreSQL is ready.
  - Retries every 10 seconds until successful.

### db-dockerfile

- postgres:latest means the Docker image will use the latest version of PostgreSQL available.
-  ```RUN apt-get update -y ```
   - apt-get update is a command used to update the package lists for the apt package manager.
   - It ensures that your container has the latest information about the available packages and their versions from the repositories.
   - -y flag automatically confirms the update without asking for user input.
- ```RUN apt-get install -y vim procps net-tools iputils-ping```
   - This command installs additional tools and utilities inside the container that are not part of the default PostgreSQL image.
   - These tools can help with troubleshooting, monitoring, and working within the container during development or in case of an issue.
- The additional tools make it easier to:
  - Edit configuration files or logs (vim).
  - Monitor processes and system resources (procps).
  - Inspect and manage networking (net-tools, ping).


#### Configuration
- Using Docker for PostgreSQL containerization
- Persistent volume storage for database data
- Connection with Flask app

---

## Redis (Caching Layer)
**Explanation of how Redis is used for caching.**
- Builds Redis from the Dockerfile inside redis/.
- Pulls a Redis image from the private registry.[Using a Pre-Built Image]
- Sets a custom container name **kittycash_redis**
- Maps Redis default port 6379.
- Connects Redis to cool_network.
- Uses redis-cli ping to check if Redis is running.
- 
### redis-dockerfile
- **Base Image:** Starts from the official Redis image, providing the Redis server.
- **Updates the Package List:** Ensures the latest available package versions are used.
- **Installs Useful Tools:** Adds vim, procps, net-tools, and iputils-ping to help with debugging, process management, network monitoring, and editing files directly within the container.
  
#### Usage
- Caching user sessions
- Reducing database queries
- Improving response times

---

## Nginx (Reverse Proxy)
**Details about setting up Nginx as a reverse proxy.**
- Builds the Nginx image from  the docker file inside nginx/.
- Pulls an Nginx image from the private registry.[Using a Pre-Built Image]
- Sets a custom container name as ** kittycash_nginx**
- Maps port 80 on the host (HTTP).
- Mounts a custom Nginx config file.
- Ensures Nginx starts only after kittycash starts.
- Connects Nginx to cool_network.


### nginx-dockerfile

- ```FROM nginx:latest``` specifies that the Docker image will be based on the official Nginx image from Docker Hub. nginx:latest means you are using the most recent stable version of Nginx.
- Copies a custom nginx.conf file from your local directory (where the Dockerfile is located) into the container at the path /etc/nginx/nginx.conf.
- The nginx.conf file is the configuration file for Nginx, allowing you to customize how Nginx will handle incoming requests, such as defining reverse proxies, setting timeouts, and more.
- Copies a fallback HTML file from your local machine to the container.
- It places the fallback.html file inside the container at the default location for Nginx's HTML content: /usr/share/nginx/html/.
-  Installs useful debugging tools (vim, ping, etc.).
-  Opens port 80 for web traffic.
-  ```nginx``` starts the Nginx service.
-  ```-g "daemon off;"``` This keeps Nginx running in the foreground. By default, Nginx runs as a daemon (background process), but in Docker containers, you want to keep the process running in the foreground to keep the container alive. The -g "daemon off;" ensures Nginx runs in the foreground.

### nginx-conf

**This nginx.conf file configures Nginx to serve as a reverse proxy, directing incoming HTTP traffic to your Flask application (kittycash), and also providing a fallback mechanism when the Flask app is unavailable.**

- ```worker_processes auto;``` defines the number of worker processes Nginx should use.
   - auto means it will automatically detect the optimal number based on CPU cores. This helps Nginx handle more requests efficiently.
- ```events { worker_connections 1024; }```
   - Configures the event module to handle up to 1024 simultaneous connections per worker process.
   - This allows Nginx to handle a larger number of concurrent requests efficiently.
- Instructs Nginx to listen for HTTP requests on port 80, the standard HTTP port.
- The comment mentions enabling HTTP/2, but HTTP/2 is typically used with SSL. However, in this case, the configuration is for basic HTTP without SSL.
- Fallback Mechanism for Unreachable Flask App
  - ```error_page 502 503 504 /fallback.html;```
     - Specifies custom error pages to show when the Flask app is unavailable.
     - 502 → Bad Gateway (Flask app not responding).
     - 503 → Service Unavailable (Flask app temporarily down).
     - 504 → Gateway Timeout (Flask app taking too long to respond).
     - All these errors will redirect users to the fallback HTML page **(/fallback.html)**.
- ```location = /fallback.html {```
   - Defines a specific location block for /fallback.html, ensuring it serves the HTML file when Flask is down.
- ```root /usr/share/nginx/html;```
   - Specifies the directory where Nginx will look for the fallback HTML file.
   - This file is typically served to the user when the Flask app fails
- ```internal;```
   - The internal directive makes the /fallback.html URL not accessible directly from the web (only internally by Nginx), which is a security measure.
- **Below is the main reverse proxy configuration that forwards all incoming requests to the Flask application running on port 8080.**
   - ```proxy_pass http://kittycash:8080;```
      - This is the core proxy functionality.
      - proxy_pass tells Nginx to forward all incoming requests (to /) to the Flask application at http://kittycash:8080.
      - kittycash is the name of the Flask service in the Docker network (docker-compose.yml), so Nginx can resolve it as a container hostname.
      - 8080 is the port the Flask app is exposed on.
   - ```proxy_http_version 1.1;```
      - Ensures the proxy uses HTTP/1.1 when communicating with the Flask application.
   - **Header Configuration**
      - ```proxy_set_header``` allows you to customize the headers forwarded to the Flask app.
      - ```Upgrade``` and ```Connection``` headers are typically used for WebSocket upgrades.
      - ```Host``` forwards the original Host header, ensuring the Flask app knows the domain being requested.
      ```X-Real-IP``` and ```X-Forwarded-For``` help Flask detect the client's real IP address, which can be important for logging or security.
      - ```X-Forwarded-Proto``` forwards the protocol (http or https) the client used to reach the Nginx server.

   - Timeouts
      - ```proxy_connect_timeout``` Sets the timeout for establishing a connection with the upstream service (Flask).
      - ```proxy_read_timeout``` Sets the timeout for reading the response from the upstream service (Flask).
    - Error Handling
      - Ensures Nginx intercepts error responses from Flask (like 500 Internal Server Error) and can handle them gracefully (such as redirecting to a custom error page).
     
- **Summary of Why This Configuration is Needed**
  - Nginx acts as an intermediary between external clients and the Flask application, forwarding HTTP requests to the correct service.
  - This allows you to keep the internal Flask app isolated while managing external traffic at the Nginx level.
  If Flask is unavailable (due to crashes or downtime), Nginx serves as a fallback page to improve the user experience and prevent error messages from appearing.
  - Nginx handles incoming requests efficiently and can support WebSocket connections if needed.
  - Timeout and header management ensure smooth communication between Nginx and Flask.
  - Headers like ```X-Real-IP``` and ```X-Forwarded-For``` help track and secure the origin of traffic, which could be useful for logging, security, or troubleshooting.


#### How It All Works Together
- **Step 1:** A client sends a request to the server on port 80.
- **Step 2:** Nginx checks if Flask is available. If the Flask app is unreachable (502/503/504 errors), it serves the fallback HTML page.
- **Step 3:** If Flask is up and running, Nginx forwards the request to the Flask application running on port 8080.
- **Step 4:** The Flask app processes the request and sends the response back to Nginx, which then forwards it to the client.

---

## volumes-and-networks

**In Docker Compose, volumes and networks play a crucial role in managing data persistence and service communication. Let’s break them down clearly.**

- Creates a named volume called postgres_data_new
- Ensures PostgreSQL data is not lost when the container stops or is removed.
- Docker stores the volume's data outside the container.
- **By default, when a container is deleted, all its data is lost. Using volumes, data remains persistent even if the container is restarted.**
- The PostgreSQL container (db) mounts postgres_data_new at: ```volumes:postgres_data_new:/var/lib/postgresql/data```
  - ```var/lib/postgresql/data ``` → Default directory where PostgreSQL stores its data.
  - This ensures that even if the database container is removed and recreated, all tables, records, and indexes remain intact.
- The first time docker-compose-up runs, Docker creates postgres_data_new if it doesn’t exist.
- If the container is restarted, it reuses the same volume.
> [!WARNING]
> If you remove the volume with docker volume rm postgres_data_new, all data is lost.
- Creates a user-defined Docker network named ***cool_network.**
- Ensures all services in the Compose file communicate securely without exposing ports externally.
- Each service in docker-compose.yml automatically connects to cool_network.
- This allows containers to communicate by name, instead of using IP addresses.


  ### How They Work Together

  - ```volumes``` ensure that the database data remains even if the database container is restarted.
  - ```networks``` allow services to communicate without needing external ports.


    ## Additional Documentation  
- [Query](query.md)  
   


  



