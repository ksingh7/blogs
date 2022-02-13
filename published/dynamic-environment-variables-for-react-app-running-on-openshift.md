Dynamic Environment Variables for React App Running on OpenShift (Kubernetes)
=============================================================================

![](https://miro.medium.com/max/1400/0*JA0wpl0fIaXR2tKY.jpg)

Using Environment variables for React App running locally is dead-easy. However this in production get complex when the same app is deployed on Kubernetes and you are supposed to use dynamic environment variables.

This is my recipe to achieve `$Title`

What is an environment file ?
=============================

Environment file or just env is a file holds variables and some sensitive data about your app

How to push Dynamic Environment Vars to React App hosted on K8s
===============================================================

[Here](https://www.freecodecamp.org/news/how-to-implement-runtime-environment-variables-with-create-react-app-docker-and-nginx-7f9d42a91d70/) is a great blog that documents how to use dynamic env vars on Docker, however in my problem statement, i want to deploy the React app on OpenShift(Kubernetes)

The approach described below is useful in a situation where you want to fetch API URL from an OpenShift route and use that as environment variable for React App. This means you do not want to hardcode variable value in `.env`

*   Export values that needs to be supplied to React App in a local variable

```
export REACT\_APP\_CHAT\_BACKEND=$(oc get route chat-app-backend -o=jsonpath=’{.spec.host}’)
```

*   Using `REACT_APP` as variable prefix add the variable and its value into a local file called `.env` (you can use any name)

```
echo "REACT\_APP\_CHAT\_BACKEND="$REACT\_APP\_CHAT\_BACKEND >> .env  
echo "REACT\_APP\_HTTP\_PROXY=https://"$REACT\_APP\_CHAT\_BACKEND >> .env# Verify file contentscat .env
```

*   Create a Kubernetes Config Map using this `.env`

```
oc create configmap chat-app-frontend --from-file=.env# Verify Config Map  
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

> **Note:** You can the change the config map , volume mount point directory, if your containerised application directory permission allows you to do

*   This will trigger a new pod creation that will now have your custom environment file under path `/tmp/.env` and `env-cmd` command will apply this into your React app once its boots up
*   In case you need to change the environment variable or add a new one to `.env` file, you just need to edit the config map or delete & recreate a new config map. Followed by manually destroying the React App POD. Deployment will trigger a new pod creation with the new config map data.

This is how you can apply a dynamic environment variable to your React App, when the app is deployed on Kubernetes.

> Hope this was useful \\o
