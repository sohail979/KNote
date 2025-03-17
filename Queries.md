
# Debugging Approaches for Kittycash (Dockerized Flask App)

This document provides troubleshooting steps for diagnosing and resolving common issues in the Kittycash application running inside a Docker container.

---

## **1. Check If the Container Exited** ✅
- **Command:**  
  ```docker ps -a | grep kittycash```
- **Result:**
   -bb2ebdf8b7fb   vamsikrishnasadhu.hub.docker.com/dev-kittycash/kittycash:latest  
"/kittyCash/gunicorn…"   5 minutes ago   Exited (2) 4 minutes ago   kittycash 

## **2.Check Database & Redis Connectivity(Manually connect to the database from another container)**
- **Command:** ✅
  - docker exec -it kittycash_db psql -U postgres -d mydb
- **Result:**
  -  Connecting 
  -  sql: error: connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: FATAL:  role "-d" does not exist - This error with  the following command **docker exec -it kittycash_db psql -U $POSTGRES_USER -d $DB_NAME**


## **3.Run the Container in Debug Mode** 🔴
- **Command:**  
  ``` sudo docker run --rm -it --entrypoint /bin/sh vamsikrishnasadhu.hub.docker.com/dev-kittycash/kittycash:latest ```
   - **Approach1:**  Manually Start Flask
   - **Command:**  
  ```python -m flask run --host=0.0.0.0 --port=8080```
   - **Result:**
  No Module Found flask 

   - **Approach2:**  Using Gunicorn 
   - **Command:**  
  ``` gunicorn -b 0.0.0.0:8080 "login.__init__:application"```
   - **Result:**
     - gunicorn: command not found

##  **4.List all environment variables inside the kittycash container**
- **Command:**  🔴
  ```docker run --rm -it --entrypoint /bin/sh vamsikrishnasadhu.hub.docker.com/dev-kittycash/kittycash:latest -c "env"```
- **Result:**
HOSTNAME=caf0969e995d
PWD=/kittyCash
HOME=/root
TERM=xterm
ENVIRONMENT=dev
SHLVL=0
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
PROJECTDIR=/kittyCash/login
DEBIAN_FRONTEND=noninteractive
_=/usr/bin/env
             
these are showing results at path _=/usr/bin/env not environmen variables as expected like DB_HOST,REDIS_URL etc.,             




## **5.Running the image** 🔴
- **Command:**
    ```sudo docker run --rm -it vamsikrishnasadhu.hub.docker.com/dev-kittycash/kittycash:latest /bin/bash```
- **Result:**
  - The entrypoint script (gunicorn.sh) is failing or exiting immediately.
  - The CMD or ENTRYPOINT process finishes, causing the container to stop.
  - There is a missing dependency or a failure during startup.
 
 - **Approach1** Check Container logs
    - ```docker logs $(docker ps -a -q --filter ancestor=vamsikrishnasadhu.hub.docker.com/dev-kittycash/kittycash:latest --filter status=exited)```
   - **Result**
    - Cannot connect to the Docker daemon at unix:///home/shaik/.docker/desktop/docker.sock. Is the docker daemon running?
"docker logs" requires exactly 1 argument.
See 'docker logs --help'.

Usage:  docker logs [OPTIONS] CONTAINER

Fetch the logs of a container

  - **Approach2** Run the Container in Debug Mode
     - ```docker run --rm -it --entrypoint /bin/bash vamsikrishnasadhu.hub.docker.com/dev-kittycash/kittycash:latest```
     - **Result**
     - Bash Opened So ptobably issue with **gunicorn.sh**
        
  - **Approach3** Check the Entrypoint (gunicorn.sh)
      - ```docker run --rm -it vamsikrishnasadhu.hub.docker.com/dev-kittycash/kittycash:latest cat /kittyCash/gunicorn.sh```
    - **Result** -
      - Exited without showing anything
        
  - **Approach4** Verify if the script is correct and has executable permissions:
      - ```docker run --rm -it vamsikrishnasadhu.hub.docker.com/dev-kittycash/kittycash:latest ls -l /kittyCash/gunicorn.sh```
        
  - **Result**
     - Exited without showing anything

  - **Approach5** Try Running the Entrypoint Manually  If the container starts with /bin/bash, manually run the entrypoint inside
      - ```/kittyCash/gunicorn.sh```
  - **Result**
      - Exited without showing anything
  - **Approach6** Check Environment Variables
      - ```docker run --rm -it --entrypoint /bin/bash vamsikrishnasadhu.hub.docker.com/dev-kittycash/kittycash:latest -c "env"```
  - **Result**
      ```
      HOSTNAME=622e9ae80f6a
      PWD=/kittyCash
      HOME=/root
      TERM=xterm
      ENVIRONMENT=dev
      SHLVL=0
      PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
      PROJECTDIR=/kittyCash/login
      DEBIAN_FRONTEND=noninteractive
      _=/usr/bin/env
     ```

