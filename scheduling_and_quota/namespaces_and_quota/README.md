# RESOURCE QUOTAS

In this section, we go through a demo example of how to create and define the CPU resource quota on a namespace with requests and limits.
- Create a namespace to test Kubernetes resource quotas, quota-demo:

```
$ kubectl create namespace quota-demo
namespace/quota-demo created
```

- Then, create the CPU quota on a namespace:

``` 
$ kubectl apply -f cpu-quota.yaml
resourcequota/test-cpu-quota created
```

- You can verify the  _ResourceQuota_  object has been created. Note: the “used” column, as initially no quota is used in the configuration and the namespace is populated as empty.

```
$ kubectl describe resourcequota/test-cpu-quota --namespace quota-demo
Name:         test-cpu-quota
Namespace:    quota-demo
Resource      Used  Hard
--------      ----  ----
limits.cpu    0     300m
requests.cpu  0     200m
```

- Thereafter, you can create a test pod with requests and limits defined as shown below:

```
$ kubectl create -n quota-demo -f- <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: testpod1
spec:
  containers:
  - name: quota-test
    image: busybox
    imagePullPolicy: IfNotPresent
    command: ['sh', '-c', 'echo Pod is Running ; sleep 5000']
    resources:
      requests:
        cpu: "100m"
      limits:
        cpu: "200m"
  restartPolicy: Never
EOF
pod/testpod1 created
```

See the updated settings on the namespace and note the data displayed in the “used” column. You will now notice a difference, with the new pod just created having used some of the quota:

```
$ kubectl describe resourcequota/test-cpu-quota --namespace quota-demo
Name:         test-cpu-quota
Namespace:    quota-demo
Resource      Used  Hard
--------      ----  ----
limits.cpu    200m  300m
requests.cpu  100m  200m
```

To continue the process, create another pod and observe the remaining quota values:

```
$ kubectl create -n quota-demo -f- <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: testpod2
spec:
  containers:
  - name: quota-test
    image: busybox
    imagePullPolicy: IfNotPresent
    command: ['sh', '-c', 'echo Pod is Running ; sleep 5000']
    resources:
      requests:
        cpu: "10m"
      limits:
        cpu: "20m"
  restartPolicy: Never
EOF

pod/testpod2 created

$ kubectl describe resourcequota/test-cpu-quota --namespace quota-demo
Name:         test-cpu-quota
Namespace:    quota-demo
Resource      Used  Hard
--------      ----  ----
limits.cpu    220m  300m
requests.cpu  110m  200m
```

As noted above, the used column is now updated again. The new pod is now listed as having consumed more of the available quota limits for the total allocated CPU resources.

If we try to create a pod requesting more resources than what’s available in quota, we will now receive an error message that states we don’t have enough quota left to create the new pod:

```
C02W84XMHTD5:terraform-dev iahmad$ kubectl create -n quota-demo -f- <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: testpod3
spec:
  containers:
  - name: quota-test
    image: busybox
    imagePullPolicy: IfNotPresent
    command: ['sh', '-c', 'echo Pod is Running ; sleep 5000']
    resources:
      requests:
        cpu: "100m"
      limits:
        cpu: "200m"
  restartPolicy: Never
EOF


Error from server (Forbidden): error when creating "STDIN": pods "testpod3" is forbidden: exceeded quota: test-cpu-quota, requested: limits.cpu=200m,requests.cpu=100m, used: limits.cpu=220m,requests.cpu=110m, limited: limits.cpu=300m,requests.cpu=200m

```

Finally, do the clean-up of the installation and configuration files by deleting the namespace and resources contained in it. You can use the command below with your configuration variables:

```
$ kubectl delete ns quota-demo --cascade
namespace "quota-demo" deleted
```
