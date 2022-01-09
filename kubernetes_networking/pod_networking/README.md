
# POD NETWORKING

In this lab, we will create multiple pods and check the network communication between both

- Create  a new pod "nginx1" using the following command
```sh
$ kubectl apply -f CNC-Handson-211/kubernetes_networking/pod_networking/nginx1.yaml
$ kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx1   1/1     Running   0          9s
```
- Create a second pod "nginx2" using the following command
```
$ kubectl apply -f CNC-Handson-211/kubernetes_networking/pod_networking/nginx2.yaml
pod/nginx2 created
$ kubectl get pods
NAME     READY   STATUS    RESTARTS   AGE
nginx1   1/1     Running   0          86s
nginx2   1/1     Running   0          4s
```
- Now, get the IP address of both pods by running the following command
```
$ kubectl get pods -o wide
```
- Connect to nginx1 pod and ping the IP address of nginx2 pod and check the result.
```
$ kubectl exec -it nginx1 -- /bin/bash
```
- Try to ping the IP address of nginx2 pod

```
$ root@nginx1:/# ping -c2 10.132.0.16
PING 10.132.0.16 (10.132.0.16): 56 data bytes
64 bytes from 10.132.0.16: icmp_seq=0 ttl=63 time=0.090 ms
64 bytes from 10.132.0.16: icmp_seq=1 ttl=63 time=0.095 ms
--- 10.132.0.16 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max/stddev = 0.090/0.092/0.095/0.000 ms
```
Please note the IP address will be different in your case. Please refer the output of *kubectl get pods -o wide* command
- Next, try to ping the nginx2 pod using its FQDN. 

```
root@nginx1:/# ping -c2 10-132-0-16.default.pod.cluster.local
PING 10-132-0-16.default.pod.cluster.local (10.132.0.16): 56 data bytes
64 bytes from 10.132.0.16: icmp_seq=0 ttl=63 time=0.154 ms
64 bytes from 10.132.0.16: icmp_seq=1 ttl=63 time=0.079 ms
--- 10-132-0-16.default.pod.cluster.local ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max/stddev = 0.079/0.116/0.154/0.038 ms
```
Please note the FQDN of  a pod will be of the following format
***pod-ip-address.my-namespace.svc.cluster-domain.example*** if it is a standalone pod and ***pod-ip-address.deployment-name.my-namespace.svc.cluster-domain.example*** if the pod is created as part of a Kubernetes deployment.

- Now, please go ahead a delete both pods and then re-create the pods using the same manifest files.
```
$ kubectl delete pods/nginx1
$ kubectl delete pods/nginx2
$ kubectl apply -f CNC-Handson-211/kubernetes_networking/pod_networking/nginx1.yaml
$ kubectl apply -f CNC-Handson-211/kubernetes_networking/pod_networking/nginx2.yaml
```
- Check the IP address of your pods again. It is possible that the IP addresses will be different this time. IP address of a Kubernetes pod is ephemeral. 
