Redis-GoLang-React Sample Chat App on OpenShift (Kubernetes)
============================================================
Here is my journal to deploy a sample Redis-GoLang-React Real Time chat application on OpenShift

![](https://miro.medium.com/max/1266/1*AcdUcAfPZEFeTs394MpzRg.png)

Why do you care ?
=================

If you are new to deploying full stack applications on OpenShift (i.e. Kubernetes) you should read on. You will learn, how to

*   Deploy a Go Lang App that implements socket.io and connects to external Redis Enterprise Database and finally expose the API service externally
*   Deploy a React.js Frontend App, that connect to web-socket exposed by your backend API. You will learn how to handle dynamic environment variables for your React App (which is tricky on Kubernetes)

> Game On … !!!

**GitHub Repository :** [ksingh7/basic-redis-chat-demo-go: A Basic redis chat demo app written in Go language (github.com)](https://github.com/ksingh7/basic-redis-chat-demo-go)

Get a Free OpenShift Cluster
============================

*   You can get your Free OpenShift Developer Sandbox by simply creating a free account [here](https://developers.redhat.com/developer-sandbox/get-started)
*   Launch your Developer Sandbox and login to OpenShift Console
*   Download the [OpenShift Client](https://docs.openshift.com/container-platform/4.7/cli_reference/openshift_cli/getting-started-cli.html) on your local machine
*   Grab `OC CLI` login credentials from your OpenShift Console (shown below)

![](https://miro.medium.com/max/526/0*T4g_C73HulYU6sY7.png)

![](https://miro.medium.com/max/700/0*MCLEDY3vzBo4JM6L.png)

*   Login to OC CLI

![](https://miro.medium.com/max/700/1*u9NWzUFi2bMZTMhHBv-wQw.png)

Deploying Go Lang Backend on OpenShift
======================================

This demo application requires Redis, you can create your free Redis cluster provide by Redis Inc from [here](https://redis.com/try-free/)

*   Get Redis cluster address and credentials and set then as environment variables

```
export REDIS\_ADDRESS=redis-xxxx.ap-south-1-1.ec2.cloud.redislabs.com:18689  
export REDIS\_PASSWORD=xxxx
```

*   Using `oc new-app` deploy Go Lang backend app on OpenShift. The `oc new-app` will use `image steam` to build container image from source github repository and create Openshift Deployment configuration

```
oc new-app golang~https://github.com/ksingh7/basic-redis-chat-demo-go.git --name=chat-app-backend --env REDIS\_ADDRESS=$REDIS\_ADDRESS --env REDIS\_PASSWORD=$REDIS\_PASSWORD ;
```

*   Create Service and Deployment

```
oc apply -f https://raw.githubusercontent.com/ksingh7/basic-redis-chat-demo-go/master/backend\_ocp.yaml
```

*   Building container image from source repo could take sometime, you can monitor the build process by using the following command

```
oc logs -f buildconfig/chat-app-backend
```

*   Once the build process is completed, you can check the status of pods

```
oc get po
```

*   Once you see the backend pods running, you can access the backend API using the following endpoint

```
export REACT\_APP\_CHAT\_BACKEND=$(oc get route chat-app-backend -o=jsonpath='{.spec.host}')curl https://$REACT\_APP\_CHAT\_BACKEND/health
```

*   If you see a `{“health”:”OK”}` in your API response your Chat backend is all good

Deploying React Frontend on OpenShift
=====================================

*   React frontend needs to know the backend API endpoint to connect to it and provide you a full chat app experience. For this we will create a `.env` file containing Backend API endpoint details

```
echo "REACT\_APP\_CHAT\_BACKEND="$REACT\_APP\_CHAT\_BACKEND >> .env  
echo "REACT\_APP\_HTTP\_PROXY=https://"$REACT\_APP\_CHAT\_BACKEND >> .env
```

> More details in the last section on how to use Dynamic Environment variables for React App running on OpenShift / Kubernetes

*   Create a OpenShift config map from the `.env` file

```
oc create configmap chat-app-frontend --from-file=.env
```

*   Use the following YAML to deploy React App on OpenShift. This configuration file will use Image Stream to Build Container image from Source git repository, create OpenShift Deployment, Service and Route (all in one, power of OpenShift)

```
oc create -f https://raw.githubusercontent.com/ksingh7/basic-redis-chat-demo-go/master/frontend\_ocp.yaml
```

*   Building container image from source repo could take sometime, you can monitor the build process by using the following command

```
oc logs -f buildconfig/chat-app-frontend
```

*   Once the build process is completed, you can check the status of pods

```
oc get po
```

> **Tip** : Even after frontend container is running, allow a few minutes for React app to compile and start the server and be available to the world

```
export REACT\_APP\_CHAT\_FRONTEND=$(oc get route chat-app-frontend -o=jsonpath='{.spec.host}')  
echo "https://"$REACT\_APP\_CHAT\_FRONTEND
```

At this point you would be having the full fledge `Redis-GoLang-Socket.io-React :: Chat App Demo` running on OpenShift

![](https://miro.medium.com/max/700/1*bYgR7V13VDuoCyxlloabAA.png)

Chat App Login Screen

![](https://miro.medium.com/max/700/1*xmS4umu3kWWlqY1VpX5KTw.png)

Chat App Main Screen

![](https://miro.medium.com/max/700/1*TvwNgv2gGGOGRym4Ek1r2A.png)

OpenShift Resources

> Bonus

Handling Dynamic Runtime Environment Variables in React App Deployed on Kubernetes
==================================================================================

What is an environment file ?
-----------------------------

Environment file or just **env** is a file holds variables and some sensitive data about your app

How to push Dynamic Environment Vars to React App hosted on K8s
---------------------------------------------------------------

[Here](https://www.freecodecamp.org/news/how-to-implement-runtime-environment-variables-with-create-react-app-docker-and-nginx-7f9d42a91d70/) is a great blog that documents how to use dynamic env vars on Docker, however in my problem statement, i want to deploy the React app on OpenShift(Kubernetes)

The approach described below is useful in a situation where you want to fetch API URL from an OpenShift route and use that as environment variable for React App. This means you do not want to hardcode variable value in `.env`

*   Export values that needs to be supplied to React App in a local variable

```
export REACT\_APP\_CHAT\_BACKEND=$(oc get route chat-app-backend -o=jsonpath=’{.spec.host}’)
```

*   Using `REACT_APP` as variable prefix add the variable and its value into a local file called `.env` (you can use any name)

```
echo "REACT\_APP\_CHAT\_BACKEND="$REACT\_APP\_CHAT\_BACKEND >> .env  
echo "REACT\_APP\_HTTP\_PROXY=https://"$REACT\_APP\_CHAT\_BACKEND >> .env\# Verify file contentscat .env
```

*   Create a Kubernetes Config Map using this `.env`

```
oc create configmap chat-app-frontend --from-file=.env\# Verify Config Map  
oc get cm
```

*   In your react app, install `env-cmd` library

```
npm install env-cmd
```

*   Edit start script in `package.json` and add `env-cmd -f <container_path>/.env` as prefix (shown below)

```
"scripts": {"start": "env-cmd -f /tmp/.env react-scripts start","build": "react-scripts build"},
```

*   Git commit and deploy your modified React App. If you are using OpenShift S2i / Build Config, just trigger the build
*   Don’t worry your app container will fail at first complaining it can’t find `/tmp/.env` file
*   To fix it add the config map to deployment configuration

```
oc set volume deployment/chat-app-frontend --add --type configmap --configmap-name chat-app-frontend --mount-path /tmp
```

*   This will trigger a new pod creation that will now have your custom environment file under path `/tmp/.env` and `env-cmd` command will apply this into your React app once its boots up
*   In case you need to change the environment variable or add a new one to `.env` file, you just need to edit the config map or delete & recreate a new config map. Followed by manually destroying the React App POD. Deployment will trigger a new pod creation with the new config map data.

This is how you can apply a dynamic environment variable to your React App, when the app is deployed on Kubernetes.
