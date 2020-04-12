# GitOps with Tekton and ArgoCD
## Simple Node JS App Soup to Nuts: From DeskTop Docker to OpenShift Cluster

This is a simple tutorial that helped me get an understanding of Tekton and ArgoCD.  

### Create a Fork of this Repo for your own fun.


1. Since you are running a GitOps Example, you want to create a fork of this repo into your own git-repo

    ![alt fork-repo](images/fork-repo.png)

2. Select your user 

    ![alt fork-repo-2](images/fork-repo-2.png)

3. You want to create a Clone of your new repo
```console
 git clone https://github.com/<your-user>/node_web_app.git

 cd node_web_app
```

### Run Node App and Test Locally with Docker or Podman 

The Node Applicaiton is created following this tutorial simulating how a new user might learn to containerize a Node App.  
[Dockerizing a Node.js web app](https://nodejs.org/fr/docs/guides/nodejs-docker-webapp/)

1. To run the applicaiton locally, [Install Docker Desktop](https://www.docker.com/products/docker-desktop) or you could use [podman](https://podman.io/) on Linux (This assumes you use podman CLI instead of docker.  Substitute docker <command> with podman <command>)

2. You can run the app locally if you have [node](https://nodejs.org/en/) installed

```
npm install 
node server.js

```



3. Since you have the code, docker build



```
docker build -t <your username>/node-web-app .
```
If you want to use podman locally, follow directions [here](https://developers.redhat.com/blog/2019/09/13/develop-with-node-js-in-a-container-on-red-hat-enterprise-linux/) to build with buildah.


4. Run the applicaiton in a container.
```
docker run -p 49160:8080 -d <your username>/node-web-app

```

5. Check that the container is running.
```
docker ps
```

6. Test the Application 

```
curl -i localhost:49160

```
### Change Pipeline Resource to your git repo.

This change is required to run a build from the console without a Trigger Event.  

![alt fork-repo](images/pipeline-resource-fork.png)

### Using OpenShift 4.3 as my Kubernetes Cluster 

You need your own 4.3 OpenShift Cluster.  Here are some options.  

I [installed OpenShift 4.3 into AWS following these instructions](https://docs.openshift.com/container-platform/4.3/installing/installing_aws/installing-aws-account.html).

You could use [Code Ready Containers](https://cloud.redhat.com/openshift/install/crc/installer-provisioned?intcmp=7013a000002CtetAAC) Locally.  

This tutorial can work also on any Kubernetes, but you have to install Tekton and use the [buildah](https://github.com/tektoncd/catalog/tree/v1beta1/buildah) task. 

### Needed CLI and log into OpenShift

You need the following CLI's 

- [kuberenetes](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [oc](https://docs.openshift.com/container-platform/4.3/cli_reference/openshift_cli/getting-started-cli.html#installing-the-cli)
- [tkn](https://openshift.github.io/pipelines-docs/docs/0.10.5/assembly_cli-reference.html)
- [argocd](https://argoproj.github.io/argo-cd/cli_installation/)


[Log into your OpenShift Cluster](https://docs.openshift.com/container-platform/4.3/cli_reference/openshift_cli/getting-started-cli.html#cli-logging-in_cli-developer-commands) should automatically log you into kubernetes and tekton.  

```
oc login --server=https://<OCP API Server>  --token=<Your Auth Token>
```
Your ID should use an ID with admin access since you will be installing a set of tools.  

You can also log into your OpenShift web console.  


### Install ArgoCD Operator into OpenShift 

There are a few ways to install argocd into a Kubernetes Cluster.  We used the Argo CD Operator.  

For this tutorial, I used [the Console Install](https://argocd-operator.readthedocs.io/en/latest/install/openshift/) using the Argo CD Operator on OpenShift.  

### Install OpenShift Pipeline Operator 

OpenShift delivers a preview of tekton through the OpenShift Pipeline Operator.  We used the OpenShift Pipeline Operator.  

For this tutorial I follwed the instructions to install the [OpenShift Pipeline Operator](https://openshift.github.io/pipelines-docs/docs/0.10.5/assembly_installing-pipelines.html).

### Install the Argo CD Tekton Task into the argocd namespace

After tekton builds the application and pushed the container image into the Image Repository, tekton needs to trigger a new OpenShift Deployment.  There is a special task that allows Tekton to trigger a argocd sync.  You have to install the [Argo CD Tekton Task](https://github.com/tektoncd/catalog/tree/v1beta1/argocd)

### Create OCP Project 
You need to create an OpenShift project called node-web-project or you will need to change all namespaces in the YAML File to match your project 

```
oc new-project node-web-project
```

### Allow Pipeline to access registry for build and deploy
Your project will need the ability to publish and pull from the image repository.  

```
oc policy add-role-to-user registry-editor builder

oc policy add-role-to-user registry-editor deployer
```

### Update argocd secret.

![alt argo-secret](images/argosecret.png)

There is a file caled node-web-app-argocdsecret.template.  Create a copy of that file as yaml.

```
cd pipeline
cp node-web-app-argocdsecret.template  node-web-app-argocdsecret.yaml
```
CAUTION !!!!! the .gitigonore file contains the name of this yaml file to avoid checkin of your credentials.  If you use another name, then you must make sure you DO NOT CHECKIN credentials.  

![alt git-ignore](images/gitignore.png)

In the newly created file, replace the value for ARGOCD_SERVER to your server.  Either enter your ARGOCD_AUTH_TOKEN or User and Password Base 64 encoded.  

![alt argo-secret](images/argosecret.png)

### Examime Pipeline
[node-web-app-pipeline-resources](pipeline/node-web-app-pipeline-resources)


### Create and configure ArgoCD App for Tekton Resources 

We can use argocd to deploy the tekton build for the app.  IN a real project, having your pipeline in a separate repo might be better.  [You can create an argo cd app via the GUI or commandline](https://argoproj.github.io/argo-cd/getting_started/).   

The screenshot below shows the parameters I entered.  You need to use your own forked git repo.  

![alt argo-pipeline](images/argo-node-web-pipeline.png)

- Project: default
- cluster: (URL Of your OpenShift Cluster)
- namespace should be the name of your OpenShift Project
- Targer Revision: Head
- PATH: pipeline
- AutoSync Enabled.  

Once you run sync, your pipeline should be deployed and your screen in argo should look like below.  

![alt argo-pipeline](images/argocd-tekton-pipeline.png)


### Create ArgoCD App for Web App Resources 


![alt argo-pipeline](images/argo-node-web-app.png)

Create Webhook in Git.  

### Sync Repo 
Resources should be created but the deployments fail to start.  



### Run Pipeline 

### Look at app 