## **6. Check If the Container Exited** 🔴
- **Command:**  
  ```docker logs $(docker ps -a | grep kittycash | awk '{print $1}')```
- **Result:**
   - Cannot connect to the Docker daemon at unix:///home/shaik/.docker/desktop/docker.sock. Is the docker daemon running?
"docker logs" requires exactly 1 argument.
See 'docker logs --help'.

Usage:  docker logs [OPTIONS] CONTAINER

Fetch the logs of a container

## **7. Python & Gunicorn Issues** 🔴
- **Command:**  
  ```docker run --rm -it --entrypoint /bin/bash vamsikrishnasadhu.hub.docker.com/dev-kittycash/kittycash:latest``` and then check if flask and Gunicorn are intstalled 
- **Result:**
   - They are not installed using ```pip install flask gunicorn```

## **8. gunicorn.sh Permissions** ✅
- **Command:**  
  - docker run --rm -it --entrypoint /bin/bash vamsikrishnasadhu.hub.docker.com/dev-kittycash/kittycash:latest
ls -l /kittyCash/gunicorn.sh
  - ```ls -l /kittyCash/gunicorn.sh```
- **Result:**
   - -rwxrwxrwx 1 root root 1593 Jan 26 18:41 /kittyCash/gunicorn.sh
 


## **9. Fix the Database Connectivity**✅
- **Command:**  
  ```docker exec -it kittycash_db psql -U postgres -d mydb```
- **Result:**
   - Success 
    

## **10. Fix Environment Variables**🔴
- **Command:**  
  ```docker exec -it kittycash /bin/bash``` then check if .env exists using ```ls -l /kittyCash/.env```
- **Result:**
   - Error response from daemon: container ea0bc5f052aeaa37ef9543ea22d037365bbf877ffa9dd3a39959a4309a1cd9a7 is not running

## **11. Container Crashes Immediately-If it crashes due to gunicorn.sh, try running it manually:** 🔴
- **Command:**  
  - docker run --rm -it --entrypoint /bin/bash vamsikrishnasadhu.hub.docker.com/dev-kittycash/kittycash:latest
sh /kittyCash/gunicorn.sh
  - pip install flask gunicorn
  - python3 -m flask run --host=0.0.0.0 --port=8080
- **Result:**
   - Error: Could not locate a Flask application. Use the 'flask --app' option, 'FLASK_APP' environment variable, or a 'wsgi.py' or 'app.py' file in the current directory.



   
## **12.Check Container Logs**
- **Command:**  
  ``` sudo docker logs kittycash```
- **Result:**
   - It's not returning anything tell me how to check in log files such as gnucorn_error.log and celery.logs 


## **13.Inspect Running Processes Inside the Container**
- **Command:**  
  ```sudo docker exec -it kittycash ps aux```
- **Result:**
    ```
      USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
    root           1  0.0  0.0   3984  2816 ?        Ss   08:26   0:00 /bin/sh /kittyCash/gunicorn.sh /kittyCash/gunicorn.sh &
    root          12  0.0  0.1  43444 34176 ?        S    08:26   0:00 /kittyCash/venv/bin/python3 ./venv/bin/gunicorn --worker-class eventlet -w 5 --bind 0.0.0.0:8080 -m 007 login:crea
    root          13  0.2  0.3 142040 120864 ?       S    08:26   0:01 /kittyCash/venv/bin/python3 ./venv/bin/celery -A login.celery worker -c 1 --without-gossip --without-mingle --with
    root          16  0.2  0.4 172452 146608 ?       S    08:26   0:01 /kittyCash/venv/bin/python3 ./venv/bin/gunicorn --worker-class eventlet -w 5 --bind 0.0.0.0:8080 -m 007 login:crea
    root          17  0.2  0.4 172040 146028 ?       S    08:26   0:02 /kittyCash/venv/bin/python3 ./venv/bin/gunicorn --worker-class eventlet -w 5 --bind 0.0.0.0:8080 -m 007 login:crea
    root          18  0.2  0.4 172128 146148 ?       S    08:26   0:01 /kittyCash/venv/bin/python3 ./venv/bin/gunicorn --worker-class eventlet -w 5 --bind 0.0.0.0:8080 -m 007 login:crea
    root          19  0.2  0.4 171980 146604 ?       S    08:26   0:01 /kittyCash/venv/bin/python3 ./venv/bin/gunicorn --worker-class eventlet -w 5 --bind 0.0.0.0:8080 -m 007 login:crea
    root          20  0.2  0.4 170432 144096 ?       S    08:26   0:01 /kittyCash/venv/bin/python3 ./venv/bin/gunicorn --worker-class eventlet -w 5 --bind 0.0.0.0:8080 -m 007 login:crea
    root          21  0.0  0.0   5900  2816 pts/0    Rs+  08:38   0:00 ps aux

  ```

