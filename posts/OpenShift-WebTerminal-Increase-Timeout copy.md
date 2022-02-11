---
title: Andys blog-3
description: My article description
tags: 'openshift,web-terminal, kubernetes, openshift-console'
cover_image: ./assets/cat.jpeg
canonical_url: null
published: true
---
![](https://developers.redhat.com/sites/default/files/styles/article_feature/public/blog/2020/09/WebTerminal_TechPreview_1x.png)
## Introduction

Hello World!

[OpenShift](https://short.ksingh.in/rhd-openshift) is an Enterprise-Grade Container Platform that is built on top of Kubernetes and comes with an extensive set of tools and services that make it super easy to build, deploy, and manage applications, part of the credit goes to OpenShift Console. Directly from OpenShift Console you can also launch a cloud shell or a web terminal

In this post i will share details on OpenShift Web Terminal an embedded command line terminal instance in the OpenShift web console. If you are familiar with public cloud providers consoles or portals, you would have noticed a shell icon in the top right corner of the console. OpenShift Web Terminal Operator enables the same functionality on OpenShift. 

I will not go into the details of installing OpenShift WebTerminal, instead will recommend read  these amazing blogs posts

- [Install OpenShift's Web Terminal Operator in any namespace](https://short.ksingh.in/ocpwebterminal)
- [Web Terminal : One more reason to ❤️ OpenShift](https://ksingh7.medium.com/web-terminal-one-more-reason-to-%EF%B8%8F-openshift-38b640e8c6b)

## Increasing Web Terminal Timeout

By default the web terminal pod timeout is set for 15 minutes, for all practical reasons i found it too short. I will share with you how to increase the timeout of OpenShift's web terminal pod. Let's go !!

- Prerequisites : OpenShift Web Terminal Operator is installed
- Lanuch OpenShift Web Terminal
- Execute the following command to create DevWorkspaceOperatorConfig
```python
cat <<EOF | oc apply -f -
apiVersion: controller.devfile.io/v1alpha1
kind: DevWorkspaceOperatorConfig
metadata:
  name: devworkspace-operator-config
  namespace: openshift-operators
config:
  workspace:
    idleTimeout: 8h
EOF
```

## Summary
This how to increase the timeout of OpenShift's web terminal pod. Now the web terminal pod will timeout after 8 hours. Incase the web terminal UI report `disconnect` don't worry, its just the UI component that was unable to establish a connection with the web terminal pod. To fix, close the web terminal and re-open it.

## More Information on this topic
- [Install OpenShift's Web Terminal Operator in any namespace](https://short.ksingh.in/ocpwebterminal)
- [Cluster tooling updates and more in Red Hat OpenShift's Web Terminal Operator 1.3](https://short.ksingh.in/ocpwebterminal2)
- [What's new in Red Hat OpenShift's Web Terminal Operator](https://short.ksingh.in/ocpwebterminal3)
