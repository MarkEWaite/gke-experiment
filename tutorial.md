<walkthrough-author name="Jeff Duska" tutorialName="CloudBees Core on GKE">
</walkthrough-author>

# Introduction to CloudBees Core on GKE

## Introduction 

<walkthrough-tutorial-duration duration="60"></walkthrough-tutorial-duration>

In this guide you will learn to:

* Create a Kubernetes cluster
* Install Cloudbees Core
* Troubleshoot basic issues with Kubernetes

At the end of this lab you will have a basic understanding Cloudbees Core on modern cloud plaforms.

Click the **Forward** or **Continue** button to move to the next step.

## Navigate to Kubernetes Engine

First, we need to navigate to the Google Kubernetes Engine in the Google Console. 

Open the [menu][spotlight-console-menu] on the left side of the console.

**Tip:** Links will highlight menus, buttons, and fields in browser.

Then, select the **Kubernetes Engine** section.
<walkthrough-menu-navigation sectionId="KUBERNETES_SECTION"></walkthrough-menu-navigation>

Click the **Continue** button to move to the next step.

## Create a Kubernetes cluster

A cluster consists of at least one cluster master machine and multiple worker
machines called nodes. You deploy CloudBees Core a Kubernetes cluster.

In this step, you will create your Kubernetes cluster. Use a cluster name that you'll remember.

### Creating a cluster

Click the [Create cluster][spotlight-create-cluster] button.

*   Enter your cluster name into the [name][spotlight-instance-name] field.
*   In the [zone][spotlight-instance-zone] field select 
    for this cluster. For example, select one of the `us-east-1` zones.
* In `Number of Nodes`, enter 2.
* In `Machine type` select `2 vCPUs`.
*   Click the [Create][spotlight-submit-create] button to create the cluster.

Google Kubernetes Engine will now create your Kubernetes cluster for your. 

Click the **Continue** button to move to the next step.


## What is Cloud Shell?

While we wait for your cluster to be build, let's briefly go over what Cloud Shell can do.

Cloud Shell is a personal hosted Virtual Machine which comes pre-loaded with developer tools. This interactive shell environment comes with a built-in code editor, persistent disk storage, and web preview functionality.

### Open the Cloud Shell

Open Cloud Shell by clicking
<walkthrough-cloud-shell-icon></walkthrough-cloud-shell-icon>
[icon][spotlight-open-devshell] in the navigation bar at the top of the console.

This tutorial is integrated into the Google Cloud Shell, so it can copy command into your environment.

### Copy a command to the shell.
Try running this command now:

```bash
echo "Hello Cloud Shell"
```
**Tip:** You can click the copy button on the side of the code box, 
it will past the code into the Cloud Shell.

Click the **Continue** button to move to the next step.

## Grant Admin Permissions
During the tutorial, you will be using Kubernetes command line tool, `kubectl`. In this step, you setup your cluster in the kubeconfig. Then, you'll give your account  `cluster-admin` permission.

### Add your cluster to kubeconfig
1. Next cluster at the far right is a `Connect` button. Click it.
1. Click the `Run in Cloud Shell` button.
1. Click `OK` to close the window.

### Assign Cluster Admin Permissions
1. Bind your user account to the `cluster-admin` role 
```bash
kubectl create clusterrolebinding cluster-admin-binding  --clusterrole cluster-admin  --user $(gcloud config get-value account)
```
  **Note** Cluster-admin (full) permission is only needed during installation, services will run using the created roles with limited privileges.

Click the **Continue** button to move to the next step.

