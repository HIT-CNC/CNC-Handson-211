

# KUBERNETES SERVICES

In this lab, we will configure Kubernetes services of type ClusterIP, NodePort and LoadBalancer

- We will start by creating a new deployment (myapp-deployment) which has 3 Nginx pods
```
$ kubectl apply -f CNC-Handson-211/kubernetes_networking/kubernetes_services/myapp-deployment.yaml
```
- To see the created deployment, execute the following command and make sure 3 replicas of the deployments is READY and AVAILABLE
```
$ kubectl get deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
myapp-deployment   3/3     3            3           12s
```

## Service type ClusterIP

- Create a Service object for the “myapp-deployment” we created in previous steps.
```
$ kubectl apply -f CNC-Handson-211/kubernetes_networking/kubernetes_services/myapp-svc.yaml
```
- Now, please check the status of the Service (myapp-svc) we created
```
$ kubectl get svc
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.136.0.1     <none>        443/TCP   91m
myapp-svc    ClusterIP   10.136.10.76   <none>        80/TCP    9s
```
- Describe the Service to see whether the endpoints (Pods) are added to the service or not.
```
$ kubectl describe svc/myapp-svc
Name:              myapp-svc
Namespace:         default
Labels:            <none>
Annotations:       cloud.google.com/neg: {"ingress":true}
Selector:          app=nginx
Type:              ClusterIP
IP:                10.136.10.76
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.132.0.19:80,10.132.0.20:80,10.132.2.8:80
Session Affinity:  None
Events:            <none> 
```
Please note that there are three endpoint IP addresses, which are the IP addresses of the "myapp-deployment" pods. And **10.136.10.76** is the Service ClusterIP. Any traffic hitting the port 80 of ClusterIP will be load balanced across the backend endpoints in a round-robin fashion.

**Please note that the ClusterIP is reachable only within the cluster. To expose the application to external world, you should either use Service type NodePort or LoadBalancer.**

## Service type NodePort

Next, we will configure Service type NodePort for our deployment (myapp-deployment). NodePort is used to expose the application to external world using a port in the Kubernetes worker nodes. 

**NodePort is assigned from a pool of cluster-configured NodePort ranges which typically lies in the range of 30000–32767.**
- Modify the service (myapp-svc) to use NodePort and check the status of the service again.
```
$ kubectl apply -f CNC-Handson-211/kubernetes_networking/kubernetes_services/myapp-svc-nodeport.yaml

kubectl get svc
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.136.0.1     <none>        443/TCP        104m
myapp-svc    NodePort    10.136.10.76   <none>        80:30799/TCP   12m
```
Please note the NodePort assigned for the service is 30799.

- Access the nginx application using the browser or curl on the “NodePort” and see whether the Nginx page is getting displayed.

## Service type LoadBalancer

Next, we will configure the service type LoadBalancer for myapp-deployment.

- Execute the following command to apply Service type LoadBalancer. 
```
$  kubectl apply -f CNC-Handson-211/kubernetes_networking/kubernetes_services/myapp-svc-load-balancer.yaml
service/myapp-svc configured
```
Now, a new Layer4 load balancer would be created automatically using your respective cloud provider API. Please wait for some time and you would be able to see the IP address of the LB.

```
$ kubectl get svc
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)        AGE
kubernetes   ClusterIP      10.136.0.1     <none>           443/TCP        114m
myapp-svc    LoadBalancer   10.136.10.76   35.198.247.165   80:30799/TCP   23m
```
In this case, the IP address is **35.198.247.165**. Now try to access Nginx application via browser or CURL using the URL `http://35.198.247.165:80`

Please note the IP address will be different in your case.

