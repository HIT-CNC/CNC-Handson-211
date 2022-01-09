
# Docker Networking

## Host Networking
If you use the `host` network mode for a container, that container’s network stack is not isolated from the Docker host (the container shares the host’s networking namespace), and the container does not get its own IP-address allocated. For instance, if you run a container which binds to port 80 and you use `host` networking, the container’s application is available on port 80 on the host’s IP address.

The goal of this tutorial is to start a `nginx` container which binds directly to port 80 on the Docker host. From a networking point of view, this is the same level of isolation as if the `nginx` process were running directly on the Docker host and not in a container. 
1.  Create and start the container as a detached process. The  `--rm`  option means to remove the container once it exits/stops. The  `-d`  flag means to start the container detached (in the background).
    
    ```
    $ sudo docker run --rm -d --network host --name my_nginx nginx   
    ```
2.  Access Nginx by using curl
```
curl http://localhost
 ```
3.  Examine your network stack using the following commands:
    
    -   Examine all network interfaces and verify that a new one was not created.
        
        ```
        $ ip addr show
     ```
        
    -   Verify which process is bound to port 80, using the  `netstat`  command. You need to use  `sudo`  because the process is owned by the Docker daemon user and you otherwise won’t be able to see its name or PID.
        
        ```
        $ sudo netstat -tulpn | grep :80
        ```
        
5.  Stop the container. It will be removed automatically as it was started using the  `--rm`  option.
    
    ```
    docker container stop my_nginx
    ```

