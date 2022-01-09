# Docker Networking

## BRIDGE NETWORKING
A bridge network uses a software bridge which allows containers connected to the same bridge network to communicate, while providing isolation from containers which are not connected to that bridge network. The Docker bridge driver automatically installs rules in the host machine so that containers on different bridge networks cannot communicate directly with each other.

There are 2 types of bridges

 - Default Bridge
 - User Defined bridge

## Use default network bridge

In this example, you start two different `busybox` containers on the same Docker host and do some tests to understand how they communicate with each other.

 - Docker by default will create some networks. First of all let us have a look at the default networks
 
 ```sh
 $ docker network ls
 NETWORK ID     NAME      DRIVER    SCOPE
96b44b007d62   bridge    bridge    local
546b2ac8dbd1   host      host      local
cda4856868b9   none      null      local
 ```
The default `bridge` network is listed, along with `host` and `none`.

- Start two `busybox` containers running `sh`. The `-dit` flags mean to start the container detached (in the background), interactive (with the ability to type into it), and with a TTY (so you can see the input and output). Since you are starting it detached, you won’t be connected to the container right away. Instead, the container’s ID will be printed. Because you have not specified any `--network` flags, the containers connect to the default `bridge` network.
```sh
$ docker run -dit --name busybox1 busybox /bin/sh
$ docker run -dit --name busybox2 busybox /bin/sh
```
- Check that both containers are actually started
```sh
$ docker ps
CONTAINER ID   IMAGE     COMMAND     CREATED         STATUS         PORTS     NAMES
634127410f8e   busybox   "/bin/sh"   2 minutes ago   Up 2 minutes             busybox2
b6c8bd113884   busybox   "/bin/sh"   3 minutes ago   Up 3 minutes             busybox1
```
- Inspect the `bridge` network to see what containers are connected to it.
```sh
$ docker network inspect bridge
 {
        "Name": "bridge",
        "Id": "96b44b007d626658b0ed5abfc6352d65c5bae6da42dd90213de6117aabd6a096",
        "Created": "2022-01-09T02:32:57.626749292Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "634127410f8ef8e5a73b00c12248d81153078f2bf6670844587696585e7ac054": {
                "Name": "busybox2",
                "EndpointID": "dd615bdcb5046ba1cfaa2c210db98cfccb7bec18cbe9d652002342a85fc893d7",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            },
            "b6c8bd1138849bf5cdc992c7b8b90d534af8d2c79134ce10070dc135e16e0796": {
                "Name": "busybox1",
                "EndpointID": "ece3feb712b072c4f43008c8bf288def94bee64df25c265016f5df47084a9ab2",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
```
Near the top, information about the `bridge` network is listed, including the IP address of the gateway between the Docker host and the `bridge` network (`172.17.0.1`). Under the `Containers` key, each connected container is listed, along with information about its IP address (`172.17.0.2` for `busybox1` and `172.17.0.3` for `busybox2`).

- The containers are running in the background. Use the `docker attach` command to connect to `busybox1`
```sh
$ docker attach busybox1
/ # 
```
- The prompt changes to `#` to indicate that you are the `root` user within the container. Use the `ip addr show` command to show the network interfaces for `busybox1` as they look from within the container:
```sh
/ # ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
6: eth0@if7: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
/ # 
```
The first interface is the loopback device. Ignore it for now. Notice that the second interface has the IP address `172.17.0.2`, which is the same address shown for `busybox1` in the previous step.
- From within `busybox1`, make sure you can connect to the internet by pinging `google.com`. The `-c 2` flag limits the command to two `ping` attempts.
```sh
/ # ping -c 2 google.com
PING google.com (142.251.10.139): 56 data bytes
64 bytes from 142.251.10.139: seq=0 ttl=103 time=1.665 ms
64 bytes from 142.251.10.139: seq=1 ttl=103 time=1.730 ms

--- google.com ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 1.665/1.697/1.730 ms
/ # 
```
- Now try to ping the second container. First, ping it by its IP address, `172.17.0.3`:

