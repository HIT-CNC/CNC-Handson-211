### Pod AutoScaler

This lab will help you to scale application and set auto-scale rules
   

### Deploy sample application

Check the `sample-application-stress.yaml` application

Under Deployment spec you should be able to see replicas

spec:
  replicas: 1


Run a application Pod using `sample-application-stress.yaml`, this will create an application  with 1 pod and service

	$ kubectl apply -f sample-application-stress.yaml
	
## Inspect

   $ kubectl get pod
   $ kubectl get services 
  
 Modify the replicas 2 and inspect again.

## Apply AutoScaler rules


`$ kubectl autoscale deployment php-apache  --cpu-percent=50 --min=3 --max=10`

`$kubectl get hpa`

## Increase the load 

`$ kubectl run -i --tty load-generator -n dev --rm --image=busybox --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"`

## Inspect

   $ kubectl get pods -n dev
   
You can see the pods created

### Cleanup

```sh
kubectl delete -f sample-application-stress.yaml
kubectl delete hpa php-apache
kubectl delete pod load-generator
```