## Install Helm (1/3)
[Helm](https://www.helm.sh/) is the package manager for Kubernetes. For this installation of CloudBees Core, you will use Helm to configure [ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) to your cluster.

### Download Helm
1. In Cloud Shell, download and install the Helm binary. Helm has two parts the client command line tool and the server component called Tiller. 
```bash
wget https://storage.googleapis.com/kubernetes-helm/helm-v2.6.2-linux-amd64.tar.gz
```
2. Exact the file
```bash 
tar zxfv helm-v2.6.2-linux-amd64.tar.gz 
```
```bash
cp linux-amd64/helm .
```

Click the **Continue** button to move to the next step.

## Install Helm (2/3)

### Configure Helm Permissions

1. Ensure your account has `cluster-admin` role in your cluster.
```bash
kubectl create clusterrolebinding user-admin-binding --clusterrole=cluster-admin --user=$(gcloud config get-value account)
```
2. Create a service account that Tiller, the server side of Helm, will use for deploying your charts.
```bash
kubectl create serviceaccount tiller --namespace kube-system
```
3. Grant the Tiller service account `cluster-admin` role in your cluster.
```bash
kubectl create clusterrolebinding tiller-admin-binding --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
```

Click the **Continue** button to move to the next step.

## Install Helm (3/3)

### Initialize Helm

1. Initialize Helm to install Tiller in your cluster:
```bash
./helm init --service-account=tiller
```
```bash
./helm update
```
2. Ensure that Helm is properly installed:
```bash
./helm version
```
You see output similar to the following. If Helm is correctly installed, v2.6.2 appears for both client and server.
```
./helm version
Client: &version.Version{SemVer:"v2.6.2", GitCommit:"be3ae4ea91b2960be98c07e8f73754e67e87963c", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.6.2", GitCommit:"be3ae4ea91b2960be98c07e8f73754e67e87963c", GitTreeState:"clean"}
```
It can take a few minutes for Tiler to install.

Click the **Continue** button to move to the next step.

##  Create SSD Storage Class
CloudBees Core requires SSD storage for the `JENKINS_HOME` directory. In this section, we'll create a new ssd storage type.

### Create a new SSD Storage Class
1. Create a ssd-storage.yaml using either the web file editor, nano or you favorite file editor
2. Copy the following contents into the new file (Control-V pastes clipboard content into the cloud shell)
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ssd
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
```
3. Save the file.
4. Create the new storage class in your cluster
```bash 
kubectl create -f ssd-storage.yaml
``` 

Click the **Continue** button to move to the next step.

## Review Namespaces
A Kubernetes cluster will instantiate a default namespace when provisioning the cluster to hold the default set of Pods, Services, and Deployments used by the cluster.

Assuming you have a fresh cluster, you can inspect the available namespaces by doing the following:

### View cluster namespaces 
```bash
kubectl get namespaces
```

You should see something like the following:
```
NAME          STATUS    AGE
default       Active    2h
kube-public   Active    2h
kube-system   Active    2h
```

We recommend dedicating a namespace for CloudBees Core. 

Click the **Continue** button to move to the next step.

## Create CloudBees Jenkins Enterprise Namespace

### Create namespace
1. Create the namespace `cje`
```bash
kubectl create namespace cje
```
1. Attach a label to that namespace
```bash
kubectl label namespace cje name=cje
```
1. Make namespace cje the default namespace for the kubectl context
```bash
kubectl config set-context $(kubectl config current-context) --namespace=cje
```
Click the **Continue** button to move to the next step.

## Install Ingress Controller (1/3)
CloudBees Core does not support the GKE ingress controller at this point but instead, requires the use of the NGINX Ingress Controller. This section walks through the installation using Helm.

### Create the Ingress Controller
```bash
./helm install --namespace ingress-nginx --name nginx-ingress stable/nginx-ingress \
             --set controller.service.externalTrafficPolicy=Local 
```
Click the **Continue** button to move to the next step.

## Install Ingress Controller (2/3)

### Validate the Ingress Controller
Creating the Ingress Controller results in the creation of the corresponding service, along with its corresponding Load Balancer, both of which will take a few moments. You may then execute the following command to retrieve the external IP address to be used for the CloudBees Core cluster domain name. 
```bash
kubectl get services -n ingress-nginx
```

You should see something like the following:
```
NAME                            TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)                      AGE
nginx-ingress-controller        LoadBalancer   10.3.244.187   35.203.153.152   80:30396/TCP,443:31290/TCP   3m
```
**IMPORTANT:** It may take GKE a few minutes to provision the EXTERNAL-IP. You will need it, **before** you move to the next step.

Click the **Continue** button to move to the next step **after the EXTERNAL-IP has been assigned**.

## Install Ingress Controller (2/3)

We need to keep track of the external IP address so we can access our CloudBees Core once our install is done. 

1. Capture the external IP address of our Ingress Controller.
```bash
CJE_IP=$( kubectl -n ingress-nginx get svc nginx-ingress-controller -o jsonpath={".status.loadBalancer.ingress[0].ip}") 
```
```bash
DOMAIN_NAME=jenkins.${CJE_IP}.xip.io
```

Click the **Continue** button to move to the next step.

## Get CloudBees Core Installation files
CloudBees Core installations are configured with YAML files. The CloudBees Core installer provides a cloudbees-core.yml file that is modified for each installation. 

### Download the install files
1. Get the installer files
```bash 
wget https://downloads.cloudbees.com/cloudbees-core/cloud/latest/cloudbees-core_2.138.1.2_kubernetes.tgz
```
2. Unpack the installer 
``` bash
 tar xzvf cloudbees-core_2.138.1.2_kubernetes.tgz
```
Click the **Continue** button to move to the next step.

## Update the install for your domain.
Before we can finish our installation, we need to customize it for your local environment. In this step, we'll update the domain name in the yaml file to point to your cluster.

### Update the domain name
1. Switch to the new `cloudbees-core_2.138.1.2_kubernetes` directory your just created.
```bash
cd cloudbees-core_2.138.1.2_kubernetes/
```
2. Replace the sample domain name of `cje.example.com` with your cluster IP address 
```bash
sed -i s,cje.example.com,$DOMAIN_NAME,g cloudbees-core.yml
```

Click the **Continue** button to move to the next step.

## Disable HTTPS
For this tutorial, we'll diisable SSL.

### Disable SSL 
1. Change all HTTPS references to HTTP
```bash 
sed -i s,https://$DOMAIN_NAME,http://$DOMAIN_NAME,g cloudbees-core.yml
```
### Disable SSL redirects
1. Open `cloudbees-core.yml` in an editor (vi, nano, etc.)

<!--
<walkthrough-editor-open-file filePath="cloudbees-core_2.138.1.2_kubernetes/cloudbees-core.yml">
</walkthrough-editor-open-file>
-->

2. Search for `ssl-redirect` 
<!--
<walkthrough-editor-select-regex filePath="cloudbees-core_2.138.1.2_kubernetes/cloudbees-core.yml" regex="ssl-redirect">
</walkthrough-editor-select-regex>
-->

3. Change the value from `"true" to "false"
4. Save your changes.

Click the **Continue** button to move to the next step.

## Use SSD persistent disks
Finally, you'll use the SSD storage class you created earlier. 

To use the 'ssd' storage class for Operations Center, you will need to uncomment and set the storageClassName definition under 'volumeClaimTemplates' to 'ssd' in the cloudbees-core.yml file prior to installation.

```
 volumeClaimTemplates:
  - metadata:
      name: jenkins-home
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 20Gi
      storageClassName: ssd
```

You can use the file editor to edit the cloudbees-core.yml file.

To configure Managed Masters to use SSD disks by default, update the storage class in the cloudbees-core.yml file. Search for the commented-out section

```
# To allocate masters using a non-default storage class, add the following
# -Dcom.cloudbees.masterprovisioning.kubernetes.KubernetesMasterProvisioning.storageClassName=some-storage-class
```
Change it so that the storage class is now `ssd`.
```
-Dcom.cloudbees.masterprovisioning.kubernetes.KubernetesMasterProvisioning.storageClassName=ssd
```

Click the **Continue** button to move to the next step.

## Run the installer
You can now install CloudBees Core. 

### Install CloudBee Core on your cluster
1. Run the `kubectl` command to deploy CloudBees Core 
```bash 
kubectl apply -f cloudbees-core.yml
```
You will see the following output 
```
serviceaccount "cjoc" created
role "master-management" created
rolebinding "cjoc" created
configmap "cjoc-config" created
configmap "cjoc-configure-jenkins-groovy" created
statefulset "cjoc" created
service "cjoc" created
ingress "cjoc" created
ingress "default" created
serviceaccount "jenkins" created
role "pods-all" created
rolebinding "jenkins" created
configmap "jenkins-agent" created
```
1. Validate that CJOC is running 
```bash 
 kubectl rollout status sts cjoc
 ```
 1. Read the Admin Password 
 ```bash
 kubectl exec cjoc-0 -- cat /var/jenkins_home/secrets/initialAdminPassword
 ```
 1. Open CloudBees Core in your browser
 ```bash
 echo "Your CloudBees Core URL is http://$DOMAIN_NAME/cjoc"  
```

Click the **Continue** button to move to the next step.

## Access CloudBees Core
At this point, you have a new installation of the CloudBee Core on Modern Plaforms. 

### Create a new master.
Let's create a new manage master for our cluster. 

1. In your new CloudBees Core installation, open Operation Center.
2. Click the [New Master...] button. 
3. Name the new master `develop`
4. Accept the default setting and click [Save] button.

### Create a new pipeline job
Let's create a new pipeline job on our develop manage master.

1. In your new master, Click [New Item] button
2. Enter name for your new example pipeline build.
3. Click the menu New Item[Pipeline] option in the list.
4. Click Ok button
5. Scroll down to the Pipeline script section. Paste the following script into the text box.

```groovy
podTemplate(label: 'kubernetes', containers: [
    containerTemplate(name: 'maven', image: 'maven:3.5.2-jdk-8-alpine', ttyEnabled: true, command: 'cat')
]) {
    stage('Build') { 
        node("kubernetes") {
            container("maven") {      
                git 'https://github.com/jglick/simple-maven-project-with-tests.git';
                sh "mvn -Dmaven.test.failure.ignore clean package"
                junit '**/target/surefire-reports/TEST-*.xml'
                archive 'target/*.jar'              
            }
        }
    }
}
```
6. Click Save
7. Run new build

## Using the `kubectl` to manage the cluster.

In this section, we'll review a few `kubectl` to manage our new cluster. First, let's look at the high-level view of our cluster using `kubectl`

### Setting the context
`Kubeclt` uses context to manage the different enviornments that a user may work with 
```bash
kubectl config set-context $(kubectl config current-context) --namespace=cje 
```
Let's look at all the important objects in our cluster. This done with the `kubectl get` command.

```bash
kubectl get pod,statefulset,svc,ingress,pvc,pv -o wide
``` 

### Reviewing Logs on Cluster Nodes.
Let's review the logs on operation center noe.
1. `kubectl log` provides access to logs on nodes. 

```bash
kubectl logs -f cjoc-0
```
Click the **Continue** button to move to the next step.

## Logging into a node
Sometime you have no choice, but to log into a node to troubleshoot a problem.

Let's log into our master.

### Get list of pods
First, we'll need the name of pod we want to access.
We'll use the get pods command to list all pods

```bash
kubectl get pods
```

You should see output similar to this.

```
NAME             READY     STATUS    RESTARTS   AGE
cjoc-0           1/1       Running   0          3d
develop          1/1       Running   0          9h
```

### Log into the desired pod.
In this cluster, we have `develop` as our managed master.

To log into `develop`, I used the following command.

```bash
kubectl exec teams-myteam-0  -i -t -- bash -li
```
You should now be in the bash shell of your managed master.

Click the **Continue** button to move to the next step.

## Congratulations!!!

<walkthrough-conclusion-trophy></walkthrough-conclusion-trophy>


You've completed the lab. 

[spotlight-create-cluster]: walkthrough://spotlight-pointer?cssSelector=.p6n-zero-state-link-test.jfk-button-primary,gce-create-button
[spotlight-console-menu]: walkthrough://spotlight-pointer?spotlightId=console-nav-menu
[spotlight-console-menu]: walkthrough://spotlight-pointer?spotlightId=console-nav-menu
[spotlight-open-devshell]: walkthrough://spotlight-pointer?spotlightId=devshell-activate-button
[spotlight-instance-checkbox]: walkthrough://spotlight-pointer?cssSelector=.p6n-checkbox-form-label
[spotlight-delete-button]: walkthrough://spotlight-pointer?cssSelector=.p6n-icon-delete
[spotlight-create-cluster]: walkthrough://spotlight-pointer?cssSelector=.p6n-zero-state-link-test.jfk-button-primary,gce-create-button
[spotlight-submit-create]: walkthrough://spotlight-pointer?spotlightId=gce-submit-button
[spotlight-instance-name]: walkthrough://spotlight-pointer?spotlightId=gke-cluster-add-name
[spotlight-instance-zone]: walkthrough://spotlight-pointer?spotlightId=gce-vm-add-zone-select
[spotlight-number-nodes]: walkthrough://spotlight-pointer?cssSelector=p6n-grid-colp6n-col12
[spotlight-file-editor]: walkthrough://spotlight-pointer?cssSelector=p6n-devshell-launch-web-editor-button
[spotlight-cluster-connect]: walkthrough://spotlight-pointer?spotlightId=gke-cluster-connect
