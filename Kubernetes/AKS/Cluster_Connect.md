Connect to your cluster using command line tooling to interact directly with cluster using kubectl, the command line tool for Kubernetes. Kubectl is available within the Azure Cloud Shell by default and can also be installed locally.

Set cluster context

Login to your azure account
az login

Set the cluster subscription
az account set --subscription 52f167e5-dfca-4d77-a744-e6c7bc1a3235

Download cluster credentials
az aks get-credentials --resource-group rg-aks-test --name aks_test --overwrite-existing

=======================================================================================

az aks get-credentials --resource-group rg-aks-test --name aks_test --overwrite-existing

PS C:\Users\DELL> az aks get-credentials --resource-group rg-aks-test --name aks_test --overwrite-existing
az : WARNING: Merged "aks_test" as current context in C:\Users\DELL\.kube\config
At line:1 char:1
+ az aks get-credentials --resource-group rg-aks-test --name aks_test - ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (WARNING: Merged...LL\.kube\config:String) [], RemoteException
    + FullyQualifiedErrorId : NativeCommandError
 
=======================================================================================

WARNING: Merged "aks_test" as current context in C:\Users\DELL\.kube\config

• AKS credentials were successfully fetched
• The kubeconfig file was updated
• Context "aks_test" is now your current kubectl context
• --overwrite-existing replaced any existing entry

There is no failure here.

=======================================================================================

test-tzcf2ij4.hcp.centralindia.azmk8s.io : This is dns resolution for the Public IP Of the API Server With this we will be able to connect with Cluster

C:\Users\DELL>kubectl get pods
Unable to connect to the server: dial tcp: lookup test-tzcf2ij4.hcp.centralindia.azmk8s.io: no such host