## **14.Test the Application Internally**
- **Command:**  
  ```sudo docker exec -it kittycash curl http://localhost:8080```
- **Result:**
    ```
    <!doctype html>
<html lang=en>
<title>500 Internal Server Error</title>
<h1>Internal Server Error</h1>
<p>The server encountered an internal error and was unable to complete your request. Either the server is overloaded or there is an error in the application.</p>
  ```

## **15.Verify Port Mapping and Network Connectivity** 
- **Command:**  
  ```sudo docker ps```
- **Result:**
    ```
  4d8f00932f2e   vamsikrishnasadhu.hub.docker.com/test-docker-kittycash/kittycash:latest   "/kittyCash/gunicorn…"   29 minutes ago   Up 29 minutes             0.0.0.0:8080->8080/tcp, :::8080->8080/tcp   kittycash
    ```


## **16.Test Directly on the Host** 
- **Command:**  
  ```sudo curl http://localhost:8080```
- **Result:**
    ```
     <!doctype html>
      <html lang=en>
      <title>500 Internal Server Error</title>
      <h1>Internal Server Error</h1>
      <p>The server encountered an internal error and was unable to complete your request. Either the server is overloaded or there is an error in the application.</p>
    ```

## **17.Check the Nginx Proxy** 
- **Command:**  
  ```sudo curl http://localhost```
- **Result:**
    ```
       <!doctype html>
      <html lang=en>
      <title>500 Internal Server Error</title>
      <h1>Internal Server Error</h1>
      <p>The server encountered an internal error and was unable to complete your request. Either the server is 
      overloaded or there is an error in the application.</p>
    ```

---

## **18.Check Gunicorn and Celery Logs -checking logs inside the container:** 
- **Command:**  
  ```sudo docker exec -it kittycash ls -l /kittyCash/```
  -  If you find gunicorn_error.log or celery.logs, view their contents:
     - ``` sudo docker exec -it kittycash cat /kittyCash/gunicorn_error.log ```
     - ``` sudo docker exec -it kittycash cat /kittyCash/celery.logs ```
- **Result:**
  -  gunicorn_error.log :
      ```
      [2025-02-01 08:26:46 +0000] [12] [INFO] Starting gunicorn 23.0.0
      [2025-02-01 08:26:46 +0000] [12] [INFO] Listening at: http://0.0.0.0:8080 (12)
      [2025-02-01 08:26:46 +0000] [12] [INFO] Using worker: eventlet
      [2025-02-01 08:26:46 +0000] [16] [INFO] Booting worker with pid: 16
      [2025-02-01 08:26:46 +0000] [17] [INFO] Booting worker with pid: 17
      [2025-02-01 08:26:46 +0000] [18] [INFO] Booting worker with pid: 18
      [2025-02-01 08:26:46 +0000] [19] [INFO] Booting worker with pid: 19
      [2025-02-01 08:26:46 +0000] [20] [INFO] Booting worker with pid: 20
      
      ```
  -  celery.logs:
      ```
      [2025-02-01 08:26:48,087: DEBUG/MainProcess] | Worker: Preparing bootsteps.
      [2025-02-01 08:26:48,088: DEBUG/MainProcess] | Worker: Building graph...
      [2025-02-01 08:26:48,089: DEBUG/MainProcess] | Worker: New boot order: {StateDB, Beat, Timer, Hub, Pool, 
       Autoscaler, Consumer}
      [2025-02-01 08:26:48,093: DEBUG/MainProcess] | Consumer: Preparing bootsteps.
      [2025-02-01 08:26:48,093: DEBUG/MainProcess] | Consumer: Building graph...
      [2025-02-01 08:26:48,103: DEBUG/MainProcess] | Consumer: New boot order: {Connection, Agent, Events, Heart, 
      Mingle, Tasks, Control, Gossip, event loop}
      [2025-02-01 08:26:48,124: DEBUG/MainProcess] | Worker: Starting Pool
      [2025-02-01 08:26:48,124: DEBUG/MainProcess] ^-- substep ok
      [2025-02-01 08:26:48,124: DEBUG/MainProcess] | Worker: Starting Consumer
      [2025-02-01 08:26:48,124: DEBUG/MainProcess] | Consumer: Starting Connection
      [2025-02-01 08:26:48,126: ERROR/MainProcess] consumer: Cannot connect to redis://localhost:6379/0: Error 
       111 connecting to localhost:6379. ECONNREFUSED..
      Trying again in 2.00 seconds... (1/100)
      
      ```

## **19.Check Flask App Logs** 
- **Command:**  
  ```sudo docker exec -it kittycash cat kittycash.log```
