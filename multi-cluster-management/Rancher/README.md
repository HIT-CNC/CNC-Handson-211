# MANAGE CLUSTER USING RANCHER

In this lab, you will have access to an already provisioned RANCHER UI. You will manage Kubernetes clusters from the RANCHER UI.

 - Login to RANCHER UI using the following URL
 ```
 https://54.255.195.171/dashboard
 ```
 It is using a self-signed certificate, so you may have to by-pass the browser warnings.

- Once logged in, you will be able to see a cluster which is already registered to your Rancher server.

Perform the following actions from the Rancher UI.

- Create a new namespace. You can select any random name
- Connect to kubectl shell
- Deploy a new Nginx pod. You can either upload the YAML file or copy the contents of YAML file to RANCHER UI. You can use nginx.yaml file under this repo.
- Monitor the status of all the nodes in your cluster.
