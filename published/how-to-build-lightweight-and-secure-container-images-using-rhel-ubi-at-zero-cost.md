
# How to Build Lightweight and Secure Container Images using RHEL UBI at Zero Cost

![](https://www.sdxcentral.com/wp-content/uploads/2019/08/McAfee-Buys-NanoSec-Boosts-Container-Security.jpeg)

Deploying applications in lightweight container images has practical benefits because container images pack all of the dependencies required for your application to function properly. However, you could lose the benefits of [containerization](https://developers.redhat.com/topics/containers) if the container images are too large, and thus take several minutes to boot up the application. In this article, I guide you through how to use [Red Hat Universal Base Images](https://developers.redhat.com/products/rhel/ubi) (UBI) as the foundation to build lightweight and secure container images for your applications.

## Building containerized applications with UBI

Red Hat Universal Base Images provides a lightweight and secure foundation for building cloud-based applications and web applications in containers. With UBI images, reliability, security, performance, and image-lifecycle features are baked in. You can build a containerized application on a UBI image, push it to your choice of registry server, share it easily, and [even deploy it on non-Red Hat platforms](https://developers.redhat.com/blog/2020/03/24/red-hat-universal-base-images-for-docker-users).

Every UBI image is based on [Red Hat Enterprise Linux](https://developers.redhat.com/products/rhel) (RHEL), the most popular enterprise-grade Linux distribution for the past two decades. Building your container image on a foundation of RHEL software ensures the image is reliable, secure, and freely distributable. You don’t need to be a Red Hat customer to use or redistribute UBI images: Just use them and let Red Hat manage the base images for you at no cost.

Ready to get started with Universal Base Images? Let’s go!

## Where to get UBI images

You might be wondering where to get UBI images. Well, they’re easily available on both the [Red Hat Container Catalog](https://catalog.redhat.com/software/containers/search) and [Docker Hub’s official Red Hat repository](https://hub.docker.com/u/redhat). I personally prefer using the Red Hat Container Catalog console because it shows information such as the image size, version, health index, package list, Dockerfile, and multiple options available for fetching the image. The animation in Figure 1 demonstrates these features.

![](https://cdn-images-1.medium.com/max/2000/1*vTOzo4natiKTKxxDv6tQUQ.gif)

## Choose the UBI image for your build

Red Hat Universal Base Images are offered in several flavor:

* **Micro**: A stripped-down image that uses the underlying host’s package manager to install packages, typically using [Buildah](https://buildah.io) or multi-stage builds with [Podman](https://podman.io).

* **Minimal**: A stripped-down image that uses [microdnf](https://github.com/rpm-software-management/microdnf) as a package manager.

* **Standard**: Designed and engineered to be the base layer for all of your containerized applications, middleware, and utilities.

* **Init**: Designed to run a system as PID 1 (the Linux init process) for running multi-services inside a container.

![Choose the right type of UBI for building your applications](https://cdn-images-1.medium.com/max/3154/1*-r6FeWkJLKK1n9Ql9ZgYQA.png)*Choose the right type of UBI for building your applications*

## Container images for language runtimes

In the next sections, we’ll package UBI images for two different language runtimes-one for [Golang](https://developers.redhat.com/topics/go) and one for [Python](https://developers.redhat.com/topics/python). I’ve already created a sample application for each runtime, so we will simply pack the application into a UBI image. You can get the code for the sample applications, including the Dockerfile, from my [GitHub repository](https://github.com/ksingh7/dockerfile-hub.git).

## Build and run a Golang application with UBI

We’ll start with a Dockerfile:

    FROM golang AS builder
    WORKDIR /go/src/github.com/alexellis/href-counter/
    RUN go get -d -v golang.org/x/net/html  
    COPY ./app.go ./go.mod ./
    RUN CGO_ENABLED=0 GOOS=linux go build -a -o app .

    FROM registry.access.redhat.com/ubi8/ubi-micro
    WORKDIR /
    COPY --from=builder /go/src/github.com/alexellis/href-counter/app /
    EXPOSE 8080
    CMD ["./app"]

Here we are using a multi-stage build, which is a popular way to build a container image from a Dockerfile. The first section of the Dockerfile brings in the official Golang image, and the second section brings in the official UBI image. This Dockerfile demonstrates that UBI images work well with other base images

Now, to pack our sample Golang app into a UBI image, we need to use the FROM command to specify the base image. Here, the base image is the official Red Hat UBI micro image. The WORKDIR command specifies the directory where the app is located inside the container image. The COPY command copies the Golang single binary app to the UBI image and the EXPOSE command specifies the port that the app will be listening on. Finally, the CMD command specifies the command that will be executed when the container runs.Great-we now have the Dockerfile. Let’s build the image and run it:

    git clone ksingh7/dockerfile-hub.git
    cd dockerfile-hub/go-hello-world-app-ubi
    docker build -t go-hello-world-app-ubi .
    docker images | grep -i go-hello-world-app-ubi
    docker run -it -p 8080:8080 -d go-hello-world-app-ubi
    curl [http://localhost:8080](http://localhost:8080)

The cURL command should return the following response:

    Hello OpenShift!

Upon checking the final image size, we see it’s just 42.8MB Woooo now that’s tiny :)

    $ docker images
    REPOSITORY              TAG     IMAGE ID       CREATED          SIZE
    go-hello-world-app-ubi  latest  ac7f4c163f5c   6 hours ago      42.8MB

## Build and run a Python application with UBI

Now, let’s do the same process we did for Golang for the Python runtime, just for fun. Here’s the Dockerfile:

    FROM registry.access.redhat.com/ubi8/ubi-minimal
    RUN microdnf install -y python3
    WORKDIR /app
    COPY ./requirements.txt ./app ./
    RUN python3 -m pip install -r /app/requirements.txt
    EXPOSE 8080
    CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8080"]

I intentionally choose to use a classic Dockerfile for the Python example because it is easier to understand. The base image used here is the official Red Hat UBI minimal image. The microdnf command installs the Python runtime. The WORKDIR command specifies the directory where the app is located inside the container image. The COPY command copies the Python requirements file to the UBI image, which will then be used by RUN command to install the dependencies. The EXPOSE command specifies the port that the app will listen on. Finally, the CMD command specifies the command that will be executed when the container runs.

Here’s our build and run:

    # Clone the git repo if you have not already done
    git clone ksingh7/dockerfile-hub.git
    cd dockerfile-hub/python-hello-world-app-ubi
    docker build -t python-hello-world-ubi .
    docker images | grep -i python-hello-world-ubi
    docker run -it -p 8080:8080 -d python-hello-world-ubi
    curl [http://localhost:8080](http://localhost:8080)

The cURL command should return the following response:

    {"Hello":"World"}

The final image size here is impressively just 169MB:

    $ docker images
    REPOSITORY              TAG     IMAGE ID       CREATED          SIZE
    python-hello-world-ubi  latest  50c12e1ca549   55 minutes ago   169MB

## Conclusion

If you are serious about containerizing your application and running in production, you should consider using Red Hat Universal Base Image. UBI gives you greater reliability, security, and peace of mind for your containerized applications. And, you can freely distribute UBI-based containerized applications with your friends (and enemies) on both Red Hat and non-Red Hat platforms.

Happy UBI-ing.
