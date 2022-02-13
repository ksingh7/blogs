
# Let’s do GitOps using ArgoCD on OpenShift

![](https://miro.medium.com/max/700/1*igfVD-jhpsbrMDdx4Zge3g.png)

Argo CD is a declarative, GitOps continuous delivery tool for Kubernetes (OpenShift). Argo CD follows the **GitOps** pattern of using Git repositories as the source of truth for defining the desired application state.

### Installing ArgoCD Operator on OpenShift

To deploy ArgoCD, you can follow the ArgoCD Operator installation guide [here](https://argocd-operator.readthedocs.io/en/latest/install/openshift/). To ease out things for you (and me), i composed a nifty yaml that can deploy ArgoCD on an OpenShift 4 ( tested on v4.7 ) cluster. [Github Repo](https://github.com/ksingh7/argocd-openshift.git)

    git clone [https://github.com/ksingh7/argocd-openshift.git](https://github.com/ksingh7/argocd-openshift.git)
    cd argocd-openshift

    # Login as Kubeadmin

    # Deploy ArgoCD Operator
    oc create -f 1_argocd_operator.yaml

This will create a namespace argocd and create an openshift subscription for argocd-operator . Once created you can validate the deployment of ArgoCD Operator is successful, by checking on the pods oc get po

    NAME                                 READY   STATUS    RESTARTS   AGE
    argocd-application-controller-0      1/1     Running   0          4h56m
    argocd-dex-server-c79b779d-rgvg2     1/1     Running   0          4h56m
    argocd-redis-6f7cfddbcb-4sx7n        1/1     Running   0          4h56m
    argocd-repo-server-9dc579fc5-qhm26   1/1     Running   0          4h56m
    argocd-server-54b7c68886-9fsjr       1/1     Running   0          4h56m

Or from OpenShift Console, that confirm ArgoCD operator is installed

![](https://cdn-images-1.medium.com/max/2000/1*lW-jdtvmQ3BnE1OtnPGDnA.png)

### Setting up ArgoCD

You need to login to ArgoCD before you can create apps. You can either choose to login via credentials or via OpenShift SSO. I prefer the latter

* Enable ArgoCD to use OpenShift SSO for Authentication by creating all the required RBAC policies, scopes and routes.

    oc create -f 2_argocd_openshift_sso.yaml

* Download ArgoCD CLI & Login as kubeadmin

    brew install argocd
    argocd version --client

    # Login as kubeadmin, skip if you have done it already
    oc login -u kubeadmin -p yeBhc-tIHaQ-r5XtC-BYtIA [https://api.crc.testing:6443](https://api.crc.testing:6443)

* Login to ArgoCD CLI using OpenShift SSO

    ARGOCD_ROUTE=$(oc -n argocd get route argocd-server -o jsonpath='{.spec.host}')

    argocd login --sso $ARGOCD_ROUTE

Note : If the auth URL does not open up in your browser automatically, you might need to manually copy and open that in your browser and authenticate using kubeadmin credentials

* List Clusters and add OpenShift(k8s) cluster to ArgoCD

    argocd cluster add
    argocd cluster add argocd/api-crc-testing:6443/kubeadmin
    argocd cluster list

Note : cluster status might be unknown at this time, but this turn successful once we deploy our first app in the next section.

### Deploying an App the GitOps Way

Let’s understand the configuration file that ArgoCD consumes and deploys our app in GitOps style

![](https://cdn-images-1.medium.com/max/2000/1*AGAWVpDat8uUG6G4a9dmSQ.png)

* Once this file is applied to your OpenShift cluster, ArgoCD will create an ArgoCD application named guestbook under argocd namespace/project.

Note : This is ArgoCD application and not your actual application. Keep in mind, both are different things.

* As per the specs section, (your) application will be deployed in default namespace/project

* (your) application source of truth is git url [https://github.com/argoproj/argocd-example-apps.git](https://github.com/argoproj/argocd-example-apps.git)

* Git branch is HEAD which is the main or master branch, and the path of the application under git repo is guestbook

* Once you create this ArgoCD application, ArgoCD will deploy your guestbook app

    oc create -f guestbook-app/[guestbook-app.yaml](https://github.com/ksingh7/argocd-openshift/blob/main/guestbook-app/guestbook-app.yaml)

* List ArgoCD application

    argocd app list

* Open ArgoCD dashboard from ArgoCD project > Networking > Routes , authenticate using kubeadmin credentials

![](https://cdn-images-1.medium.com/max/2000/1*NknL5LBou-zDYwol_eR5kQ.png)

* Initially the status of the ArgoCD application will be outOfSync , you need to manually click sync >> replace >> synchronise , in-order for ArgoCD to deploy all the application resources on to the OpenShift cluster

![](https://cdn-images-1.medium.com/max/3074/1*eMLEIPSREsEwJ4FZCWerag.png)

* Once your application is deployed on your openshift cluster, the status of your ArgoCD app will turn Healthy and synced

![](https://cdn-images-1.medium.com/max/2000/1*vhjVUGHsSWD3Z6cJKVU8KQ.png)

* At this point all the resources of your app as defined in the git repository, will get deployed to the openshift cluster automatically.

![](https://cdn-images-1.medium.com/max/2000/1*W0LcbRPniDpRkgpdET8zdg.png)

### Manual (intentional) Tweaking

If we manually tweak the app, such as deleting or adding new resource to the existing application, the ArgoCD application status will change from Healthy to outOfSync , because ArgoCD continuously keep a track of your app and all its resources, if there is any change observed during reconciliation, ArgoCD warns and can bring back the app to its original state.

* Let’s try to introduce a manual change, by creating a route from CLI

    oc expose svc/guestbook-ui -n default

* Verify ArgoCD application status, it will be changed from Healthy to OutOfSync

![](https://cdn-images-1.medium.com/max/2000/1*LLMpW-1d4tt_yhFarzTDjw.png)

* To synchronise the app to its original state, click sync >> purge >> synchronise . ArgoCD will delete any extra resource that is not a part of your app as per git repository. Git repo is the single source of truth for ArgoCD

* Once synchronise in a matter of minutes, your application will be reconciled and extra route will be deleted, and status of ArgoCD app will be Healthy again.

![](https://cdn-images-1.medium.com/max/2000/1*PoGGhyY2HMzXNlgXDu5-_w.png)

**Bonus Tip** : if you like ArgoCD to synchronise the app automatically, i.e without you clicking sync every-time, add this section to the ArgoCD application

    spec:
       syncPolicy:
         automated: {}
> This is how Argo CD automates the deployment of the desired application states in the specified target environments.
