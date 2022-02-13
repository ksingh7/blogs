Allow Containers to run as root on OpenShift 4 : Hack
=====================================================

🤫 Don’t tell anyone that i shared this trick with you

![](https://miro.medium.com/max/1400/1*_JjDhhBlJWH0fAc_LshG9w.jpeg)Photo by [Max Bender](https://unsplash.com/@maxwbender?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/hacker?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

Let me tell you that OpenShift is the most secure Kubernetes distribution on this planet. So OpenShift has the responsibility to secure your apps, which is why OpenShift does not allow containers to run as root.

> **“ First Principles : Never ever run your containers as root user”**

Having said that, there are some instances when you want to run a pokemon container image that you found on some random container repository and want to run that to your OpenShift homelab/dev/test clusters.

Well to do so, you need to allow running container image as root and this is how you can do it.

1.  Login to OpenShift as system:admin

```
oc login -u system:admin -n default
```

2\. Create a new project where you will be running that in-secure container

```
oc new-project pokemon-prj
```

3\. Add the security policy `anyuid` to the service account responsible for creating your deployment, by default this user is default. The dash `z` indicates that we want to manipulate a service account

```
oc adm policy add-scc-to-user anyuid -z default
```

4\. You are all set, go and deploy or re-deploy your containers, it should work now, in `pokemon-prj` project

Summary
=======

*   Don’t ever run containers as root in production environments
*   Don’t tell anyone that you learned this hack from this blog
