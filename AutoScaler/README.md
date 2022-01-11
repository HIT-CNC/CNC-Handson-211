### Pod AutoScaler

This lab will help you to scale application and set auto-scale rules
   

### Deploy sample application

Check the `sample-application-flask.yaml` application

Under Deployment spec you should be able to see replicas

spec:
  replicas: 1


Run a application Pod using `sample-application-flask.yaml`, this will create an application  with 1 pod and service

    $ kubectl create ns dev
	$ kubectl apply -f sample-application-dotnet.yaml
	
## Inspect

   $ kubectl get pods -n dev
   $ kubectl get services -n dev
  
 Modify the replicas 2 and inspect again.

## Apply AutoScaler rules


`$ kubectl autoscale deployment flask-blog -n dev --cpu-percent=20 --min=3 --max=10`

`$kubectl get hpa`

## Increase the load 

`$ kubectl run -i --tty load-generator -n dev --rm --image=busybox --restart=Never -- /bin/sh -c "while sleep 0.000000001; do wget -q -O- http://flask-blog:8000; done"`

## Inspect

   $ kubectl get pods -n dev
   
You can see the pods created

### Cleanup

```sh
kubectl delete ns dev
```