- **Result:**
    ```
          File "/kittyCash/venv/lib/python3.8/site-packages/redis/connection.py", line 690, in _connect
        raise err
      File "/kittyCash/venv/lib/python3.8/site-packages/redis/connection.py", line 678, in _connect
        sock.connect(socket_address)
      File "/kittyCash/venv/lib/python3.8/site-packages/eventlet/greenio/base.py", line 267, in connect
        socket_checkerr(fd)
      File "/kittyCash/venv/lib/python3.8/site-packages/eventlet/greenio/base.py", line 51, in socket_checkerr
        raise socket.error(err, errno.errorcode[err])
    ConnectionRefusedError: [Errno 111] ECONNREFUSED
    ```
   

## **20.Verify Network and Nginx- From inside the Nginx container** 
- **Command:**  
  ```sudo docker exec -it 3c30e890de9d curl -v http://kittycash:8080```
- **Result:**
    ```
          *   Trying 192.168.80.4:8080...
      * Connected to kittycash (192.168.80.4) port 8080 (#0)
      > GET / HTTP/1.1
      > Host: kittycash:8080
      > User-Agent: curl/7.88.1
      > Accept: */*
      > 
      < HTTP/1.1 500 INTERNAL SERVER ERROR
      < Server: gunicorn
      < Date: Sat, 01 Feb 2025 09:27:08 GMT
      < Connection: keep-alive
      < Content-Type: text/html; charset=utf-8
      < Content-Length: 265
      < Access-Control-Allow-Origin: *
      < 
      <!doctype html>
      <html lang=en>
      <title>500 Internal Server Error</title>
      <h1>Internal Server Error</h1>
      <p>The server encountered an internal error and was unable to complete your request. Either the server is overloaded or there is an error in the application.</p>
      * Connection #0 to host kittycash left intact
   ```
---

## **21.Celery Cannot Connect to Redis (ECONNREFUSED error)** 

- The Celery worker is trying to connect to Redis at redis://localhost:6379/0, but the connection is refused.
- **Command:**  
  ```sudo docker ps --filter "name=redis"```
- **Result:**
     ```
     07d28f23a209   vamsikrishnasadhu.hub.docker.com/test-docker-kittycash/redis:latest   "docker-entrypoint.s…"   9 minutes ago   Up 9 minutes (healthy)   0.0.0.0:6379->6379/tcp, :::6379->6379/tcp   kittycash_redis
     ```
   
## **22.Flask App Throws 500 Internal Server Error Checking Full Logs)** 
- Flask app is running (gunicorn_error.log shows it started fine).
- Nginx successfully connects to kittycash:8080.
- However, the app responds with 500 Internal Server Error.
  
- **Command:**  
  ``` sudo docker exec -it kittycash cat /kittyCash/kittycash.log```
