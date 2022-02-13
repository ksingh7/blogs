
# Kubernetes Endpoint Object: Your Bridge to External Services
![](https://miro.medium.com/max/1400/1*riDxnSkow0nnsxX1oRZJKQ.jpeg)

Photo by Andre A. Xavier

Being an active kubernetes user for the last 3 years, I practically learned an old concept in a new way. It's very likely that being a k8s user you haven’t paid attention to or never know what an **endpoint object **is, however under the covers, you have been using it, full guarantee :)

**One liner explanation to 2 key concepts of kubernetes**
> **What is a Service in k8s**
A service is a k8s object that exposes an application running in one or many pods as a “network service”
> **What is an Endpoint in k8s
**An Endpoint is a k8s object that lists all the addresses (IP addresses and Ports) that are used by a service

**Summary: **For every Service Object, there is a corresponding Endpoint object, and its created automagically by the Service object.

![](https://cdn-images-1.medium.com/max/2316/1*1t8WYc18RS9SWGWRJBuKww.png)

## A unique use-case for K8s Endpoint Object

If you are using k8s in production, then most likely it's not “The Only” infrastructure/application layer on which your business run. For example :

* If your business runs on public cloud, then very likely you will be using a combination of k8s and non-k8s services. Example k8s for cloud-native apps and externally managed DB services such as RDS, Cloud SQL, etc

* If your business runs on-premise, then you might have k8s for modern applications, while VMware, OpenStack, HyperV, RHEV, etc for other traditional applications.

So how can you connect your cloud-native apps running in k8s to external services

![](https://cdn-images-1.medium.com/max/2000/1*WWGO46n16FerHGe-631dSQ.png)

## **The Endpoint Way**

Simple answer, you can create a kubernetes Endpoint object by providing the IP addresses and port number of your external (non-k8s) services. And later create a kubernetes service using that endpoint.

![](https://cdn-images-1.medium.com/max/2000/1*KhcfhurYD8RUTZVXuaWf1A.png)

## Practical use-case and Implementation

In my setup, I have been running Red Hat Ceph Object Storage (Ceph) with 12 RadosGateway services, which means I had a total of 12 RGW endpoints (for performance and high availability reasons). Ceph was running external to kubernetes.

In order for my apps (Presto, Spark, and Juypter) (running inside kubernetes) to access the Ceph Object Storage service, there has to be a single, reliable endpoint which the apps should connect to and access storage over the network.

So I created a k8s Endpoint object with all 12 Ceph RadosGW endpoints and later created a k8s service using that k8s endpoint. This allowed me to have a k8s native single, reliable, and auto load-balanced endpoint for my Ceph object storage service.

![](https://cdn-images-1.medium.com/max/2000/1*SRsoVf_QToKWz3bbJC4Qng.png)

```
apiVersion: v1
kind: Service
metadata:
  name: ocs-external-rgw-lb
spec:
 ports:
    - protocol: TCP
      port: 80
      targetPort: 80
---
apiVersion: v1
kind: Endpoints
metadata:
  name: ocs-external-rgw-lb
subsets:
  - addresses:
    - ip: 192.168.7.50
    ports:
    - port: 8080
  - addresses:
    - ip: 192.168.7.50
    ports:
    - port: 8081
  - addresses:
    - ip: 192.168.7.51
    ports:
    - port: 8080
  - addresses:
    - ip: 192.168.7.51
    ports:
    - port: 8081
  - addresses:
    - ip: 192.168.7.52
    ports:
    - port: 8080
  - addresses:
    - ip: 192.168.7.52
    ports:
    - port: 8081
  - addresses:
    - ip: 192.168.7.53
    ports:
    - port: 8080
  - addresses:
    - ip: 192.168.7.53
    ports:
    - port: 8081
  - addresses:
    - ip: 192.168.7.54
    ports:
    - port: 8080
  - addresses:
    - ip: 192.168.7.54
    ports:
    - port: 8081
  - addresses:
    - ip: 192.168.7.55
    ports:
    - port: 8080
  - addresses:
    - ip: 192.168.7.55
    ports:
    - port: 8081
```

For the sake of argument, I can definitely throw an SW or HW load balancer outside k8s and terminate all my 12 Ceph RGWs there. This approach should have worked as well but comes with some limitations such as 1) the LB would have been a SPOF, 2) delivers a limited performance 3) 1 extra hop to access storage.

## Summary

I found the k8s Endpoint object approach to be very useful, simple, and k8s native. I am definitely gonna use this for accessing other external services, maybe DB running outside k8s.
