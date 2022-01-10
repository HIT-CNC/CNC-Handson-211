
# NODE SELECTORS

NodeSelector is used for assigning a pod to a specific node based on the labels attached to the nodes. For eg: scheduling an AI/ML workload on a node with GPU available.

- Check the labels attached to a node.
```
$ kubectl get nodes --show-labels
```
- Create  a new label (environment=prod) for any one of the nodes from the list. 
```
$ kubectl label nodes ip-10-0-153-191.ap-southeast-1.compute.internal environment=prod
node/ip-10-0-153-191.ap-southeast-1.compute.internal labeled
```
Please note the hostname of your node might differ based on the cluster you use. The command takes the following format:
```
$ kubectl label nodes <node_name> <label_key>=<label_value>
```
- Test whether the label is added by describing the respective node
```
$ kubectl describe node ip-10-0-153-191.ap-southeast-1.compute.internal
Name:               ip-10-0-153-191.ap-southeast-1.compute.internal
Roles:              <none>
Labels:             alpha.eksctl.io/cluster-name=201-cluster-1
                    alpha.eksctl.io/nodegroup-name=managed-ng-private
                    beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/instance-type=t3.large
                    beta.kubernetes.io/os=linux
                    eks.amazonaws.com/capacityType=SPOT
                    eks.amazonaws.com/nodegroup=managed-ng-private
                    eks.amazonaws.com/nodegroup-image=ami-04c33e5aa8fd573a3
                    eks.amazonaws.com/sourceLaunchTemplateId=lt-03cdb73d162b8e7c0
                    eks.amazonaws.com/sourceLaunchTemplateVersion=1
                    **environment=prod**
                    failure-domain.beta.kubernetes.io/region=ap-southeast-1
                    failure-domain.beta.kubernetes.io/zone=ap-southeast-1a
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=ip-10-0-153-191.ap-southeast-1.compute.internal
                    kubernetes.io/os=linux
                    node.kubernetes.io/instance-type=t3.large
                    role=worker
                    topology.kubernetes.io/region=ap-southeast-1
                    topology.kubernetes.io/zone=ap-southeast-1a
 ```
 - In order to assign a Pod to the node with the label we just added, you need to specify a nodeSelector field in the PodSpec. Please check and apply the following YAML manifest
 
 ```
 $ kubectl apply -f nodeselector.yaml
pod/httpd created
```

- Once the pod is created, please check in which node the pod got scheduled.

```
 $ kubectl describe pods/httpd
Name:         httpd
Namespace:    default
Priority:     0
Node:         ip-10-0-153-191.ap-southeast-1.compute.internal/10.0.153.191
 ```
 
 You can see the pod got scheduled as per the "nodeSelector" configuration we defined in the YAML manifest file.

- Now, go ahead and delete the pod. Modify the YAML file to use another random nodeSelector and create the pod again. This time do not attach the a label to the node. Observe the status whether the pod is getting scheduled or not. You can use "kubectl describe" command to check the pod status.
