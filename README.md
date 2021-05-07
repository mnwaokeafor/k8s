# k8s
Overview of Argo CD


This is a declarative continuous deployment tool that can be used to deploy apps in a k8s cluster.

The way it works is you define all your manifest files in a github repository and Argo Cd will pull all the changes and deploy them for you in a k8s cluster.
Literally is looks out for changes in deployment, replicaSets, service(pods).
The Interactions can be done within the argoCd webUi or  argocd CLI to create apps and sync your resources.

The Components that support ArgoCd are:
1. argocd-server -  This is the apiSever and all the interactions done with ArgoCd goes through it. It works more like the kube apiSever in k8s.
2. repo-server  - This communicates with the github repository and maintains the cache. It pulls all the changes that happens in the github repo, to maintain the deployment changes.
3.  app controller - Maintains the state of deployed resources in the k8s cluster.
5. redis - For Caching 
4. dex server - Forms part of the intallation for the purpose of delegation and authentication.

Installation Steps: {Specific to this use_case}.

 # kubectl create namespace argocd     ---- To create the namespace where argo will run
    
 # kubectl -n argocd apply -f ./filep_path/ARGO/argo-cd/install.yaml  ---- To install the argo dependencies pods ----- The Argo manifest file link is on : https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

   This command creates a a bunch of resources in the argocdNamesoace :
   # kubectl -n argocd get pods  : also a get-all command will show more resources in the current ns argocd
   
   Next step is to consider how argoCd will be exposed.
   At this point, the argoCdServer  is running on the Cluster IP; which can be seen with the command "$ kubectl -n argocd get all"
   service/argocd-server           ClusterIP   xxx.xx.xxx.xxx   <none>        80/TCP,443/TCP

   For this use-case and considering security/management control, it is better to expose argocd with the local Host: 
   $ kubectl port-forward svc/argocd-server -n argocd 8080:443
    On the local browser, this will fireup the argocd UI. 

   Alternatives to options mentioned above:
   1. exposing as a LB: kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'  - the Lb external IP gives the UI interface
   2. As Ingress: https://argoproj.github.io/argo-cd/operator-manual/ingress/

   The default username is admin and for versions 1.9 and later, the password is gotten from:
   $ kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo

   This password can be updated:
    1. get argoCd CLI:  To use the argocd CLI : Ref: https://github.com/argoproj/argo-cd/releases/latest
    brew install argocd
    argocd login localhost:8080 --insecure --username admin --password <password>


   Validate -  $ argocd account get-user-info
                 Logged In: true
                 Username: admin
                 Issuer: argocd
                 Groups: 
               $ argocd account update-password
                *** Enter current password: 
                *** Enter new password: 
                *** Confirm new password: 
                Password updated

   re-login:
   argocd login localhost:8080 --insecure --username admin --password <newPSSWD>   
    'admin:login' logged in successfully
     Context 'localhost:8080' updated

   To test a simple app: 
   kubectl apply -n argocd -f ./<filePATH>/app.yaml

   This will automate deployment of the app on argo sever, the deployments, replicas etc can be controlled by gitops . for changes made on this app, will syncup with argo, if autosync is enabled. 