```sh
/ # ping -c 2 172.17.0.3
PING 172.17.0.3 (172.17.0.3): 56 data bytes
64 bytes from 172.17.0.3: seq=0 ttl=64 time=0.099 ms
64 bytes from 172.17.0.3: seq=1 ttl=64 time=0.075 ms

--- 172.17.0.3 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.075/0.087/0.099 ms
/ # 
```
- This succeeds. Next, try pinging the `busybox2` container by container name. This will fail.
```sh
/ # ping -c busybox2
ping: invalid number 'busybox2'
/ # 
```
- Detach from `busybox1` without stopping it by using the detach sequence, `CTRL` + `p`  `CTRL` + `q` (hold down `CTRL` and type `p` followed by `q`).
- Delete the containers using the following command
```sh
$ docker stop busybox1 busybox2
$ docker rm busybox1 busybox2
```

## Use user-defined bridge networks

In this example, we will create 4 containers. We will attach both busybox1 and busybox2 containers to a user-defined network called  `busybox-net`  which we will create. These containers are not connected to the default  `bridge`  network at all. We then start a third  `busybox`  container which is connected to the  `bridge`  network but not connected to  `busybox-net`, and a fourth  `busybox`  container which is connected to both networks.

 - Create the  `busybox-net`  network. You do not need the  `--driver bridge`  flag since it’s the default, but this example shows how to specify it.
```sh
$ docker network create --driver bridge busybox-net
```
 - List Docker networks
```sh
$ docker network ls
NETWORK ID     NAME          DRIVER    SCOPE
96b44b007d62   bridge        bridge    local
33eaee833ddc   busybox-net   bridge    local
546b2ac8dbd1   host          host      local
cda4856868b9   none          null      local
```
 - Inspect the `busybox-net` network. This shows you its IP address and the fact that no containers are connected to it:
```sh
$ docker network inspect busybox-net
[
    {
        "Name": "busybox-net",
        "Id": "33eaee833ddca1f892165bf5d6795544163940ad22e54e28b6b007a31cd7ddb3",
        "Created": "2022-01-09T03:34:46.898217071Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```
Notice that this network’s gateway is `172.18.0.1`, as opposed to the default bridge network, whose gateway is `172.17.0.1`
 - Create your four containers. Notice the `--network` flags. You can only connect to one network during the `docker run` command, so you need to use `docker network connect` afterward to connect `busybox4` to the `bridge` network as well.
 
 ```sh
 $ docker run -dit --name busybox1 --network busybox-net busybox /bin/sh

$ docker run -dit --name busybox2 --network busybox-net busybox /bin/sh

$ docker run -dit --name busybox3 busybox /bin/sh

$ docker run -dit --name busybox4 --network busybox-net busybox /bin/sh

$ docker network connect bridge busybox4
 ```

- Verify that all containers are running:

```
$ docker ps
CONTAINER ID   IMAGE     COMMAND     CREATED          STATUS          PORTS     NAMES
02d3f6d8cd5f   busybox   "/bin/sh"   18 seconds ago   Up 16 seconds             busybox4
eb3ef9d695a6   busybox   "/bin/sh"   25 seconds ago   Up 24 seconds             busybox3
f3f250d055dd   busybox   "/bin/sh"   33 seconds ago   Up 32 seconds             busybox2
2e4becfff4b0   busybox   "/bin/sh"   45 seconds ago   Up 43 seconds             busybox1

```
- Inspect the `bridge` network and the `busybox-net` network again:

```sh
$ docker network inspect bridge
 {
        "Name": "bridge",
        "Id": "96b44b007d626658b0ed5abfc6352d65c5bae6da42dd90213de6117aabd6a096",
        "Created": "2022-01-09T02:32:57.626749292Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "02d3f6d8cd5f3d7bfdadcf45367035cc26a17d4d07439e38516e6e829cefe624": {
                "Name": "busybox4",
                "EndpointID": "a6113e17ffcdc61f483aeb1e9264387fb4a16ef85ae4a1f28ba2f48c6f0f80ac",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            },
            "eb3ef9d695a69bba14d47bc512797b5c8f3f82d0e84d948e9d01d948ebf25b68": {
                "Name": "busybox3",
                "EndpointID": "c5d8a5b2a42aa6bbd906b97392be6cab60193e82fc49a3c79f2df58d673fd119",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {
  ```
  - Containers `busybox3` and `busybox4` are connected to the `bridge` network.     
  
  - Now, inspect busybox-net network
  ```
  $ docker network inspect busybox-net
  ```
  Containers `busybox1`, `busybox2`, and `busybox4` are connected to the `busybox-net` network.
  
