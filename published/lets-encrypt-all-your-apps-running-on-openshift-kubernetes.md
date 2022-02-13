
# Let’s Encrypt all your apps running on OpenShift / Kubernetes

![](https://miro.medium.com/max/560/0*_ZYBHXVxki_h8Qtm.jpg)

## **Introduction**

If you need an automatic SSL/TLS certificate for free, for all your internet facing applications running on OpenShift and Kubernetes, you gotta read this.

## What’s ACME ?

**Automatic Certificate Management Environment** (**ACME**) protocol is a communications protocol for automating interactions between certificate authorities and their users’ web servers, allowing the automated deployment of public key infrastructure at very low cost ([Source](https://en.wikipedia.org/wiki/Automated_Certificate_Management_Environment))

## What’s OpenShift-ACME

[Tomáš Nožička](https://github.com/tnozicka) developed a fantastic ACME controller for OpenShift and Kubernetes that automatically provisions certificates from Let’s Encrypt CA using ACME v2 protocol and manage their lifecycle including automatic renewals. Link to the original Github project is [here](https://github.com/tnozicka/openshift-acme)

## Show me the Code !!

Its a simple two step process

- Deploy OpenShift-ACME controller on your OpenShift cluster, cluster wide
```
    oc new-project acme

    oc apply -fhttps://raw.githubusercontent.com/tnozicka/openshift-acme/master/deploy/cluster-wide/{clusterrole,serviceaccount,issuer-letsencrypt-live,deployment}.yaml

    oc create clusterrolebinding openshift-acme --clusterrole=openshift-acme --serviceaccount="$( oc project -q ):openshift-acme" --dry-run -o yaml | oc apply -f -

    oc get po
```
- Annotate your application route to request SSL Cert from Let’s Encrypt

Once openshift-acme controller is running on your cluster all you have to do is annotate your Route like this:
```
    metadata:
      annotations:
        kubernetes.io/tls-acme: "true"
```
Once you annotate your application route, it might take one or two minutes for Let’s Encrypt to process your request and release a valid cert for your app. You can verify the cert status by listing your route using oc get route my-proxy -o yaml command. Here you will see provisioningStatus and tls certificate

![](https://cdn-images-1.medium.com/max/2000/1*LVY13hoqGpOjVSo_LiuABQ.png)

At this point once you browse your app in browser, it should not complaint about self-sign certs, thanks to Let’s Encrypt and OpenShift-Acme project to automate free certs for our apps on OpenShift.

### \0 That’s All Folks
