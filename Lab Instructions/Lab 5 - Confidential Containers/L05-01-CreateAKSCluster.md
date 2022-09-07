L05-01

# Deploy an AKS cluster with confidential nodes

In this lab, you'll deploy an Azure Kubernetes Service (AKS) cluster with enclave-aware (DCsv2/DCSv3) VM nodes. You'll then run a simple Hello World application in an enclave. 

For more info see [Confidential computing nodes on Azure Kubernetes Service](https://docs.microsoft.com/en-us/azure/confidential-computing/confidential-nodes-aks-overview).

## Create an AKS cluster with a system node pool

Let's create an AKS cluster with the confidential computing add-on enabled:

```powershell
# Make sure the name is unique
$aksClusterName = "aksCelAccHol"
az aks create `
   -g $rgName `
   --name $aksClusterName `
   --generate-ssh-keys `
   --enable-addons confcom
```

The above command will deploy a new AKS cluster with system node pool of non confidential computing node. Confidential computing Intel SGX nodes are not recommended for system node pools.

## Add a user node pool with confidential computing capabilities to the AKS cluster

Run the following command to add a user node pool of Standard_DC4s_v3 size (or you can choose another size from the [list of supported DCsv2/DCsv3 SKUs and regions](https://docs.microsoft.com/en-us/azure/virtual-machines/dcv3-series)) with three nodes to the AKS cluster.

```powershell
$aksConfPoolName = "aksconfpool1"
az aks nodepool add `
   --cluster-name $aksClusterName `
   --name $aksConfPoolName `
   --resource-group $rgName `
   --node-vm-size Standard_DC4s_v3 `
   --node-count 3
```

After you run the command, a new node pool with DCsv3 should be visible with confidential computing add-on DaemonSets.

## Verify the node pool and add-on

Get the credentials for your AKS cluster:

```powershell
az aks get-credentials `
   --resource-group $rgName `
   --name $aksClusterName
```

Use the kubectl get pods command to verify that the nodes are created properly and the SGX-related DaemonSets are running on DCsv2 node pools:

```powershell
kubectl get pods --all-namespaces
```

If the output includes 3 rows named "sgx-plugin-*****" in kube-system (one row per node in the pool) with status "Running", then your AKS cluster is ready to run confidential applications.