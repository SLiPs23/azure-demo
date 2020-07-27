# Install Kubernetes Cluster in Azure

To install the K8s cluster I will be using Terraform. You can use brew to install Terraform and the Azure CLI

`brew install terraform`

`brew install azure-cli`

Before we initialize Terraform we want to create a storage container in Azure to store our state files. I used the Azure console to create a storage account name of cstfstorage. After creating the storage account I used the console to grab the account-key. Now we are ready to create the storage container.

`az storage container create -n tfstate --account-name cstfstorage --account-key $account-key`

Now we are ready to initialize Terraform. Go into the tf-deploy directory and run the following:

`terraform init -backend-config="storage_account_name=cstfstorage" -backend-config="container_name=tfstate" -backend-config="access_key=$account-key" -backend-config="key=k8s.demo.tfstate"`

The $account-key is the same key from the command above which I grabbed from the Azure console. Now that Terraform is initialized we want to create a service principal to install our cluster:

`az ad sp create-for-rbac --name ServicePrincipalK8s`

Next we are going to set a couple of Terraform variables and you can get these by looking at the output of the previous command

`export TF_VAR_client_id=$appId`

`export TF_VAR_client_secret=$password`

Now it is time to create the cluster, first we run Terraform plan and save it to and out file:

`terraform plan -out out.plan`

Assuming there were no errors we then run the Apply:

`terraform apply out.plan`

Now we wait for the cluster to be created. I used the Azure console to watch and see when the cluster was up and running. To gain access to the clusters we can run a couple of commands to get the KUBECONFIG setup:

`echo "$(terraform output kube_config)" > ./azurek8s`

` export KUBECONFIG=./azurek8s`

`kubectl get nodes`

That's it, we now have a running Kubernetes cluster in Azure. Head over to the kubernetes directory for deploying Prometheus, Grafana, Nginx-Ingress, and Nginx.