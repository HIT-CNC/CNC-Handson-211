#Ingress

Deploying Ingress Controller to Kubernetes Cluster

   $ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.0/deploy/static/provider/cloud/deploy.yaml
   
 This will create
 
 1. `ingress-nginx` namespace
 2. Ingress Controller Pods
 3. A service that serve the requests to Ingress Controller and ingress resources
 
 Please note the IP address for Ingress Service for adding to DNS Server
 
$ kubectl get services -n ingress-nginx

Incase of on-prem deployment Ingress service can be exposed using NodePort or by pointing LoadBalancer

### Simple Ingress Example

Run a application Pod using `sample-application-dotnet.yaml`, this will create an application pod and service

    $ kubectl create ns dev
	$ kubectl apply -f sample-application-dotnet.yaml
    
	
	

## Inspect

   $ kubectl get pods -n dev
   $ kubectl get services -n dev
   

## Apply Ingress
Refer `ingress-dotnet.yaml`

1. Edit and change the hostname of the ingress to your desired hostname
2. Point the hostname to the loadbalancer IP with respective to DNS
3. Verify the routes

    $ kubectl apply -f ingress-dotnet.yaml
   
### Output

The application will be served at http://<hostname>:80

## Ingress Types
The above Ingress only points to one application Service. For serving multiple application. Ingress can be used with
1. Name based routing
   Multiple hostname can share the same ingress controller and point the LB ip at respective DNS and Ingress Controller will resolve based on hostname 
2. Path based routing
   Pointing to multiple application based on Path and regex in path

## Path based Example
Run a application Pod using `sample-application-flask.yaml`, this will create an application pod and service

    
	$ kubectl apply -f sample-application-flask.yaml
    ...

## Inspect

    $ kubectl get pods -n dev
    $ kubectl get services -n dev


## Apply Ingress
Refer `ingress-flask.yaml`

1. Edit and change the hostname of the ingress to your desired hostname
2. Point the hostname to the loadbalancer IP with respective to DNS
3. Verify the routes

    $ kubectl apply -f ingress-flask.yaml
-----
### Output

The application will be served at http://<hostname>:80/demo


### Cleanup

```sh
kubectl delete ns dev ingress-nginx
```