- **Result:**
    ```
          02/01/2025 09:27:08 AM Exception on / [GET]
      Traceback (most recent call last):
        File "/kittyCash/venv/lib/python3.8/site-packages/redis/connection.py", line 624, in connect
          sock = self.retry.call_with_retry(
        File "/kittyCash/venv/lib/python3.8/site-packages/redis/retry.py", line 46, in call_with_retry
          return do()
        File "/kittyCash/venv/lib/python3.8/site-packages/redis/connection.py", line 625, in <lambda>
          lambda: self._connect(), lambda error: self.disconnect(error)
        File "/kittyCash/venv/lib/python3.8/site-packages/redis/connection.py", line 690, in _connect
          raise err
        File "/kittyCash/venv/lib/python3.8/site-packages/redis/connection.py", line 678, in _connect
          sock.connect(socket_address)
        File "/kittyCash/venv/lib/python3.8/site-packages/eventlet/greenio/base.py", line 267, in connect
          socket_checkerr(fd)
        File "/kittyCash/venv/lib/python3.8/site-packages/eventlet/greenio/base.py", line 51, in socket_checkerr
          raise socket.error(err, errno.errorcode[err])
      ConnectionRefusedError: [Errno 111] ECONNREFUSED
      
      During handling of the above exception, another exception occurred:
      
      Traceback (most recent call last):
        File "/kittyCash/venv/lib/python3.8/site-packages/flask/app.py", line 2525, in wsgi_app
          response = self.full_dispatch_request()
        File "/kittyCash/venv/lib/python3.8/site-packages/flask/app.py", line 1822, in full_dispatch_request
          rv = self.handle_user_exception(e)
        File "/kittyCash/venv/lib/python3.8/site-packages/flask_cors/extension.py", line 176, in wrapped_function
          return cors_after_request(app.make_response(f(*args, **kwargs)))
        File "/kittyCash/venv/lib/python3.8/site-packages/flask/app.py", line 1818, in full_dispatch_request
          rv = self.preprocess_request()
        File "/kittyCash/venv/lib/python3.8/site-packages/flask/app.py", line 2309, in preprocess_request
          rv = self.ensure_sync(before_func)()
        File "/kittyCash/venv/lib/python3.8/site-packages/flask_limiter/extension.py", line 1168, in _check_request_limit
          raise e
        File "/kittyCash/venv/lib/python3.8/site-packages/flask_limiter/extension.py", line 1149, in _check_request_limit
          self.__evaluate_limits(endpoint, all_limits)
        File "/kittyCash/venv/lib/python3.8/site-packages/flask_limiter/extension.py", line 1086, in __evaluate_limits
          if not method(lim.limit, *args, **kwargs):
        File "/kittyCash/venv/lib/python3.8/site-packages/limits/strategies.py", line 84, in hit
          return cast(MovingWindowSupport, self.storage).acquire_entry(
        File "/kittyCash/venv/lib/python3.8/site-packages/limits/storage/base.py", line 27, in inner
          return fn(*args, **kwargs)
        File "/kittyCash/venv/lib/python3.8/site-packages/limits/storage/redis.py", line 233, in acquire_entry
          return super()._acquire_entry(key, limit, expiry, self.storage, amount)
        File "/kittyCash/venv/lib/python3.8/site-packages/limits/storage/redis.py", line 108, in _acquire_entry
          acquired = self.lua_acquire_window([key], [timestamp, limit, expiry, amount])
        File "/kittyCash/venv/lib/python3.8/site-packages/redis/commands/core.py", line 5794, in __call__
          return client.evalsha(self.sha, len(keys), *args)
        File "/kittyCash/venv/lib/python3.8/site-packages/redis/commands/core.py", line 5181, in evalsha
          return self._evalsha("EVALSHA", sha, numkeys, *keys_and_args)
        File "/kittyCash/venv/lib/python3.8/site-packages/redis/commands/core.py", line 5165, in _evalsha
          return self.execute_command(command, sha, numkeys, *keys_and_args)
        File "/kittyCash/venv/lib/python3.8/site-packages/redis/client.py", line 1255, in execute_command
          conn = self.connection or pool.get_connection(command_name, **options)
        File "/kittyCash/venv/lib/python3.8/site-packages/redis/connection.py", line 1427, in get_connection
          connection.connect()
        File "/kittyCash/venv/lib/python3.8/site-packages/redis/connection.py", line 630, in connect
          raise ConnectionError(self._error_message(e))
      redis.exceptions.ConnectionError: Error 111 connecting to localhost:6379. ECONNREFUSED.

    ```

  **Cause**
   - Celery is trying to connect to Redis at redis://localhost:6379/0

     ![image](https://github.com/user-attachments/assets/db4c7d4d-7953-44de-8681-f6e22039c4af)

  **Solution**
 > [!IMPORTANT]
 > Assign **redis_server** value to kittycash_redis as we are running the docker in test-docker environment and its not mentioned in the given list.

     



## **23.Manually test Flask inside the container:** 
- **Command:**  
  ```sudo docker exec -it kittycash curl -v http://localhost:8080```
   - If it also returns a 500 error, **check for missing environment variables**:
      - ```sudo docker exec -it kittycash env```
- **Result:**
   - After checking environment variables it returned:
      ```
              
        PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
        HOSTNAME=b28c60df4abb
        TERM=xterm
        DB_NAME={DB_NAME}
        AWS_DEFAULT_REGION=us-west-1
        DB_ADDRESS={DB_ADDRESS}
        DB_PORT={DB_PORT}
        AWS_ACCESS_KEY_ID=AKIAQ4NSA6ROB3GDXCP7
        AWS_SECRET_ACCESS_KEY=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
        ENV=test-docker-kittycash
        POSTGRES_USER={POSTGRES_USER}
        POSTGRES_PASSWORD={POSTGRES_PASSWORD}
        PROJECTDIR=/kittyCash/login
        DEBIAN_FRONTEND=noninteractive
        ENVIRONMENT=test-docker
        HOME=/root

      ```
     - My .env is like this 

       ```
        AWS_ACCOUNT_ID=061039768668
        AWS_REGION=us-west-1
        TAG=latest
        ENV='test-docker-kittycash'
        # For Docker Hub
        DOCKERHUB_USERNAME=vamsikrishnasadhu
        DOCKERHUB_TOKEN=dckr_pat_xOKtEe7OOWwdKnr9EVdJsATT1OM
        IMAGE_REGISTRY='vamsikrishnasadhu.hub.docker.com'
        # For Amazon ECR
        # IMAGE_REGISTRY=${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
        DEPLOY='test-docker'
        AWS_ACCESS_KEY_ID=AKIAQ4NSA6ROB3GDXCP7
        AWS_SECRET_ACCESS_KEY=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
        AWS_DEFAULT_REGION=us-west-1
        APP_NAME=kittycash
       
  
        ```

       
---

       
## **24.Check Nginx Logs** 
- **Command:**  
  ```sudo docker logs kittycash_nginx```
- **Result:**
  ```
    docker-entrypoint.sh: Configuration complete; ready for start up
      2025/02/06 05:50:57 [error] 28#28: *1 upstream timed out (110: Connection timed out) while reading 
      response header from upstream, client: 192.168.96.1, server: , request: "GET / HTTP/1.1", upstream: 
      "http://192.168.96.4:8080/", host: "localhost"
      192.168.96.1 - - [06/Feb/2025:05:50:57 +0000] "GET / HTTP/1.1" 504 1064 "-" "Mozilla/5.0 (X11; Linux 
      x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/123.0.0.0 Safari/537.36"
      2025/02/06 05:51:07 [error] 28#28: *1 upstream timed out (110: Connection timed out) while reading 
      response header from upstream, client: 192.168.96.1, server: , request: "GET / HTTP/1.1", upstream: 
      "http://192.168.96.4:8080/", host: "localhost"
      192.168.96.1 - - [06/Feb/2025:05:51:07 +0000] "GET / HTTP/1.1" 504 1064 "-" "Mozilla/5.0 (X11; Linux 
      x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/123.0.0.0 Safari/537.36
  ```


## **25. Check Flask Logs (KittyCash)** 
- **Command:**  
  ```sudo docker logs kittycash```
- **Result:**
    Returned Nothing 

## **26. Verify Flask (Gunicorn) is Running** 
- **Command:**  
  ```sudo docker exec -it kittycash ps aux```
- **Result:**
  ```
   USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
    root           1  0.0  0.0   3984  2944 ?        Ss   05:50   0:00 /bin/sh /kittyCash/gunicorn.sh /kittyCash/gunicorn.sh &
    root          12  0.0  0.1  43440 34020 ?        S    05:50   0:00 /kittyCash/venv/bin/python3 ./venv/bin/gunicorn --worker-class eventlet -w 5 --bind 0.0.0.0:8080 -m 007 login:creat
    root          13  0.2  0.3 144880 124080 ?       S    05:50   0:01 /kittyCash/venv/bin/python3 ./venv/bin/celery -A login.celery worker -c 1 --without-gossip --without-mingle --witho
    root          16  0.2  0.4 173648 147688 ?       S    05:50   0:01 /kittyCash/venv/bin/python3 ./venv/bin/gunicorn --worker-class eventlet -w 5 --bind 0.0.0.0:8080 -m 007 login:creat
    root          17  0.2  0.4 173844 147744 ?       S    05:50   0:01 /kittyCash/venv/bin/python3 ./venv/bin/gunicorn --worker-class eventlet -w 5 --bind 0.0.0.0:8080 -m 007 login:creat
    root          18  0.2  0.4 173452 147440 ?       S    05:50   0:01 /kittyCash/venv/bin/python3 ./venv/bin/gunicorn --worker-class eventlet -w 5 --bind 0.0.0.0:8080 -m 007 login:creat
    root          19  0.2  0.4 173412 147676 ?       S    05:50   0:01 /kittyCash/venv/bin/python3 ./venv/bin/gunicorn --worker-class eventlet -w 5 --bind 0.0.0.0:8080 -m 007 login:creat
    root          20  0.2  0.4 173424 147660 ?       S    05:50   0:01 /kittyCash/venv/bin/python3 ./venv/bin/gunicorn --worker-class eventlet -w 5 --bind 0.0.0.0:8080 -m 007 login:creat
    root          21  0.0  0.0   5900  2816 pts/0    Rs+  05:57   0:00 ps aux
  
  ```


## **27.Verify Nginx Reverse Proxy Configuration**
- **Command:**  
    My nginx.conf is as follows 
      ```
             worker_processes auto;
        events {
            worker_connections 1024;
        }
        http {
            server {
                listen 80;  # Enable HTTP/2 on port 80 without SSL
        
                # Fallback location to serve an HTML file if the Flask app is unreachable
                error_page 502 503 504 /fallback.html;
                location = /fallback.html {
                    root /usr/share/nginx/html;
                    internal;
                }
        
                location / {
                    proxy_pass http://kittycash:8080;
                    proxy_http_version 1.1;  # Use HTTP/1.1 for upstream as Flask might not support HTTP/2
                    proxy_set_header Upgrade $http_upgrade;
                    proxy_set_header Connection "upgrade";
                    proxy_set_header Host $host;
                    proxy_set_header X-Real-IP $remote_addr;
                    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                    proxy_set_header X-Forwarded-Proto $scheme;
                    proxy_connect_timeout 5s;
                    proxy_read_timeout 10s;
                    proxy_intercept_errors on;
                }
        
                 # Route Swagger requests to Flask API
                location /swagger {
                    proxy_pass http://kittycash:8080/swagger;
                    proxy_set_header Host $host;
                    proxy_set_header X-Real-IP $remote_addr;
                    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                    proxy_set_header X-Forwarded-Proto $scheme;
                }
            }
        }

      ```
- **Result:**


## **28.Verify Database Connection**
- **Command:**  
   ```
     docker exec -it kittycash bash
     env | grep DB
   ```
   - After enterring into bash executed this command:
      ```
      sudo docker exec -it kittycash_db psql -U postgres -d mydb
      ```
- **Result:**
      bash: sudo: command not found



## **29.Verify Ports**
- **Command:**  
  ```sudo netstat -tulnp | grep LISTEN ```
- **Result:**
   ```
    tcp        0      0 127.0.0.1:9000          0.0.0.0:*               LISTEN      864/zstunnel        
    tcp        0      0 0.0.0.0:9000            0.0.0.0:*               LISTEN      864/zstunnel        
    tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN      20785/docker-proxy  
    tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      687/systemd-resolve 
    tcp        0      0 0.0.0.0:6379            0.0.0.0:*               LISTEN      20114/docker-proxy  
    tcp        0      0 0.0.0.0:5432            0.0.0.0:*               LISTEN      20096/docker-proxy  
    tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN      848/cupsd           
    tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      20863/docker-proxy  
    tcp        0      0 0.0.0.0:33221           0.0.0.0:*               LISTEN      852/pmd             
    tcp6       0      0 ::1:631                 :::*                    LISTEN      848/cupsd           
    tcp6       0      0 :::8080                 :::*                    LISTEN      20792/docker-proxy  
    tcp6       0      0 :::6379                 :::*                    LISTEN      20128/docker-proxy  
    tcp6       0      0 :::5432                 :::*                    LISTEN      20102/docker-proxy  
    tcp6       0      0 :::80                   :::*                    LISTEN      20869/docker-proxy
    
   ```
    


## **30.Manually test Flask inside the container**
- **Command:**  
  ```docker exec -it kittycash bash```
  ```curl -I http://localhost:8080```

- **Result:**
   For curl -I http://localhost:8080
   ```
    HTTP/1.1 404 NOT FOUND
    Server: gunicorn
    Date: Thu, 06 Feb 2025 06:30:36 GMT
    Connection: keep-alive
    Content-Type: text/html; charset=utf-8
    Content-Length: 207
    Access-Control-Allow-Origin:
   
   ```
   For
    gunicorn -w 4 -b 0.0.0.0:8080 wsgi:application

    gunicorn command not found 


## **31. Check Flask is Running & Listening on Port 8080**
- **Command:**  
    ```
      sudo docker exec -it kittycash bash
    ```
  Then
    ```
    netstat -tulnp | grep 8080
    ```
   
- **Result:**
   ```
    tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN      1234/python
   
   ```
   

## **32.Nginx might not be resolving kittycash correctly.Find Flask's actual container IP**
- **Command:**  
  ```sudo docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' kittycash```
- **Result:**
   172.24.0.4

   Swagger is accessible at  ```http://172.24.0.4:8080/swagger/```

   ```http://172.25.0.4:8080/swagger/```


## **33.Run gunicorn in debugging mode**
- **Command:**  
  ``` sudo docker run --rm -it --entrypoint /bin/sh vamsikrishnasadhu.hub.docker.com/test-docker-kittycash/kittycash:latest ```
  Then inside bash
  ```./venv/bin/gunicorn --worker-class eventlet -w 5 --bind 0.0.0.0:8080 'login:create_app()'```
- **Result:**


## **34.Verify That Flask is Serving Swagger at /swagger**
- **Command:**  
  ``` sudo docker exec -it kittycash sh``` and then ```curl http://localhost:8080/swagger```
- **Result:**


## **35.Running Container's Bash Shell**
- **Command:**  
  ```sudo docker exec -it kittycash /bin/sh```
    - command to see the last 50 lines of the log ```tail -n 50 /kittyCash/kittycash.log```
    - Monitor Logs in Real-Time ```tail -f /kittyCash/kittycash.log```
    - Search For Errors ```grep -i "error" /kittyCash/kittycash.log``` / ```grep -i "invalid" 
         /kittyCash/kittycash.log```
    -  Check Logs with Timestamps ```grep "2024-02-28" /kittyCash/kittycash.log```
    -  View Logs Without Entering the Container ```docker exec -it kittycash tail -f /kittyCash/kittycash.log``` 
  
- **Result:**


## **36.Debug Swagger URL Failure**
- **Command:**  
  ```curl -v http://localhost:8080/swagger```

## **37.Connect to PostgreSQL from Inside the Container**
- **Command:**  
  ```docker exec -it kittycash_db psql -U postgres -d mydb```
   - Command to insert data
      ```
      INSERT INTO kittycashUser (username, password, authenticated) 
      VALUES ('testuser@example.com', crypt('testpassword', gen_salt('bf')), TRUE);
      ```
  - Create table 
    ```
    CREATE TABLE kittycashUser (
    id SERIAL PRIMARY KEY,
    username VARCHAR(256) UNIQUE NOT NULL,
    password VARCHAR(512) NOT NULL,
    address VARCHAR(2000),
    firstname VARCHAR(1000),
    lastname VARCHAR(1000),
    temppassword VARCHAR(1000),
    authenticated BOOLEAN DEFAULT FALSE,
    authToken VARCHAR(2000),
    refresh_token VARCHAR(512)
    );
    ```
  - Check if table exists
    ```
    \dt
    ```
      
  
## **38.Connect to SQLite from Inside the Container**
- **Command:**  
  ```sudo docker exec -it kittycash bash``` then ```sqlite3 /kittyCash/instance/kittycash.sqlite``` install using ```apt update && apt install sqlite3 -y ``` or ```sudo docker exec -it kittycash sqlite3 /kittyCash/instance/kittycash.sqlite```
  - Command to check count of records ```SELECT COUNT(*) FROM kittycash_profile;```
  - Command to check existing tables ```.tables```
  - Command to check schema ```.schema kittycash_profile```
  -  Insert Test User into kittycash_profile
  ```
       INSERT INTO kittycash_profile (
        pid, username, address, city, state, postalcode, firstname, lastname, 
        alternateemail, contactno, age, day, month, year, activeFrom, 
        profilePhoto, status, credit, country, twophaseauth, biometric, 
        deviceid, osname, stripestatus, paypalstatus, idnumber, ssnlast4, 
        identitystatus, sid, identity
        ) VALUES (
        'test_pid_123', 'testuser@example.com', '123 Test Street', 'TestCity', 
        'TestState', '12345', 'Test', 'User', 'test.alt@example.com', '1234567890', 
        30, 1, 1, 1995, datetime('now'), NULL, 1, 100.0, 'TestCountry', 
        0, NULL, 'test_device_001', 'TestOS', 1, 1, 'ID123456', 1234, 1, 
        'test_sid_001', 'test_identity'
        );

  ```
  - Verify the Inserted Data ``` SELECT * FROM kittycash_profile WHERE username = 'testuser@example.com';```
  -  Verify SQLite database file already exists inside your running kittycash container
  ```
   sudo docker exec -it kittycash ls -lh /kittyCash/instance/kittycash.sqlite
  ```
  -  Connect to SQLite inside the container and check the tables:
  ```
  sudo docker exec -it kittycash sqlite3 /kittyCash/instance/kittycash.sqlite

  ```

  
- **Result:**


## **39.SQlite Insertions**
- **Command:**
- Insert into kittycash_user Table 
  ```
      INSERT INTO kittycash_user (
        username, password, address, firstname, lastname, temppassword, 
        authenticated, "authToken", "refreshToken"
    ) VALUES (
        'testuser@example.com', 
        'testpassword',  -- ⚠️ Store a hashed password in a real application!
        '1234 Test Street, Test City', 
        'John', 
        'Doe', 
        'temp123', 
        1, 
        'sample_auth_token', 
        'sample_refresh_token'
    );
  ```
 Then Run
 ```
   from werkzeug.security import generate_password_hash

hashed_password = generate_password_hash("testpassword")
print(hashed_password)  # Copy this output and insert it manually

 ```
Then insert in db 

```
UPDATE kittycash_user 
SET password = 'PASTE_HASHED_PASSWORD_HERE' 
WHERE username = 'testuser@example.com';

```

- Insert into kittycash_profile_key record
```
INSERT INTO kittycash_profile_key (pid, publickey, privatekey, encryptionKey) 
VALUES ('test_pid_123', 'sample_public_key', 'sample_private_key', 'sample_encryption_key');

```
- Verify Insertion
```
SELECT * FROM kittycash_profile_key WHERE pid = 'test_pid_123';

```


- **Result:**


## **40.**
- **Command:**  
  ``````
- **Result:**


## **41.**
- **Command:**  
  ``````
- **Result:**




## **42.**
- **Command:**  
  ``````
- **Result:**


## **43.**
- **Command:**  
  ``````
- **Result:**


## **44.**
- **Command:**  
  ``````
- **Result:**


## **45.**
- **Command:**  
  ``````
- **Result:**


## **46.**
- **Command:**  
  ``````
- **Result:**




## **47.**
- **Command:**  
  ``````
- **Result:**


## **48.**
- **Command:**  
  ``````
- **Result:**


## **49.**
- **Command:**  
  ``````
- **Result:**


## **50.**
- **Command:**  
  ``````
- **Result:**


## **51.**
- **Command:**  
  ``````
- **Result:**