- On user-defined networks like `busybox-net`, containers can not only communicate by IP address, but can also resolve a container name to an IP address. This capability is called **automatic service discovery**. Let’s connect to `busybox1` and test this out. `busybox1` should be able to resolve `busybox2` and `busybox4` (and `busybox1`, itself) to IP addresses.
```sh
$ docker container attach busybox1
/ # ping -c2 busybox2
PING busybox2 (172.18.0.3): 56 data bytes
64 bytes from 172.18.0.3: seq=0 ttl=64 time=0.084 ms
64 bytes from 172.18.0.3: seq=1 ttl=64 time=0.078 ms

--- busybox2 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.078/0.081/0.084 ms
/ # ping -c2 busybox4
PING busybox4 (172.18.0.4): 56 data bytes
64 bytes from 172.18.0.4: seq=0 ttl=64 time=0.101 ms
64 bytes from 172.18.0.4: seq=1 ttl=64 time=0.079 ms

--- busybox4 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.079/0.090/0.101 ms
/ # ping -c2 busybox1
PING busybox1 (172.18.0.2): 56 data bytes
64 bytes from 172.18.0.2: seq=0 ttl=64 time=0.046 ms
64 bytes from 172.18.0.2: seq=1 ttl=64 time=0.060 ms

--- busybox1 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.046/0.053/0.060 ms
/ # 
```
- From `busybox1`, you should not be able to connect to `busybox3` at all, since it is not on the `busybox-net` network.
```sh
/ # ping -c2 busybox3
ping: bad address 'busybox3'
/ # 
```
- Not only that, but you can’t connect to `busybox3` from `busybox1` by its IP address either. Look back at the `docker network inspect` output for the `bridge` network and find `busybox3`’s IP address: `172.17.0.2` Try to ping it.

```sh
/ # ping -c2 172.17.0.2
PING 172.17.0.2 (172.17.0.2): 56 data bytes
--- 172.17.0.2 ping statistics ---
2 packets transmitted, 0 packets received, 100% packet loss
```

- Detach from `busybox1` using detach sequence, `CTRL` + `p`  `CTRL` + `q` (hold down `CTRL` and type `p` followed by `q`).

- Remember that busybox4 is connected to both the default bridge network and busybox-net. It should be able to reach all of the other containers. However, you will need to address busybox3 by its IP address. Attach to it and run the tests.
```sh
 $ docker container attach busybox4
/ # ping -c2 busybox1
PING busybox1 (172.18.0.2): 56 data bytes
64 bytes from 172.18.0.2: seq=0 ttl=64 time=0.078 ms
64 bytes from 172.18.0.2: seq=1 ttl=64 time=0.083 ms

--- busybox1 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.078/0.080/0.083 ms

/ # ping -c2 busybox2
PING busybox2 (172.18.0.3): 56 data bytes
64 bytes from 172.18.0.3: seq=0 ttl=64 time=0.098 ms
64 bytes from 172.18.0.3: seq=1 ttl=64 time=0.081 ms

--- busybox2 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.081/0.089/0.098 ms

/ # ping -c2 busybox3
ping: bad address 'busybox3'
/ # ping -c 2 172.17.0.2
PING 172.17.0.2 (172.17.0.2): 56 data bytes
64 bytes from 172.17.0.2: seq=0 ttl=64 time=0.105 ms
64 bytes from 172.17.0.2: seq=1 ttl=64 time=0.077 ms

--- 172.17.0.2 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.077/0.091/0.105 ms
/ # 
/ # ping -c2 busybox4
PING busybox4 (172.18.0.4): 56 data bytes
64 bytes from 172.18.0.4: seq=0 ttl=64 time=0.050 ms
64 bytes from 172.18.0.4: seq=1 ttl=64 time=0.062 ms

--- busybox4 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.050/0.056/0.062 ms
/ # 
```
- Stop and remove all containers and busybox-net
```
$ docker container stop busybox1 busybox2 busybox3 busybox4

$ docker container rm busybox1 busybox2 busybox3 busybox4

$ docker network rm busybox-net
```
