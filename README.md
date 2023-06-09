# argocd-learning

# IN-CLUSTER DEPLOYMENT
create a cluster using cluster config

`k3d cluster create argocd-cluster --config ./cluster_config.yaml`

Create a namespace for argo cd resources
 ` kubectl create ns argocd`

 Apply the argocd resources

`kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.6.8/manifests/install.yaml`

By default, the argocd api server is not accessible from outside. We need to patch the argocd-server service to `NodePort`or `LoadBalancer` type.

`kubectl patch svc argocd-server  -n argocd -p '{"spec":{"type": "NodePort"}}'`

Now we can port forward the argocd service.

`kubectl port-forward svc/argocd-server -n argocd 8080:443`

In order to login we need the admin password.

`kubectl get secret -n argocd argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo`


# EXTERNAL-CLUSTER DEPLOYMENT
Argo CD can be used to deploy resources to a cluster different from where it is installed.

* Let's Create an external  k8s cluster

`k3d cluster create argocd-cluster-dev --config ./dev_cluster_config.yaml`

* Replace the file host with the ip of your machine in the kubeconfig so as to connect to the above cluster.

* Having created the external cluster we need to set the argocd running on a separate cluster about this external cluster.

We can use the argocd cli to achieve this:

First set the context to argocd-cluster
`kubectl config use-context k3d-argocd-cluster`

Then add the external cluster reference to argocd:

`argocd  cluster add k3d-argocd-cluster-dev --name external-cluster`

Now we can create a project using:
`argocd proj create loony-argocd -d https://192.168.1.3:50216,default -s https://github.com/Kartikey-star/argocd-public-repo.git`

We are creating a private image from our application code base and need to configure the deployment with the correct imagePullSecrets.

switch the context to the dev cluster.

`kubectl config use-context k3d-argocd-cluster-dev`

now create a secret to pull the docker image.

`kubectl create secret docker-registry regcred --docker-server=https://index.docker.io/v2/ --docker-username=<USERNAME> --docker-password=<YOURPASSWORD> --docker-email=<EMAILID> `