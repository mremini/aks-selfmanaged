# AKS Self Managed Cluster with ASK CNI and Calico Policy

1. Download aks-engine and move the binary to your path

```
wget https://github.com/Azure/aks-engine/releases/download/v0.64.0/aks-engine-v0.64.0-linux-amd64.zip
unzip aks-engine-v0.64.0-linux-amd64.zip
cp aks-engine-v0.64.0-linux-amd64/aks-engine /home/mremini/mybin/

```

2. Customize the aks-engine deployment template. See 

3. Create the AKS cluster using the command below,

```
aks-engine deploy --resource-group cloudteam_mremini_spokes --location eastus --api-model aks-spoke1-calico-azure.json --force-overwrite
```

You should have an output like the below

```
aks-engine deploy --resource-group cloudteam_mremini_spokes --location eastus --api-model aks-spoke1-calico-azure.json --force-overwrite
INFO[0000] No subscription provided, using selected subscription from azure CLI: cc0d730a-9395-4c66-8a74-efdfc1b05670 
WARN[0006] Running only 1 control plane VM not recommended for production clusters, use 3 or 5 for control plane redundancy 
INFO[0015] Starting ARM Deployment cloudteam_mremini_spokes-43956503 in resource group cloudteam_mremini_spokes. This will take some time... 
INFO[0465] Finished ARM Deployment (cloudteam_mremini_spokes-43956503). Succeeded 

```

The aks-engine command will create an ARM template (see azuredeploy.json) and deploy it. 

4. Use the generated  KUBECONFIG file to connect to the AKS cluster

```
cp  aks-selfmanaged/_output/mrespoke1/kubeconfig/kubeconfig.eastus.json  ~/.kube/azure-kubeconfig.eastus.json
export KUBECONFIG=~/.kube/azure-kubeconfig.eastus.json

kubectl get nodes -o wide
NAME                       STATUS   ROLES    AGE   VERSION    INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
k8s-master-17306082-0      Ready    master   12h   v1.18.19   10.61.4.10    <none>        Ubuntu 18.04.5 LTS   5.4.0-1051-azure   docker://19.3.14
k8s-nodepool1-17306082-0   Ready    agent    12h   v1.18.19   10.61.4.35    <none>        Ubuntu 18.04.5 LTS   5.4.0-1051-azure   docker://19.3.14
k8s-nodepool1-17306082-1   Ready    agent    12h   v1.18.19   10.61.4.66    <none>        Ubuntu 18.04.5 LTS   5.4.0-1051-azure   docker://19.3.14
k8s-nodepool2-17306082-0   Ready    agent    12h   v1.18.19   10.61.6.4     <none>        Ubuntu 18.04.5 LTS   5.4.0-1051-azure   docker://19.3.14
k8s-nodepool3-17306082-0   Ready    agent    12h   v1.18.19   10.61.8.4     <none>        Ubuntu 18.04.5 LTS   5.4.0-1051-azure   docker://19.3.14
```

5. Test the AKS cluster and create a deployment

```
kubectl create namespace webecho
kubectl create deployment webecho --image=k8s.gcr.io/echoserver:1.10 -r 2 --namespace webecho

kubectl get pods -n webecho
NAME                       READY   STATUS    RESTARTS   AGE
webecho-74dc6865cc-lccc7   1/1     Running   0          15s
webecho-74dc6865cc-n2j2x   1/1     Running   0          15s

kubectl describe pods webecho-74dc6865cc-lccc7 --namespace webecho
Name:         webecho-74dc6865cc-lccc7
Namespace:    webecho
Priority:     0
Node:         k8s-nodepool1-17306082-1/10.61.4.66
Start Time:   Thu, 24 Jun 2021 14:46:42 +0800
Labels:       app=webecho
              pod-template-hash=74dc6865cc
Annotations:  kubernetes.io/psp: privileged
Status:       Running
IP:           10.61.4.79  <--------------------------- PoD IP from the VNET
IPs:
  IP:           10.61.4.79
Controlled By:  ReplicaSet/webecho-74dc6865cc

```

6. try to access the pod from a VM within the VNET




Ref:
 * https://github.com/Azure/aks-engine/blob/master/docs/tutorials/quickstart.md#install-aks-engine
 * https://github.com/Azure/aks-engine/tree/master/examples/
 * https://github.com/Azure/aks-engine/tree/master/examples/networkpolicy
 * https://docs.projectcalico.org/getting-started/kubernetes/self-managed-public-cloud azure#aks-engine-for-azure-networking-and-calico-network-policy